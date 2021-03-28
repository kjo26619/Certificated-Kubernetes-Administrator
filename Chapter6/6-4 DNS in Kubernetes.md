# DNS in Kubernetes

Kubernetes에서 DNS는 클러스터 내에 DNS가 존재하며 Services와 Pod의 정보를 가지고 있고 요청이 들어오면 IP를 반환해주는 역할을 한다.

정확히 말해서는 클러스터 내에 존재하는 Pod, Serivces들의 이름으로 통신이 가능하게끔 해준다.

예를 들어서, test와 web 이라는 Pod가 존재하고 kube-proxy에 web을 연결하는 web-service라는 Service가 있다고 가정한다.

test에서 web으로 요청을 보낼 때 http://web-service 로 요청을 보내면 DNS에서 web-service에 대한 IP를 반환해준다.

하지만, 여기서 유의할 점은 Kubernetes는 클러스태 내에 가상 클러스터인 Namespace가 존재하다는 것이다.

# DNS for Namespace

Pods나 Services는 하나의 노드에 존재하며 클러스터 내에서도 Namespace로 쪼개질 수 있다.

그렇기 때문에 Kubernetes DNS는 서브 도메인을 만들고 그에 대한 정보를 저장해 놓는다.

먼저 test는 default Namespace에 있고 web과 web-service는 apps Namespace에 존재한다고 가정한다.

그렇다면 Kubernetes의 DNS는 저장할 때 다음과 같이 저장한다.

```
web-service | apps | svc | cluster.local
```

- web-service : Host의 이름. Pod나 Services의 이름이다.
- apps : Host가 위치한 Namespace의 이름
- svc : Host의 Type이다. svc나 pod가 된다.
- cluster.local : Root 도메인으로 기본은 cluster.local이다.

이 정보를 URL로 만들며 다음과 같이 이루어진다.

```
(Host Name).(Namespace).(Type).(Root Domain)
```

즉, apps의 web-service는 web-service.apps.svc.cluster.local 이라는 URL을 갖게 되는 것이다.

test에서 다른 Namespace의 web으로 연결하기 위해서는 curl http://web-service.apps.svc.cluster.local 을 사용하면 된다.

그런데 Pod의 경우에는 Host Name을 이름으로 저장해놓지 않는다. IP로 저장을 해둔다.

web Pod가 10.244.2.5라는 IP를 할당받았으면, 10-244-2-5.apps.pod.cluster.local 이라는 URL을 갖게 된다.

# Core DNS

Core DNS는 위와 같이 클러스터 중앙에서 DNS 요청을 처리해주는 Pod를 의미한다.

Core DNS는 kube-system Namespace에 Pod로 배포가 된다. 실제로는 Replicaset으로 배포되지만 여러개의 Pod를 갖는다는 의미이며 Replicaset에 의해 Pod 수가 관리된다.

그래서 kubectl get pods 명령을 kube-system Namespace로 사용해보면 coredns Pod를 확인해볼 수 있다.

```
# kubectl get pods -n kube-system
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns1.PNG)

그리고 Core DNS는 설정이 존재하는데 이는 Config Map으로 구성되어 있다.

이에 대해서 확인하기 위해서는 kubectl get configmaps 명령과 kubectl describe configmaps 명령을 사용하면 된다.

```
# kubectl get configmaps -n kube-system

# kubectl describe configmaps -n kube-system coredns
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns2.PNG)

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns3.PNG)

이 Config Map은 Core DNS Pod에게 연결된다. Config Map을 수정하면 Core DNS의 설정을 수정하는 것이다.

이 설정에 있는 Corefile 밑에 있는 errors, health, ready, kubernetes들은 플러그인이며 이에 대한 자세한 사항은 https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#coredns-configmap-options 에서 확인할 수 있다.

Corefile의 맨 앞부분은 DNS 용 포트인 53 포트를 의미하는 것이다.

Corefile의 kubernetes는 기본 도메인 이름을 설정할 수 있으며 그것이 cluster.local이다.

그리고 forward는 도메인 내에 없는 쿼리의 경우 어느 Nameserver를 기반으로 할 것인지 정하는 것으로 /etc/resolv.conf에 설정된 값을 가지고 온다.

Core DNS가 구성되면 Pods나 Services 가 생성/삭제될 때마다 자동으로 Core DNS의 내용을 갱신한다. 

이러한 작업은 kubelet이 하며 kubelet의 설정파일을 확인해보면 Core DNS에 대한 내용이 있는 것을 확인할 수 있다.

![image4](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns4.PNG)

그리고 Core DNS에 대한 연결도 Services를 가지고 있으며 kubectl get services 를 통해서 확인해 볼 수 있다.

```
# kubectl get services -n=kube-system
```

![image5](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns5.PNG)

53 포트를 사용하며 kube-dns라는 이름을 가지고 10.96.0.10이라는 IP를 갖는다.

이제 Pod 내부에서 DNS 요청을 보내보면 10.96.0.10에서 이루어지는 것을 확인해 볼 수 있다.

![image6](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter6/Image/dns6.PNG)
