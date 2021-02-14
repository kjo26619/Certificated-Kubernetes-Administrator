# Scheduling

Kubernetes에서는 다양한 Scheduling을 지원한다.

Manual Scheduling, Daemon Sets, Label & Selector, 등등 다양한 요소들의 Scheduling이 있다.

이러한 Scheduling들을 자세히 살펴본다.

# Manual Scheduling

Manual Scheduling은 Pod를 만들 때 직접 Node를 지정할 수 있는 방법이다.

방법은 매우 쉽다. YAML 파일을 작성할 때 spec 섹션에 nodeName 특성을 추가한다.

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/schedule1.PNG)

그리고 kubectl create -f 명령어를 이용하여 만들어내면 된다.

그리고 kubectl describe pods 명령어를 통해서 확인해보면 지정된 노드로 할당된 것을 볼 수 있다.

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/schedule2.PNG)

만약, 존재하지 않는 노드에 지정하게 되면 Pods는 Pending 되면서 제대로 실행되지 않는다.

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/schedule3.PNG)

![image4](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/schedule4.PNG)
