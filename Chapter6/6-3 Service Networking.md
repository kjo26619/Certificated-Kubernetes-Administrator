# Service Networking

Kubernetes의 Services는 통신을 위한 기능이다. 클러스터 내부 Pods 간에 통신을 돕거나 내부와 외부의 연결 기능이다.

Serivces의 3가지 타입에 대해서는 https://github.com/kjo26619/Docker/blob/main/Chapter8/8-5%20Kubernetes%20Services.md 에서 설명하였다.

이제 Kubernetes에서 Services가 어떻게 IP 주소를 얻고 통신을 이루는지 알아본다.

# Pod - Services

Pods의 생성에 따른 Kubernetes 네트워크는 다음과 같이 이루어진다.

- Pod 생성 명령이 들어오면 kube-apiserver가 동작하게 되고 이를 각 노드에 있는 kubelet이 감시하다가 명령을 보고 새 Pod를 생성한다.
- Pod 생성이 끝난 뒤 kubelet에 설치된 CNI 플러그인이 네트워크를 구성한다.
- 각 노드는 kube-proxy라는 구성 요소를 실행한다.
 
이제 각 노드에 있는 kube-proxy가 Services를 구성한다.

하지만 Pods와는 달리 Services는 각 노드에 위치해있는 것이 아니다.

Services는 Cluster-wide로 하나의 Services는 하나의 노드가 아닌 클러스터에 존재한다.

# Virtual Objects

Services는 가상 객체이다. Pod처럼 실제로 구동하고 있는 것이 아니라 있다고만 생각하는 가상의 객체이다.

즉, Services는 Pod의 Container 처럼 IP, Namespace, Interface를 따로 할당받지 않는다.

그렇다면 통신은 어떻게 이루어질 것인가가 문제이다.

그래서 Serivces는 사전 정의되어 있는 범위의 IP와 Pod IP의 연결이다.

kube-proxy 내에서 사전 정의되어 있는 범위의 IP와 Pod의 IP를 연결하는 Forwarding 규칙을 만들어낸다.

그리고 사전 정의된 IP로 요청이 들어오면 Pod의 IP로 전달을 해주는 것이 Services가 이루어지는 방법이다.

만약, 10.103.132.104:3306 라는 Serivce가 있고 이것이 10.244.1.2:3306 Pod와 이어져 있다면,

10.103.132.104의 포트 3306으로 들어오는 요청을 10.244.1.2의 3306 Pod로 연결시켜라 라는 의미이다.

kube-proxy에서는 userspace, ipvs, iptables 와 같은 옵션으로 Services를 지원한다.
