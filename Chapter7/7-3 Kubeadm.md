# Kubeadm

Kubeadm은 Kubernetes에서 지원하는 Kubernetes Cluster Deployment Tool이다.

확인해야할 메모리, CPU, 포트 등이 있으므로 https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ 에서 확인하면 된다.

# Multi Node

Kubernetes를 사용하기 위해서는 여러 개의 노드로 구성해야할 필요가 있다.

적어도 1개의 Control Plane 과 1개의 Worker가 필요하다.

그래서 학습용으로 사용할 경우 2개 이상의 서버를 구비하는 것은 무리이므로 가상 환경인 Docker, VMware, Vagrant 등으로 노드를 구성하는 것을 추천한다.

# Install Kubeadm

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

각 패키지별로 kubeadm 설치 방법이 다르므로 꼭 확인하고 설치해야 한다. 

kubeadm 을 설치할 때 kubelet과 kubectl도 같이 설치한다.

# Kubeadm Initialization

Control Plane과 Worker 노드 모두에게 kubeadm, kubelet, kubectl 설치가 완료한 후 kubeadm을 시작해서 Kubernetes를 설치한다.

kubeadm init 명령을 사용하면 되며 이는 Control Plane에서 시작해야 한다.

```
# kubeadm init
```

![image1]()

기본 구성으로 사용하려면 kubeadm init 명령만 사용하면 되며 kube-apiserver의 주소를 바꾸거나 Pod Network CIDR의 범위를 바꾸고 싶으면 옵션으로 지정할 수 있다.

kubeadm의 옵션에 대해서는 https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node 에서 확인할 수 있다.

![image2]()

설치가 완료되면 밑에서 정보 3개를 확인해볼 수 있다.

첫 번째로, Kubernetes를 위한 User 설정이다. 기존의 User를 사용하거나 특정 Kubernetes용 User를 사용한다면 mkdir 부분을 따라서 진행해주면 된다.

이 부분을 하지 않으면 Kubernetes 시작을 못하므로 반드시 해주어야 한다. 만약, root일 경우 그 밑에 Alternatively 부분을 따라서 진행해주면 된다.

두 번째로, Kubernetes의 CNI 플러그인 설정이다. YAML 파일로 구성된 CNI 플러그인을 설치해주어야 Pod 간에 통신이 가능하다.

이 것을 진행하지 않으면 Pod 뿐만 아니라 노드도 통신이 되지 않고 kubectl get nodes 명령을 사용해보면 NotReady 인 것을 확인할 수 있다.

![image3]()

마지막으로 Worker 노드에서 kubeadm 을 활용한 클러스터 Join이다.

맨 밑을 보면 join 명령어와 Control Plane 주소와 포트 그리고 Token이 있다.

```
# kubeadm join (CONTROL PLANE ADDRESS:PORT) --token (TOKEN)
```

![image4]()

이 명령어를 Worker 노드에서 실행하면 된다. 만약, 토큰이 사라졌거나 다시 발행해야할 경우에는 kubeadm token create 명령을 사용하면 된다.

```
# kubeadm token create
```

![image5]()

--print-join-command는 kubeadm join 명령까지 포함해서 보여주는 옵션이다.

이 과정을 모두 끝내면 Kubernetes 클러스터 초기 설정은 모두 완료한 것이다.
