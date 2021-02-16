# DaemonSets

DaemonSets은 Deployment와 유사하나 그 기능이 다르다. DaemonSets는 특정한 Node 혹은 모든 Node에서 실행되고 있어야할 기본 Pod를 유지한다.

즉, DaemonSets은 특정 Pod를 Kubernetes Cluster에 참여하는 Node들에게 하나씩 모두 배치하고 유지될 수있도록 한다.

DaemonSets은 모니터링을 위해 Log를 수집하거나 기본적으로 모든 노드에서 실행되어야할 기본 Pod 등을 위해 사용된다.

이러한 DaemonSets이 사용되는 기본 Kubernetes 시스템에는 kube-proxy가 있다.

무조건 노드 당 한 개의 Pod를 유지하기 때문에 Replica의 수도 없으며 Scheduler에 의해 Node 배치를 받지도 않는다.

DaemonSets은 확인하기 위해서는 kubectl get daemonset 명령어를 사용하면 된다.

첫 구성 후에 DaemonSets은 kube-system Namespace에서 만들어진 기본 DaemonSets 밖에 없기 때문에 뒤에 --all-namespaces를 붙여서 확인하면 된다.

```
# kubectl get daemonset

# kubectl get daemonset --all-namespaces
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/daemonset1.PNG)

DaemonSets의 설정은 ReplicaSets와 유사하며 몇몇의 차이만 있을 뿐이다.

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/daemonset2.PNG)

apiVersion은 apps/v1으로 같으며 kind가 DaemonSet으로 바뀐다.

그 외에 자잘한 replicas 같은 설정이 필요 없으며 이름과 Pod에서 사용할 Container를 지정하면 된다.

Namespace는 원하는 Namespace로 지정하면 되며 안할 경우 default Namespace로 결정된다. 

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/daemonset3.PNG)

위 사진에서 보는 바와 같이 기존에 만들어진 DaemonSets는 3개( 노드 수 : Control Plane 1, Worker 2 )를 유지하지만 새로 만든 것은 2개만 유지하는 것을 확인할 수 있다.

이는 DaemonSets에서도 Taints와 Tolerations가 적용되기 때문이다. 

즉, Control Plane에 NoSchedule Taint가 적용되어 있고 DaemonSets에는 Tolerations가 지정되어 있지 않으므로 Worker Node에만 Pod가 생성된 것이다.

만약 필요하다면 Tolerations를 추가하여 특정 노드 혹은 전체 노드에게 DaemonSets가 배치되도록 지정할 수 있다.
