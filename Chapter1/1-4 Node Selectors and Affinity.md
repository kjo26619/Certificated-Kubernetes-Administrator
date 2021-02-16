# Node Selectors and Affinity

Taints와 Tolerations는 원하는 Node에 Pod를 배치하는 Scheduling 기법이 아니다. 잘못된 Node에 Pod를 배치하는 문제를 미연에 방지하기 위해 사용한다.

원하는 Node에 원하는 Pod를 배치하는 기법은 Node Selector와 Node Affinity이다.

Node Selector는 Node에 Labels를 정하고 Pod에 Node를 지정한다.

Node Affinity는 선호도로 어떠한 조건을 만족하는 Node에 Pod를 Scheduling할 것을 지정한다.

# Node Selectors



