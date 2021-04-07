# Cluster Upgrades

Kubernetes는 꾸준히 업데이트 되며 새로운 기능들이 추가되고 있다.

그렇기 때문에 추가 기능을 사용하기 위해서 Kubernetes 클러스터 자체를 업그레이드 해야하는 일이 발생할 수 있다.

이러한 버전은 다음과 같이 이루어진다.

```
v1.19.3
```

여기서 1.19.3의  1은 Major 버전이다. 그리고 19는 Minor 버전으로 대체로 이 숫자가 바뀌면 추가 기능이나 기능의 변화가 일어난다.

마지막으로 3은 Path로 버그나 문제가 될만한 사항을 고친 후 배포하는 것이다.

버전 별로 바뀐 요소에 대해서는 https://kubernetes.io/docs/setup/release/notes/ 에서 확인할 수 있다.

그리고 각 구성요소마다 버전이 있는데 호환 버전에 대한 것은 https://kubernetes.io/docs/setup/release/version-skew-policy/ 에서 확인해볼 수 있다.

예를 들어, kube-apiserver v1.19.0, kubelet v1.20.0 를 같이 사용할 수 있다 라는 내용들이 적혀 있다.

시간이 지날수록 버그가 고쳐지고 기능이 추가되므로 Kubernetes는 꾸준히 업그레이드해줄 필요가 있다.

하지만 현재 수많은 Pod들이 노드에 위치해있고 이 Pod들의 서비스가 최대한 끊기지 않아야 한다.

그래서 클러스터를 업그레이드 할 때는 다음과 같이 진행 해야 한다.

# Control Plane Upgrades Process

먼저, 핵심은 Control Plane의 업그레이드이다. 

아무래도 주력 구성 요소가 모두 모여 있고 각 Worker 노드들이 이 Control Plane의 버전에 맞춰야 하므로 업그레이드를 유의해서 해야 한다.

대부분의 Control Plane에는 Taint가 설정되어 있어서 Pod를 받지 못할 것이다.

하지만 내부 동작하는 Pod들도 있고 여러 구성요소도 있으므로 먼저 drain 명령을 통해서 Pod가 할당되지 못하도록 한다.

```
# kubectl drain (CONTROL PLANE NAME) --ignore-daemonset
```

drain을 통해서 Control Plane을 잠궜으면 이제 kubeadm의 upgrade 명령어를 사용해서 업그레이드 가능 현황을 확인한다.

```
# kubeadm upgrade plan
```

이 명령을 통해서 현재 구성요소들의 버전과 업그레이드 가능 버전을 확인해볼 수 있다.

확인이 되면 원하는 업그레이드 버전으로 kubeadm을 업그레이드한다.

```
# apt-get update
# apt-get install -y kubeadm=(VERSION [ex) 1.19.0-00, 1.20.0-00])
```

apt-get이나 yum 등의 패키지를 이용해서 설치하였다면 다시 kubeadm upgrade apply 명령을 이용한다.

```
# kubeadm upgrade apply (VERSION [ex) v1.19.0, v1.19.3])
```

kubeadm이 원하는 버전으로 업그레이드할 수 있다면 업그레이드가 적용된다.

kubeadm에 의해서 설치되는 대다수의 구성 요소들이 업그레이드 되나 kubelet은 따로 해주어야 한다. 

kubeadm이 끝났다면 kubelet을 업그레이드 해주어야 한다.

```
# apt-get install -y kubelet=(VERSION)
```

똑같이 패키지 툴을 통해서 kubelet을 업그레이드 해주면 된다.

kubelet의 확인은 kubectl get nodes를 통해서 현재 버전을 확인할 수 있다.

```
# kubectl get nodes
```

똑같은 방법으로 kubectl도 업그레이드 해줄 수 있다.

작업이 끝나면 drain으로 격리시켜두었던 Control Plane을 uncordon 해줄 수 있도록 한다.

```
# kubectl uncordon (CONTROL PLANE NAME)
```

# Worker Node Upgrades Process

이제 Control Plane을 업그레이드 했다면 다음은 Worker 노드이다.

Worker 노드의 경우에도 크게 다르지 않다.

먼저, Control Plane에서 Worker 노드를 drain 해준다.

```
# kubectl drain (WORKER NODE NAME)
```

drain 설정으로 인해 Worker 노드에 있는 모든 Pod들은 다른 노드로 이동한다.

여기서 유의할 점은 drain 노드 외에 다른 노드가 없다면 Pod는 종료된다. 그러므로 Control Plane에라도 옮기기 위해서는 Control Plane의 Taint를 제거해주어야 한다.

그리고 위와 마찬가지로 kubeadm을 Control Plane 버전에 맞게 설치해주면 된다. 대신 유의할 점은 Worker 노드 자체에서 진행해야 한다. 

즉, ssh나 터미널로 접속하여서 진행해야 한다.

설치 후에 kubeadm upgrade를 진행하는데 이 때 plan이나 apply가 아닌 node를 사용하면 된다.

```
# kubeadm upgrade node
```

kubeadm 적용이 끝나면 위와 똑같은 방법으로 kubelet을 업그레이드 시켜주면 된다.

모두 업그레이드가 끝나고 나서 drain을 취소해주면 된다.

```
# kubectl uncordon (WORKER NODE NAME)
```

하지만 이미 이동한 Pod의 경우에는 다시 이동하지 않는다.
