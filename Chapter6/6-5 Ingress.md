# Proxy Server

웹 서버를 제작하고 배포하면 URL 도메인에 대한 문제가 생긴다.

특히 여러 서비스를 가진 웹 서버일 경우 어떻게 서비스를 나누고 서비스에게 URL을 할당해줄 것인지 고민이 되기 마련이다.

예를 들어, 블로그와 카페, 쇼핑 등 많은 서비스를 가진 웹 사이트들은 blog.naver.com, cafe.naver.com, blog.daum.net, ... 등으로 나눈다.

이러한 서비스들이 서버 측에서도 나누어 져있을테고 Cloud-Native하고 Kubernetes를 사용한 설계라면 Pod로 구성이 되어있을 것이다.

그렇다면 이 Pod를 도메인에 연결하는 것은 꽤 곤란한 문제이다.

왜냐하면 Multi-Node 환경이면 여러 노드에게 Pod는 분산되어 있을 것이고 도메인을 모든 Pod IP에 연결할 수는 없을 것이다.

그래서 필요한 것이 Kubernetes Services이다.

Kubernetes Services의 Cluster IP는 내부에 있는 Pods를 연결한다. 그리고 Node Port는 외부에서 들어오는 트래픽과 Pod를 연결한다.

하지만, 여기서도 문제점이 Node Port는 포트 번호를 명시해야 된다. 예를 들어, http://<node-ip>:38080 같은 URL로 접속해야 한다.

그러나 DNS는 IP만을 반환해주는 서비스로 포트에 대해서는 반환해주지 않는다.

만약, 웹 서버의 도메인이 www.test-store.com 이라면 사용자는 www.test-store.com:38080 으로 접속해야할 것이다.

유일하게 포트 번호를 작성하지 않아도 되는 것은 HTTP와 HTTPS 즉, 80과 443이다. Node Port는 30000 이상의 포트를 사용하므로 이와는 관련이 없다.

그래서 나온 해결법이 Proxy-Server이다. 클러스터 외부에 Proxy-Server를 만들고 80이나 443포트와 Node Port와 연결해주면 된다.

그리고 이 Proxy-Server의 IP 주소를 DNS에 등록하면 사용자는 www.test-store.com 만 사용해도 웹에 들어올 수 있는 것이다.

# Cloud Platform

클라우드 플랫폼들이 등장하면서 새로운 방법이 제시되었다.

클라우드 플랫폼에서 지원하는 Load Balancer를 사용하는 것이다.

클라우드의 Load Balancer와 Kubernetes Services를 연결하면 클라우드 플랫폼에서 자동으로 요청을 라우팅해서 보내준다.

그리고 클라우드의 Load Balancer는 외부 IP를 가지고 있어서 이 외부 IP와 도메인을 연결해주면 된다.

하지만 이 Load Balancer는 한가지 문제점을 가지고 있다. 바로 Load Balancer 당 하나의 서비스만 연결할 수 있으므로 서비스를 늘리면 Load Balancer 수를 늘려야 한다.

즉, 웹 서버에 블로그만 있었는데 카페 기능도 늘렸다 하면 카페용 Load Balancer가 필요한 것이다. 그렇게 되면 카페용 Load Balancer에 또 다른 IP를 할당해주어야 한다.

이렇게 되면 너무 많은 유지 비용이 들어 버린다. 그래서 Load Balancer 위에 Proxy 서버나 Load Balancer를 다시 두어서 라우팅하는 방법이 있다.

하지만 이 방법은 매우 복잡하며 HTTPS 를 위한 SSL 적용, 방화벽 적용 등의 문제가 남아있다. 

이들을 여러 개의 Load Balancer에 각각 처리하는 것은 정말 힘든 일이기 때문이다.

# Ingress

이러한 문제로 Kubernetes에서는 외부 URL과 클러스터 내부에 요소들과 연결을 돕는 Ingress 기능을 추가하였다.

Ingress는 URL 기반 라우팅 규칙을 추가할 수 있고 부하 분산을 할 수 있으며 SSL, 권한 조정 등에 작업을 할 수 있다.

이 방법은 원래는 Reverse Proxy를 구축하여 할 수 있다. 하지만 Kubernetes에서 Ingress를 지원하면서 클러스터 내부에서 진행할 수 있게된 것이다.

Ingress는 총 2가지 부분이 존재한다. 바로 Ingress Controller와 Ingress Resource 이다.

Ingress Controller는 Reverse Proxy Server 자체이다. 트래픽이 들어오고 Node Port와 연결되는 Pod이다. 

그런데 Kubernets는 Ingress Controller를 기본적으로는 지원하지 않으며 서드 파티를 지원한다.

그 목록은 https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/ 에서 확인할 수 있다.

Ingress Resource는 라우팅 규칙을 설정하는 것이다. URL을 기반으로 www.test-store.com/blog, www.test-store.com/cafe 등으로 설정하거나

www.blog.test-store.com, www.cafe.test-store.com 등의 도메인 기반으로 설정할 수 있다.

라우팅 경로와 Kubernets 클러스터 내의 Service와 연결해주는 역할을 한다.

Ingress Resource는 Kubernetes의 API이며 YAML 파일로 구성된다. Ingress Controller는 Deployment로 구성된다.

# Ingress Controller

Ingress Controller는 서드 파티로 지원되며 Deployment로 구성된다.

그리고 Ingress Controller의 설정을 받고 수정하기 위해 Config Map을 구성한다.

그러고 나서 Ingress Controller Pod의 포트 80과 443를 열어주면 된다.

Ingress Controller Pod가 모두 배포되면 Services 중 Node Port를 이용하여 외부와 연결해주어야 한다.

이 Node Port는 외부에 존재하는 Proxy Server나 Cloud Load Balancer를 이용하여 도메인과 연결해주면 된다.

마지막으로, 이 Ingress Controller에 대한 사용 권한을 제한하기 위해 Service Account 등을 이용한 권한 설정을 해주면 된다.

여러 Controller를 지원하지만 대표적인 Nginx 설치 가이드는 https://kubernetes.github.io/ingress-nginx/deploy/ 에서 확인할 수 있다.

# Ingress Resource

Ingress Resource는 Kubernetes API여서 YAML 파일로 구성하면 된다.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-blog
spec:
  backend:
    serviceName: blog-service
    servicePort: 80
```

다음과 같이 기존 Services와 연결해주면 된다.

만약 URL이나 도메인 기반으로 Path를 나누고 싶다면 ( www.test-store.com/blog, www.blog.test-store.com ) rules를 추가해주면 된다.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-blog
spec:
  rules:
  - http:
      paths:
      - path: /blog
        backend:
          serviceName: blog-service
          servicePort: 80
      - path: /cafe
        backend:
          serviceName: cafe-service
          servicePort: 80
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-blog
spec:
  rules:
  - host: blog.test-store.com
      paths:
      - backend:
          serviceName: blog-service
          servicePort: 80
  - host: cafe.test-store.com
      paths:
      - backend:
          serviceName: cafe-service
          servicePort: 80
```

Ingress Controller와 Ingress Resource를 설정하면 Ingress 설정이 끝난 것이다.
