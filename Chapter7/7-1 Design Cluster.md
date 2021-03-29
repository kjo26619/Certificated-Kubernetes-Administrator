# Design Cluster

Kubernetes는 어떠한 목적을 가진 클러스터냐에 따라 설치 방법이 달라진다.

왜냐하면 애플리케이션이 구동되는 서버가 어떤 종류, 어떤 목적인지에 따라 달라지기 때문이다.

만약, 학습용이라면 Single-Node일 가능성이 높다. play-with-k8s에서는 웹 상으로 Multi-Node를 지원해주지만 세션 유지는 4시간 밖에 안된다.

그렇다면 Single-Node에서 Kubernetes를 사용하려면 어떻게 해야하는가? 예를 들어 Minikube를 활용하거나 Kind를 활용할 수 있다.

혹은 클라우드 플랫폼에서 Single-Node로 Kubeadm을 써도 된다.

그렇다면 이제 개발과 테스트 목적이라면 Multi-Node일 것이다.

이 때는 Kubeadm을 사용하는 것을 추천한다.

그리고 클라우드 플랫폼에서는 자체적으로 Kubernetes를 구성하는 툴을 지원해준다.

예를 들어 GCE의 GKE, AWS의 Kops, Azure의 AKS 등이 있다.

이제 클러스터 구성을 하면 알맞은 스토리지와 노드를 고를 필요가 있다.

스토리지는 사용하는 데이터 크기, PV의 구성 등을 고려해서 사용해야 한다.

그리고 노드는 ControlPlane과 Worker Node로 나뉜다. 모두 애플리케이션을 호스팅할 수 있지만 ControlPlane에서 호스팅하는 것은 추천하지 않는다.

그리고 모든 노드는 64비트 Linux 운영체제를 사용해야 한다.

그리고 ControlPlane에 필요한 주요 요소들( kube-apiserver, kube-controller-manager, kube-scheduller, etcd )가 모든 Control Plane 노드에게 반드시 있어야 한다.

즉, ControlPlane은 주요 요소들을 구성할 수 있을 정도의 가용성이 보장되어야 한다.

하지만 대규모 클러스터 같은 경우에는 ETCD를 따로 구성할 수도 있다.

# Turnkey Solutions vs. Hosted Solutions

Kubernetes를 사용하는 클라우드에는 Turnky Solution과 Hosted Solution으로 나뉜다.

Turnkey의 경우에는 스스로 VM을 프로비저닝하고 구성하여 직접 클러스터를 관리하는 솔루션이다.

설치 등은 자동화 되어있지만 스크립트를 짜고 스스로 클러스터를 만드는 솔루션이라고 보면 된다.

유명한 것은 AWS의 Kops이다. 

Hosted Solution은 Kubernetes-As-A-Service 라고 보면 되며 Provider가 관리해주는 솔루션이다.

Provider가 VM 프로비저닝, Kubernetes 설치, VM 유지 등을 관리해주며 사용자는 자신이 원하는 애플리케이션만 배포하면 된다.

유명한 것은 GCE의 GKE이다.
