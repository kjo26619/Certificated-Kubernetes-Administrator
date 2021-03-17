# Authorization

Authorization은 각 사용자가 가질 수 있는 Kubernetes 클러스터에 대한 권한이다.

예를 들어 Kubernetes 클러스터 관리자는 모든 기능을 실행할 수 있다.

개발자의 경우에는 Pods만 배포하고 관리할 수 있을 뿐 나머지 네트워크/스토리지 설정 등에 간섭하지 못하게끔 권한을 갖는다.

그리고 아예 Kubernetes 클러스터와 관련이 없는 사람은 Kubernetes로의 명령에 대한 대답을 받을 수 없다.

이렇게 각 사용자마다의 권한을 달리 주어 Kubernetes의 보안을 관리하는 것이 Authorization이다.

Authorization에는 여러가지 메커니즘이 있으며 Node Authorization, Attribute-based Authorization, Role-based Authorization, Webhook 이다.

# Authorization - Node

Node Authorization은 Kubelet이 만든 API 요청을 승인할 때 사용하는 특수한 Authorization이다.

즉, 노드에 존재하는 kubelet이 kube-apiserver로부터 정보를 읽거나 쓰는 행위를 할 때는 모두 Node Authorization으로 처리된다.

이는 클러스터 내에서만 존재하며 kubelet와 kube-apiserver 간에 다음과 같은 일을 처리한다.

1. Read
  - Services
  - Endpoints
  - Nodes
  - Pods
  - Kubelet 노드에 할당된 Pods에 대한 Secrets, Configmaps, Persistent Volume
2. Write
  - Nodes & Node Status
  - Pods & Pod Status
  - Events
3. Auth-releated
  - TLS Bootstrapping을 위한 CSR API에 대한 Read/Write
  - 위임된 Authentication/Authorization 확인

# Attribute-based Authorization

Attribute-based Autrhoziation은 Attribute-based Access Control이라 하여 ABAC라고 불린다.

ABAC는 속성(Attribute)를 종합한 Policy라는 것을 만든 다음 이에 대한 권한을 유저에게 주는 방법이다.

예를 들어, User1에게 어느 Namespace의 Pods를 관리할 수 있는 권한을 주겠다. 라는 Policy를 만들면 그 유저는 그 권한만 가질 수 있다.

ABAC의 Policy는 JSON 파일로 구성할 수 있다.

```
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1", 
  "kind": "Policy", 
  "spec":
    { "user": "alice",
      "namespace": "*",
      "resource": "*",
      "apiGroup": "*" }
}
```

정확한 사항은 https://kubernetes.io/docs/reference/access-authn-authz/abac/ 에서 확인할 수 있다.

ABAC의 경우에는 각 사용자마다 권한을 부여하기 위해 Policy를 계속 만들어야 하며 Policy 수정 또한 번거롭다는 단점이 있다.

그리고 권한을 부여하기 위해서는 kube-apiserver를 재시작해야 된다.

이러한 단점으로 ABAC의 경우에는 복잡하고 만들기 어렵다. 그래서 Role-based Authorization이 선호된다.

# Role-based Authorization

Role-based Authorization은 Role-based Access Control이라 하여 RBAC라고 불린다.

RBAC는 Role을 만들어내고 이를 사용자에게 연결을 하는 방법이다. 

예를 들어, Pods를 관리할 수 있는 Role(생성, 삭제, 확인)을 만든 다음에 필요한 사용자에게 이를 연결시킨다.

이렇게 하면 같은 Role을 가진 사용자들을 쉽게 관리할 수 있다. Role을 변경하게 되면 연결되어 있는 모든 사용자에게 적용이 된다.

# Webhook

Kubernetes는 기본적으로 위 3가지의 Authorization을 사용하지만, Kubernetes의 바깥에 존재하는 Authorization 어플리케이션으로 처리할 수 있게끔 설계되어 있다.

kube-apiserver는 사용자에게 요청을 받으면 먼저 다른 Autrhozation 어플리케이션에게 사용자 권한을 요청한다.

그리고 사용자가 권한을 가지고 있는지 여부를 확인한 후 응답을 보내는 방법으로 구성되어 있다.

kube-apiserver의 설정을 바꾸어주면 된다.

# kube-apiserver Setting

총 4가지의 방법외에도 기본적으로 제공되는 방법이 2가지가 있다. 그것은 AlwaysAllow와 AlwaysDeny이다. 

이름에서 알 수 있듯이 모두 접속이 가능하거나 모두 접속이 불가능하다 이다.

이러한 Authorization을 설정하기 위해서는 kube-apiserver 설정에서 --authorization-mode를 바꾸어 주면 된다.

![image1]()

이 --authorization-mode는 여러개로 설정할 수 있다.

만약 여러개의 방법으로 설정한다면 체인이 적용이 되며 순서대로 Authorization을 확인한다.

Node, RBAC, Webhook 으로 설정을 해놓았다면 Node부터 Webhook까지 모두 Authorization을 확인한다.

먼저, 사용자 요청이 들어오면 Node Authorization에서 확인을 한다. 

하지만 Node Authorization은 클러스터 내부의 노드와 kube-apiserver간에 요청을 처리하므로 거부하고 RBAC로 넘긴다.

다음 RBAC에서 사용자의 Role을 확인한 뒤 권한이 맞으면 응답을 하고 아니면 Webhook으로 넘긴다.

역시 Webhook에서도 똑같이 확인한 뒤 맞으면 응답, 아니면 다음으로 넘긴다. 그런데 Webhook이 최종이므로 Deny하여 응답을 보낸다.
