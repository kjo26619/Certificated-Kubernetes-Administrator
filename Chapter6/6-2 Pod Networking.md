# Pod Networking

6-1에서 설명한 바와 같이 Kubernetes에서는 CNI 플러그인을 사용한다.

kubenet 이라고 하는 기본적인 네트워크 플러그인을 제공해주긴 하지만 고급 기능들이 설정되어 있지 않아서 많이 사용하지 않는다.

Kubernetes에서는 모든 노드에 있는 Pod가 고유의 주소를 가지고, 서로 식별이 가능하여서 통신이 가능하기를 원한다.

이러한 네트워크 솔루션은 무수히 많다. 솔루션 하나를 콕 찝어서 설명하기란 애매하고 지원하는 기능들이 모두 다르기 때문에 큰 개념만을 소개한다.

각 노드에는 가상의 Bridge 네트워크를 갖는다. 이 Bridge에는 Virtual Ethernet(veth)를 만들 수 있다. 

각 Pod는 이 Bridge에 속해서 veth를 받게 되며 통신할 수 있다. 

노드 간 통신은 노드 외부에 라우터가 존재하며 각 노드에 대한 내용을 라우터가 라우팅 테이블로 가지고 있어서 가능하게끔 한다.

이제 이 Bridge 네트워크 구성, veth 구성, 외부 라우터 구성을 진행해주면 Pod간에 통신이 가능하다.

가장 중요한 점은 Pod가 모두 수행했을 때 veth를 해체하고 Pod를 종료해주어야 한다.

하지만 이것을 수동으로 하고 있으면 대규모 환경에서는 절대 처리하지 못할 것이다.

그래서 이제 CNI 플러그인을 사용한다.

CNI 플러그인에 있는 스크립트가 Brdige 네트워크를 구성해주고 Pod가 생성될 때 마다 veth를 구성하고 연결해준다.

그리고 각 노드에 있는 Pod들을 식별할 수 있도록 라우터를 구성한다.

마지막으로, Pod가 제거되면 veth를 제거해준다.

# CNI Plugins

CNI 플러그인은 kubelet에 위치해있다. 즉, 각 노드마다 CNI 플러그인을 가지고 있다.

ps 명령어를 통해 kubelet을 확인해보면 cni 설정이 어떻게 되어 있는지 확인해볼 수 있다.

```
# ps -aux | grep kubelet
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/cni1.PNG)

kubelet의 네트워크 플러그인이 CNI인지, cni의 bin과 conf 디렉토리를 확인해볼 수 있다.

bin 디렉토리를 확인해보면 실행 가능한 CNI 플러그인들이 모두 나열되어 있다. 

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/cni2.PNG)

그리고 conf 디렉토리를 확인해보면 conf 파일이 있다. 이 것은 kubelet이 어떤 플러그인을 사용하는지 지정해놓은 파일이다.

파일이 여러개일 경우 알파벳 순으로 하나를 사용한다.

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/cni3.PNG)

CNI 네트워크의 이름, IP 주소 대역, Gateway 사용 여부 등을 알 수 있다.
