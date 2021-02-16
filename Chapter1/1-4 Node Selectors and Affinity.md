# Node Selectors and Affinity

Taints와 Tolerations는 원하는 Node에 Pod를 배치하는 Scheduling 기법이 아니다. 잘못된 Node에 Pod를 배치하는 문제를 미연에 방지하기 위해 사용한다.

원하는 Node에 원하는 Pod를 배치하는 기법은 Node Selector와 Node Affinity이다.

Node Selector는 Node에 Labels를 정하고 Pod에 Node를 지정한다.

Node Affinity는 선호도로 어떠한 조건을 만족하는 Node에 Pod를 Scheduling할 것을 지정한다.

# Node Selectors

Node Selectors는 매우 간단한 방법이다. 

먼저, Node에 Labels를 지정한다. Labels는 <key>=<value>로 구성되며, kubectl label nodes 명령어를 사용하면 된다.
  
```
# kubectl label nodes (NODE NAMES) (KEY)=(VALUE)
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa1.PNG)

그리고 kubectl describe node 명령어를 통해서 Labels가 추가된 것을 확인할 수 있다.

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa2.PNG)

이제 Pod를 설정할 때 spec 섹션에 nodeSelector 라는 특성을 추가해주면 된다. 

nodeSelector는 Key: Value로 구성해주면 된다.

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa3.PNG)

Node Selector를 지정한 뒤 Pod를 만들면 제대로 Running 되는 것을 확인할 수 있다. 

또한 kubectl describe node 명령어를 통해 지정한 Node로 Scheduling 된 것을 확인할 수 있다.

![image4](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa4.PNG)

만약 Pod를 구성할 때 다른 Labels로 구성하면 Pending이 되면서 Node에 할당이 되지 않는다.

![image5](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa5.PNG)

![image6](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa6.PNG)

![image7](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/nodesa7.PNG)

Pending된 이유는 kubectl describe node 명령어에서 Events의 Message에서 확인할 수 있다.

# Node Affinity

Node Selector는 Node와 Pod 모두에게 Labels를 확실히 지정해주는 방법이다 보니 대규모 서버에서는 하나하나 지정해주기 어려운 상황이 있을 수 있다.

그러면서 등장한 것이 Node Affinity이다. Node Affinity는 Node에 Labels를 지정하고 Pod에 조건을 설정한다.

Node Affinity는 YAML 파일을 구성할 때 spec 섹션에 affinity 라는 특성 밑에 nodeAffinity를 추가해주면 된다.

affinity의 구성은 다음과 같다.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

affinity 밑에 nodeAffinity가 있으며 그 밑에는 여러가지 조건을 지정해줄 수 있다.

먼저 나오는 requiredDuringSchedulingIgnoredDuringExecution은 2가지 타입이 있으며 

requiredDuringSchedulingIgnoredDuringExecution 과 preferredDuringSchedulingIgnoredDuringExecution 이다.

이는 자세히 살펴보면 DuringScheduling이 다른 것을 확인할 수 있다.

1. requiredDuringSchedulingIgnoredDuringExecution : Node Affinity의 조건에 알맞은 Node에 Scheduling 할 것.

2. preferredDuringSchedulingIgnoredDuringExecution : Node Affinity의 조건에 알맞은 Node에 Scheduling 할 것. 하지만, 없을 경우 지켜지지 않음. ( 보장되지 않음 )

IgnoredDuringExecution은 Pod들이 실행중일 때 Node가 가진 Label이 달라지면 어떻게 할 것인지를 명시한다.

하지만 현재는 Ignored만 제공되고 있다. 즉, 실행중일 때 바뀌는 것은 무시한다는 의미이다.

이후에는 requiredDuringExecution를 제공할 예정이라고 한다.

그리고 다음 nodeSelectorTerms는 Node를 선택할 때의 조건을 지정하는 곳이다. 

nodeSelectorTerms를 여러 개 지정할 경우 하나의 nodeSelectorTerms의 조건에 맞는 Node가 있다면 할당한다.

그리고 다음 matchExpressions는 Set 기반 조건을 지정한다. 

matchExpressions를 여러 개 지정할 경우 모든 matchExpressions의 조건에 맞는 Node가 있어야지만 할당한다.

Set 기반 조건에 대한 자세한 내용은 https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements 를 확인하면 된다.

위 YAML 파일의 조건은 Key가 kubernetes.io/e2e-az-name 이고 Value가 e2e-az1이나 e2e-az2 인 Node에 Pod를 Scheduling 하겠다는 의미이다.

또한, Node Affinity는 여러 개를 지정할 수 있다.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

기존에 있던 requiredDuringSchedulingIgnoredDuringExecution 외에도 preferredDuringSchedulingIgnoredDuringExecution를 추가하였다.

이것은 requiredDuringSchedulingIgnoredDuringExecution 의 조건에 부합하는 Node 중에서도 preferredDuringSchedulingIgnoredDuringExecution 의 조건에 부합하는 Node에 최대한 Scheduling 하라는 의미이다.

여기서, preferredDuringSchedulingIgnoredDuringExecution 는 weight를 갖는데 Scheduling과 관련된 가중치이다.

이것은 matchExpressions의 조건에 부합할 경우 Scheduling과 관련된 모든 요소에 대한 가중치를 합산할 때 이 weight를 추가한다. 가중치가 높은 Node로의 Scheduling을 선호한다.

# Pod Affinity

Pod Affinity는 Node Affinity와 유사하지만 Node에 설정된 Labels를 따르는 것이 아닌 Pod에 설정된 Labels를 따르는 것이다.

즉, 조건에 부합하는 Pod가 있는 Node로 Scheduling 하는 방법이다.

정확한 사항은 https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity 에서 확인해볼 수 있다.
