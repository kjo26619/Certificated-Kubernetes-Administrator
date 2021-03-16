# API Groups

Kubernetes에서의 모든 자원들은 API 그룹으로 나뉘어져 있다.

우리가 아는 Pods, Replicasets, Statefulsets, Deployments, ... 등등 모두 그룹으로 나뉘어져있으며 트리 구조를 갖는다.

Kubernetes의 트리 구조의 가장 상위에는 /가 있으며 그 하위에는 /metrics, /healthz, /version, /api, /apis, /logs 가 있다.

그 중에서 /api 와 /apis가 Kubernetes의 많은 기능을 담당하는 그룹이라고 할 수 있다.

# API Groups - /api and /apis

/api 는 Core 그룹이다. Core 그룹은 Kubernetes를 구성하는 핵심 기능들이 있다.

예를 들어서 Pods, Namespaces, Replication Controllers, Events, Endpoints, Nodes, Services 등이다.

/apis는 Named 그룹이다. 더욱 체계적으로 사용하기 위해 구성된 기능이며 최신 기능들은 대부분 /apis 에 속해있다.

여기에는 /apps, /extensions, /networing.k8s.io, /storaget.k8s.io, /authentication.k8s.io, /certificates.k8s.io 등이 있다.

/apps에는 많이 사용했던 Deployments, Replicasets 등이 있다. /networking.k8s.io에는 Network Policy가 있고 /certificates.k8s.io 에는 CSR이 있다.

대부분 기능들은 v1이라는 하위 폴더에 존재한다. 

그렇기 때문에 YAML 파일을 구성할 때 apiVersion에 이 하위 폴더를 작성한다. /apps/v1 이나 v1 을 쓰는 것이 그 이유이다.

이 Deployments, Replicasets 같은 Resource 밑에 Verb라고 하는 명령들이 존재하며 kubectl에서 이 명령들을 가지고 와서 사용하는 것이다.

각 API 그룹에 대한 자세한 내용은 https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/ 에서 확인할 수 있다.

# Curl Command & Kubectl Proxy

curl 명령을 이용해서도 API 그룹을 확인해볼 수 있다.

Control Plane에게 6443 포트로 curl 명령을 보내면 API 그룹의 경로들을 확인할 수 있다.

```
curl http://(CONTROL PLANE ADDRESS):6443 -k
```

하지만 curl을 보낼 때에는 Kubernetes에서 인증된 인증서와 키가 필요하다. 즉, 인증서와 키를 지정해주어야 한다.

```
curl http://(CONTROL PLANE ADDRESS):6443 -k
  --key (KEY FILE)
  --cert (CERTIFICATE FILE)
  --cacert (ROOT CERTIFICATE FILE)
```

만약, 지정하지 않고 보내면 다음과 같이 거부당한다.

![image1]()

그래서 kubectl proxy 명령어를 이용해서 Proxy를 만들어 주면 된다.

```
kubectl proxy
```

![image2]()

![image3]()

여기서 주의할 점은, 어차피 Localhost 내에서 사용할 예정이면 --address와 --accept-hosts 옵션은 사용하지 않아도 된다.

이 두개의 옵션은 다른 곳에서 Proxy를 사용하기 위해 지정하는 것이다. address는 Proxy가 사용할 주소, accept-hosts는 누가 Proxy에 접근 가능한지 지정하는 것이다.

이미지의 설정은 Worker Node가 Proxy를 사용할 수 있도록 한 것이다.

참고로 kube-proxy와 kubectl proxy는 다른 것이다. 

kubectl proxy는 HTTP Proxy로 kubectl utility가 kube-apiserver에 접속하기 위한 서비스이다.

