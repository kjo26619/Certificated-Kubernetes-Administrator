# Resource Requirements and Limits

Kubernetes Cluster에 포함된 Node들이 아무리 높은 스펙의 하드웨어를 사용한다 해도 한계는 분명히 존재한다.

Pod를 배치할 때 한 Node가 가진 Resource들이 한계치를 넘어설 경우 다른 Node에게 배치할 수 있도록 Kubernetes Scheduller가 관리한다.

그런데 이 Pod가 불필요하게 많은 Resource들을 차지할 수 있다. 그래서 Kubernetes에서는 각 Pod가 가질 수 있는 Resource를 제한할 수 있다.

또한, Pod가 사용할 최소한의 Resource를 정해줄 수도 있다.

각 Pod가 너무 많은 CPU, Memory, Disk 들을 가질 수 없게끔 제한하고 최소 필요한 Resource를 정해줄 수 있다.

# Resource Requirements

Resource에 대한 Requirements를 정하는 것은 크게 어렵지 않다.

YAML 파일에서 spec 섹션에 각 container에게 필요한 resources를 추가해주면 된다.

![image1]()

