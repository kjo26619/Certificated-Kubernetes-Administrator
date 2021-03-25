# Kubernetes Networking

Kubernetes는 Docker의 네트워크와 유사하지만 Container 단위로 관리하는 것이 아닌 Pod 단위로 관리한다.

이러면서 조금씩의 차이를 가져오고 CNM/CNI의 차이가 있다.

# Docker Networking

Docker에서는 Bridge 네트워크, Overlay 네트워크 혹은 MACVLAN 네트워크를 구성하여 Container 간에 통신을 이루어낸다.

Docker는 이러한 네트워크의 구성을 Linux Network Namespace 기술을 사용하여서 구현한다.

흔히 우리가 사용하는 Linux에서 eth0, eth1 등의 물리 네트워크 인터페이스를 사용한 적이 있을 것이다. 

이것을 내부에서 네트워크를 여러 개로 쪼갠 다음 Bridge를 구성하여 서로 간에 연결을 시키는 것이 바로 Linux Bridge이다.

Linux Network Namespace는 여러 개로 쪼개진 네트워크이며 내부에서는 서로 다른 네트워크로 격리되는 것이다.

이 Linux Network Namespace를 다루기 위해 사용하는 것이 ip 명령어이다.

```
# ip [Options] address | route | link | tunnel | neigh | rule [ARGS]
```

ip 명령을 통해 Namespace를 늘릴 수 있고 없앨 수 있으며 서로 간에 통신이 가능하게끔 이어주는 것도 가능하다.

마치 네트워크에서 LAN 상에서 VLAN으로 다시 나누는 것과 마찬가지이다.

Docker도 이 방법을 통해 처음에는 docker0라는 Network Namespace를 가지게 된다.

![image1]()

이 docker0 Bridge가 기존의 물리 인터페이스와 연결되어 인터넷 혹은 밖과 통신할 수 있는 것이다.

Docker는 이제 Container가 생성되면 docker0 내부에 virtual eth라는 Container Network Namespace를 생성한다. 생성한 virtual eth와 Container의 eth를 연결한다.

이렇게 함으로써 Container들은 서로 통신이 가능한 것이다.

대신, 외부와 내부 Container 간에 통신을 하기 위해서는 Port 연결이 필요하다. 

그래서 외부와 내부 Container의 연결이 필요할 때에는 Docker Container를 만들 때 -p 옵션을 이용하여 포트포워딩 설정을 하는 것이다.

# Container Networking Interface

Container Networking Interface(CNI)는 Cloud Native Computing Foundation(CNCF) 프로젝트로 Linux Container에서 네트워크 인터페이스를 구성하는 플러그인을 작성하기 위한 표준이다.

CNI는 컨테이너가 생성되면서 네트워크 인터페이스를 가지고 네트워크에 연결, 그리고 삭제될 시 네트워크 할당의 취소 같은 일련의 과정을 정의해놓는다.

네트워크를 구성할 때 CNI 표준을 따르는 것을 CNI 플러그인이라고 한다.

즉, CNI는 CNI 플러그인을 개발하는 방법, Container Runtime 때 플러그인을 부르는 방법 등을 정의한다.

많은 플러그인이 CNI로 구성되어 있다. weave, flannel, VMware NSX, Calico 등등...

Kubernetes 역시 CNI 플러그인을 지원한다.

하지만 여기서 주의할 점은 Docker는 CNI를 구현하지 않는다. Docker는 CNI와 매우 유사한 Container Network Model (CNM) 으로 구현된다.

CNM 역시 CNI와 비슷한 네트워크 인터페이스 표준 구현이며 Cisco Contiv, Kuryr, Open Virtual Networking 등이 있다.

그래서 Docker Container를 Kubernetes에서 사용한다면 Container를 Docker의 None Network 에 생성한다.

이 None Network는 서로 통신할 수 없는 네트워크로 연결이 안되어있다는 의미이다.

그리고 나서 Kubernetes에서는 CNI 플러그인을 이용해 네트워크를 구성한다.

# Kubernetes Cluster Networking

Kubernetes 에서 클러스터를 구성하려면 노드는 네트워크를 위해 다음과 같은 구성이 필요하다.

- 각 노드는 한 개 이상의 네트워크 인터페이스가 있어야 한다. (네트워크에 연결되어 있어야 함)
- 각 인터페이스에는 주소가 구성되어 있어야 한다.
- 각 호스트는 고유의 호스트 네임이 존재해야 한다. (예를 들어, MAC Address)
- 반드시 열어야 하는 포트가 있다.

즉, Kubernetes Cluster 구성을 위해서는 노드 모두 서로 식별 가능한 네트워크 내에 존재해야 한다.

특히, 포트가 반드시 열려 있어야 하는데 Control Plane의 경우에는 kube-apiserver, kubelet, kube-scheduler, kube-controller, etcd를 가지므로 열려 있어야할 포트가 꽤 많다.

kube-apiserver 같은 경우에는 6443, kubelet은 10250, kube-scheduler는 10251, kube-controller-manager는 10252, etcd는 2379 포트를 갖는다.

Worker 노드는 kubelet을 위한 10250과 Service를 위한 30000-32767은 열려있어야 한다.

클러스터 구성이 완료되면, 클러스터 내부에서 Pod 간에 통신을 위한 네트워크 플러그인과 네트워킹과 네트워크 정책 관리를 위한 애드온을 선택한다.

종류와 기능은 https://kubernetes.io/docs/concepts/cluster-administration/networking/ 와 https://kubernetes.io/docs/concepts/cluster-administration/addons/ 에 있다.

이제 클러스터 내부에서 사용할 플러그인을 선택하여 설치하면 Pod간의 통신이 지원된다.
