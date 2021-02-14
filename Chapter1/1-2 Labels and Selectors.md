# Labels and Selectors

Labels와 Selectors는 구분을 위한 기능들이다. 

Labels는 이름, 종류 등을 지정하는 것이다.

그리고 Selectors는 원하는 Labels를 찾기 위한 방법이다.

쉽게 영화를 예시로 들어서 확인해볼 수 있다.

영화는 엄청나게 다양하며 이를 구분하기 위한 다양한 Labels가 있다. 영화의 이름, 장르, 별점, 나이 제한, 제작사 등이 있다.

그렇다면 이제 우리가 원하는 영화를 찾을 때 사용하는 것이 Selectors이다.

포함되는 이름이 무엇인지, 장르가 무엇인지, 별점은 몇 이상인지, 나이 제한은 몇 살부터 인지, 제작사는 어디인지 등등을 지정할 수 있다.

예를 들어 장르는 스릴러, 나이 제한은 15살로 지정하거나 장르는 애니메이션, 이름은 겨울 왕국 이렇게 다양하게 지정할 수 있다.

이렇게 Labels와 Selectors는 Kubernetes에서 다양한 객체들(Pods, Services, ReplicaSets, Deployments)을 구분할 수 있게끔 되어 있다.

Back-End 용 객체, Front-End 용 객체, Web-Serever 용 객체 등으로 나눌 수도 있고 어플리케이션에 따라 나눌 수도 있다.

# Define Labels and Selectors

Labels와 Selectors는 YAML 파일에서 지정할 수 있다.

Pods의 Labels는 metadata 섹션에서 labels 특성을 추가해주면 된다. 

그리고 Selectors는 Services, ReplicaSets, Deployments 에서 지정하며 spec 섹션에서 지정하면 원하는 Selectors를 구성할 수 있다.

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label1.PNG)

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label2.PNG)

두 Pod의 차이는 labels에서 function이 다르다. 

Selectors를 쓰는 곳은 명령어에서도 사용할 수 있으며 --selector 옵션을 붙이면 된다.

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label3.PNG)

--selector 옵션을 이용해서 Pods의 리스트를 보여줄 때 자신이 원하는 Selectors의 리스트를 받을 수 있다.

그리고 ReplicaSets에서 본격적으로 사용이 된다.

![image4](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label4.PNG)

![image5](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label5.PNG)

만들었던 Pods를 지운 뒤 ReplicaSet을 위와 같이 추가한다.

ReplicaSets도 모두 labels를 지정해줄 수 있다. 그리고 spec에서 selector를 지정할 때 App1과 App2로 차이를 두었다.

ReplicaSets은 selector를 통하여 자신이 관리할 Pods를 구분한다. 만약, 기존에 Pods를 지우지 않았다면 App1 ReplicaSet의 Pods는 기존 Pods로 지정되어 관리하는 것을 확인할 수 있다.

ReplicaSets을 만든 뒤 --selector 옵션으로 지정해서 확인해볼 수 있다.

![image6](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label6.PNG)

그리고 kubectl get 명령어에서 --show-labels 옵션을 추가하면 각 객체가 가지고 있는 Labels를 확인해볼 수 있다.

![image7](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter1/Image/label7.PNG)

# Annotations

metadata 섹션에는 Labels와 유사한 Annotaions이 있다.

하지만 Annotaions는 객체를 구분하기 위해 사용하지 않는다.

Annotaions는 빌드 버전, Contact 정보 등의 자세한 내용을 기술하며 Labels보다 더 많은 문자를 사용할 수 있다.
