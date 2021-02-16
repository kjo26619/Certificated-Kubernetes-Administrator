# Resource Requirements and Limits

Kubernetes Cluster에 포함된 Node들이 아무리 높은 스펙의 하드웨어를 사용한다 해도 한계는 분명히 존재한다.

Pod를 배치할 때 한 Node가 가진 Resource들이 한계치를 넘어설 경우 다른 Node에게 배치할 수 있도록 Kubernetes Scheduller가 관리한다.

그런데 이 Pod가 불필요하게 많은 Resource들을 차지할 수 있다. 그래서 Kubernetes에서는 각 Pod가 가질 수 있는 Resource를 제한할 수 있다.

또한, Pod가 사용할 최소한의 Resource를 정해줄 수도 있다.

각 Pod가 너무 많은 CPU, Memory, Disk 들을 가질 수 없게끔 제한하고 최소 필요한 Resource를 정해줄 수 있다.

# Define Requirements and Limits

Resource에 대한 Requirements와 Limits를 정하는 것은 크게 어렵지 않다.

YAML 파일에서 spec 섹션에 각 container에게 필요한 resources를 추가해주면 된다.

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/requirement1.PNG)

resources 밑에 필요한 Requirements를 requests로 추가하며 Limits는 limits로 추가하면 된다.

여기서 CPU는 단위가 가상 CPU 즉, vCPU이다. 그리고 Memory나 Disk 같은 경우에는 K, M, G와 Ki, Mi, Gi로 나뉜다.

K/ M/ G는 1,000bytes/ 1,000,000bytes/ 1,000,000,000bytes 를 뜻한다.

Ki/ Mi/ Gi는 1,024bytes/ 1,048,576bytes/ 1,073,741,824bytes를 뜻한다.

설정된 Resource Requirements와 Limits를 확인하려면 kubectl describe pod 명령어를 사용하면 된다.

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/requirement2.PNG)

만약 Pod가 제한된 Resource를 사용해야할 경우 Pending 된다.
