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

![image1]()

그리고 kubectl describe node 명령어를 통해서 Labels가 추가된 것을 확인할 수 있다.

![image2]()

이제 Pod를 설정할 때 spec 섹션에 nodeSelector 라는 특성을 추가해주면 된다. 

nodeSelector는 Key: Value로 구성해주면 된다.

![image3]()

Node Selector를 지정한 뒤 Pod를 만들면 제대로 Running 되는 것을 확인할 수 있다. 

또한 kubectl describe node 명령어를 통해 지정한 Node로 Scheduling 된 것을 확인할 수 있다.

![image4]()

만약 Pod를 구성할 때 다른 Labels로 구성하면 Pending이 되면서 Node에 할당이 되지 않는다.

![image5]()

![image6]()

![image7]()

Pending된 이유는 kubectl describe node 명령어에서 Events의 Message에서 확인할 수 있다.

# Node Affinity

Node Selector는 Node와 Pod 모두에게 Labels를 확실히 지정해주는 방법이다 보니 대규모 서버에서는 하나하나 지정해주기 어려운 상황이 있을 수 있다.

그러면서 등장한 것이 Node Affinity이다. Node Affinity는 Node에 Labels를 지정하고 Pod를 설정할 때 조건을 설정한다.

