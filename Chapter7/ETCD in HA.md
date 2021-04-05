# ETCD

ETCD는 Go 언어로 쓰여진 Key-Value Store로 Kubernetes에서 클러스터 내 데이터를 저장하고 유지하는데 사용한다.

하나의 Control Plane에 ETCD는 반드시 존재하며 Control Plane의 데이터를 저장한다.

그런데 Control Plane에 여러 개의 노드가 존재하면 하나의 노드만 ETCD가 있을 것인지 아니면 여러 ETCD가 존재할 것인지 문제이다.

그리고 DB 특성상 Write에 대한 동기화가 필요하다.

여러 Control Plane을 갖는 Highly Available 환경에서는 ETCD는 다음과 같이 이루어진다.

# ETCD in HA & Raft

일단 여러 Control Plane에게 각자 하나의 ETCD가 존재한다. 혹은 ETCD를 아예 따로 모아놓고 Control Plane과 연결하는 방법이 있다.

그렇다면 동기화 문제는 어떻게 해결할 것인가? 이 때 등장하는 것이 Raft 알고리즘이다.

Raft는 분산 합의 알고리즘으로 Leader & Follower 전략이다.

여러 ETCD 중에 Leader를 하나 선출하고 나머지는 Follower가 된다.

그리고 Leader는 모든 Follower의 Write 작업 요청을 받는다.

만약 1 2 3 ETCD가 있고 1번 ETCD가 Leader라고 하면 2번과 3번은 Write를 작업할 때 1번 ETCD에게 요청을 보낸다.

Leader는 Write 작업을 받으면 ETCD에서 처리를 한 다음 모든 ETCD에게 동기화 작업을 보낸다.

2번이 만약 LastName : Kim 이라는 Key-Value 를 LastName : Lee 로 변경한다면 1번에게 Write (LastName : Lee) 요청을 보낸다.

그러면 1번은 LastName : Lee로 변경한 뒤 2번과 3번에게 LastName : Lee로 변경되었음을 알린다.

그러면 2번과 3번은 자신의 ETCD의 LastName 의 Value를 Lee로 변경한다. 이렇게 해서 데이터를 유지하고 동기화 작업을 진행한다.

# Raft Leader Election

그렇다면 Raft에서 Leader 선출은 어떻게 진행하며 Leader가 만약 사라질 경우 어떻게 작업이 되는지 궁금하다.

Raft 알고리즘이 처음 시작할 때 클러스터 내 모든 노드 (ETCD) 들은 랜덤한 타이머를 가지고 있는다.

그리고 타이머가 가장 먼저 사라진 노드에서 자신이 Leader가 되겠다고 모든 노드에게 요청을 보낸다.

그렇다면 요청을 받은 노드들은 투표를 한다. 투표라고 하지만 정확히는 노드의 Leader 요청에 대한 응답을 하는 것이다.

많은 투표 수를 받은 노드가 Leader가 된다. 

Leader는 선출된 이후부터 모든 Follower에게 일정 시간마다 자신의 정보가 담긴 메시지를 보낸다.

만약 이 시간마다 들어오는 정보가 Follower에게 들어오지 않는다면 Follower에서는 Leader에게 문제가 생겼음을 인지한다.

그렇게 되면 Leader가 비어있게 되고 Raft 알고리즘으로 다시 Leader를 선출한다.

# ETCD in HA - Write

HA에서 ETCD의 Leader는 서로 다른 노드에 위치한 ETCD 간 데이터를 항상 같게 유지해야 한다.

즉, Write에 대해서 모든 노드가 처리되었는지 확인해야 한다. 

Write 작업이 이루어지면 Leader는 모든 Follower에게 Write 작업을 보내고 처리가 완료되면 응답을 받는다.

만약, Follower에게 Write 작업이 도달하지 않거나 Follower가 처리하지 못하면 어떻게 할 것인가가 문제이다.

모든 Follower가 작업을 할 때 까지 기다리면 너무 작업 속도가 느릴 수 있다.

그렇다 해서 무시하고 다른 작업을 하면 같은 데이터 유지가 불가능하다.

이러한 문제점을 해결하기 위해 Quorum 라는 개념을 도입한다.

Quorum은 여러 명이 합의하여 결정할 때 필요한 사람 수를 의미하는 것이다. 클러스터 내에 Leader와 같은 데이터를 유지해야 하는 최소 수라고 생각하면 된다.

Quorum은 N / 2 + 1로 이 수만큼 Write 작업이 완료되면 작업이 완료되었다고 생각하는 것이다.

즉, 3개의 노드이면 3/2 + 1 = 2가 되는 것이다. 7개의 노드이면 7/2 + 1 = 4이다.

이 Quorum은 데이터 유지가 필요한 최소 수이므로 (현재 노드의 수 - Quorum)을 하면 내결함성 (Falut Tolerance)가 되는 것이다.

즉, 3개의 노드는 1개의 노드까지 실패해도 되는 것이고 7개의 노드면 3개의 노드까지 실패해도 되는 것이다.

이렇게 투표를 하다보니 Kubernetes에서 Control Plane HA 구성할 때 홀수로 구성하는 것을 추천하고 있다.

6개의 노드로 구성할 때 네트워크가 쪼개져서 만약 4개 / 2개로 나뉘었다고 가정한다. 4개의 네트워크 내에는 Quorum 수를 충족하므로 ETCD 유지가 가능하다. 

하지만, 3개 / 3개로 나뉘었다고 가정하면 두 네트워크 모두 Quorum 유지가 불가능하다.

만약 7개의 노드로 구성했다면 2개의 네트워크로 나뉘었을 때 최소 노드 수는 4개 / 3개이며 4개가 Quorum 수를 충족하므로 ETCD가 유지될 수 있다.

# NO. of ETCD

ETCD의 수는 Quorum에 최대한 충족해야 하므로 홀수로 구성하는 것이 좋다.

하지만 ETCD의 수 = Control Plane의 수 일 필요는 없다. ETCD를 따로 구성하고 kube-apiserver와 연결해주면 된다.

즉, Control Plane이 2개이고 ETCD는 3개로 구성한 뒤 둘을 연결해주면 된다.
