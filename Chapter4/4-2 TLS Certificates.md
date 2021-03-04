# TLS Certificates

TLS(Transport Layer Security)는 보안 통신을 위한 프로토콜이다. 

인증을 하기 위해 아이디와 패스워드를 서버에게 보낼 때 당연히 메시지가 공격자에게 들어갈 수 있다.

만약 평문으로 메시지를 보낸다면 공격자는 탈취한 메시지를 통해 아이디와 패스워드를 마음껏 사용할 수 있다.

이런 점을 방지하기 위해 송신자와 수신자는 대칭키와 공개키 암호를 사용한다.

대칭키는 송신자와 수신자 간에 같은 암호화 키를 나눠가진 뒤 암호화하여 통신하는데 사용한다.

공개키는 개인키-공개키 쌍으로 이루어진 암호화 키를 이용하는 방식이다. 개인키로 암호화된 메시지는 공개키로, 공개키로 암호화된 메시지는 개인키로 복호화할 수 있다.

공개키 방식은 수신자가 개인키-공개키를 생성하면 공개키를 저장소에 공개한다. 그리고 송신자가 공개키로 암호화해서 보내면 수신자는 개인키로 복호화할 수 있다.

두 방식 모두 단점이 있는데 대칭키의 경우 암호화 키를 공격자에게 탈취당하면 모든 보안이 무의미해진다는 단점이 있다. 

그리고 공개키는 대칭키에 비해 속도가 느리다는 점이다.

이러한 단점을 없애기 위해 TLS 인증서를 이용해서 공개키 방식으로 대칭키를 교환한다.

1. 서버와 클라이언트가 서로 통신을 하기 전에 클라이언트는 서버에게 인증 요청을 보낸다. (TLS 버전, 설정 등이 포함)
2. 인증 요청을 받은 서버는 통신이 가능할 경우 응답을 보낸다. (TLS 버전, 설정 등이 포함)
3. 서버가 인증서를 보낸다. 서버는 자신의 공개키를 포함하여 보낸다.
4. 클라이언트는 서버에게서 받은 인증서를 검증한다. 이러한 검증을 위해 CA(Certificate Authority)에서 서버의 인증서를 검증하고 저장해놓는다.
5. 클라이언트는 받은 공개키를 이용해서 암호화된 메시지를 보낸다. 여기에 대칭키를 만들기 위한 정보가 들어있다.
6. 서버는 개인키로 메시지를 복호화한 뒤 대칭키를 만들어낸다. 클라이언트 역시 대칭키를 만든다.
7. 대칭키를 이용하여 통신한다.

TLS 인증서는 이렇게 통신의 보안을 유지하며 클라이언트가 공격자의 피싱 등에 방지할 수 있도록 돕는다.

공격자는 자신의 인증서를 만들어내도 CA에 의해 검증받지 못하게 된다. 즉, 클라이언트 측에서는 TLS 인증서 검증으로 공격자의 피싱 등을 확인할 수 있다.

# TLS Certificates in Kubernetes

Kubernetes에서는 모든 구성 요소 간에 통신에서 보안 유지를 위해서 TLS 인증서를 사용한다.

따로, 유저 정보를 저장하는 것이 아닌 인증서로 Kubernetes에 권한이 있는지 확인을 한다. 

kubeadm을 사용하면 자동으로 X.509 인증서가 생성된다.

가장 먼저 Kubernetes의 Root CA 키와 인증서를 생성한다. 이는 /etc/kubernetes/pki/ca.key 와 ca.crt 로 구성된다.

그 후 서버측에 Root CA의 인증서를 통해 kube-apiserver, etcd의 인증서를 생성한다.

kube-apiserver의 인증서는 /etc/kubernetes/pki/apiserver.crt 로 구성된다. etcd는 /etc/kubernetes/pki/etcd/ 디렉토리에 새로운 CA에 대해 구성된다.

이는 etcd 역시 관련 기능을 사용하기 위해서는 인증서를 사용하기 때문에 자신의 CA를 구성한다. 

이들의 정확한 정보를 알고 싶다면 openssl 명령어를 사용해서 인증서를 확인하면 된다.

```
# openssl x509 -in (CERTIFICATES FILE) -text
```

이를 통해 인증서의 Common Name, CA Name, Alternate Names, 만료일 등을 확인할 수 있다.

만료일의 경우 CA 인증서는 10년이며 하위 인증서들은 1년이다.

이러한 인증서의 위치는 모두 Kubernetes의 Static Pods를 만들 때 구성된다. 즉, /etc/kubernetes/manifests 에 있는 YAML 파일에서 결정한다.

그리고 이제 클라이언트 인증서들이 필요하다.

1. kubelet에서 API 서버 인증서를 인증 시 사용하는 클라이언트 인증서
2. API 서버에 클러스터 관리자 인증을 위한 클라이언트 인증서
3. API 서버에서 kubelet과 통신을 위한 클라이언트 인증서
4. API 서버에서 etcd 간의 통신을 위한 클라이언트 인증서
5. controller manager와 API 서버 간의 통신을 위한 클라이언트 인증서 / kubeconfig
6. scheduller와 API 서버 간의 통신을 위한 클라이언트 인증서 / kubeconfig
7. front-proxy를 위한 클라이언트와 서버 인증서 (kube-proxy에서 API 서버 확장을 지원할 때만 필요)

모두 kubeadm에 의해 자동으로 구성되며 각 구성요소들은 인증서를 통해 보안 통신을 할 수 있다.

정확한 사항은 https://kubernetes.io/docs/setup/best-practices/certificates/#all-certificates 에서 확인할 수 있다.

수동으로 키와 인증서를 만든다면 위를 꼭 참고해야 한다.

# Certificates API

kubeadm 이 자동으로 인증서들을 구성하거나 수동으로 모두 구성하고 나면 이제 Kubernetes Cluster는 문제없이 실행되고 있다.

만약, kube-apiserver 나 etcd의 응답이 이루어지지 않을 경우 인증서 만료나 YAML 파일의 인증서 구성이 잘못되어 있을 수 있다.

그 다음 Kubernetes를 관리자가 Control Plane의 로컬에서 혼자 사용한다면 큰 문제는 없다.

하지만 외부의 다른 사람이 Kubernetes에 접속해서 관리를 해야한다면 접속을 위해 인증서를 발급해야 한다.

모든 인증서의 발급은 Kubernetes의 Root CA가 맡는다. 관리자에게 인증서 발급 요청이 오면 관리자는 Kubernetes의 Root CA를 이용하여 발급해줄 수 있다.

즉, Root CA의 인증서와 키는 매우 안전한 곳에 존재하고 반드시 안전하게 관리해야 한다.

kubeadm의 경우에는 이를 Control Plane에게 저장하며 Control Plane이 CA 서버가 된다.

CA 인증서를 이용해서 만료될 때마다 필요한 인증서들을 발급할 수 있다. 하지만, 이걸 매번 하기엔 비효율적이므로 Certificates API 를 지원한다.

인증서 요청이 들어오면 서버의 Certificates API를 통해 인증서를 발급하는 자동화가 가능하다.

이를 CertificateSigningRequest(CSR)이라고 하며 관리자가 CSR 객체를 만들 때 Kubernetes에서의 권한 부여 및 갱신을 설정할 수 있다.

먼저 이를 위해 새로운 Kubernetes 사용자가 자신의 키를 통해 CSR 파일을 만들어야 한다.

키와 CSR파일은 openssl 을 이용해 만들 수 있다.

```
# openssl genrsa -out (KEY FILE) 2048

# openssl req -new -key (KEY FILE) -subj "/CN=(NAME)" -out (CSR FILE)
```

키 파일과 CSR 파일을 만들었다면 CSR 객체를 만들어야한다.

CSR 객체를 만들 때에는 YAML 파일을 이용한다.

```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: (USER NAME)
spec:
  groups:
  - system:authenticated
  request: (ENCODED USER CSR USING BASE64)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

request 부분은 반드시 base64로 인코딩된 CSR 파일 전문을 넣어야한다. 이는 cat (CSR FILE) | base64 를 이용하면 된다.

groups는 ClusterRole로 기본 그룹은 https://kubernetes.io/docs/reference/access-authn-authz/rbac/#discovery-roles 로 정의되어 있다.

signerName은 서명자를 지정하는 것으로 어느 구성 요소에게 서명을 받을지 지정하는 것이다. 

이미 Kubernetes에서 만든 signer는 https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#kubernetes-signers 에서 확인할 수 있다.

usages는 X.509의 Key Usage로 공개키에서 사용될 보안 서비스들을 지정한다.

YAML 파일을 통해 CSR을 만들면 kubectl get csr 명령어로 현재 CSR을 확인할 수 있다.

```
# kubectl get csr
```

처음 CSR은 무조건 Pending 상태를 유지하는데 이는 Kubernetes에서 승인이 되지 않았기 때문이다.

승인이나 거절을 하기 위해서는 kubectl certificate 명령을 사용하면 된다.

```
승인
# kubectl certificate approve (CSR NAME)

거절
# kubectl certificate deny (CSR NAME) 
```

그리고 kubectl get csr을 할 때 CSR의 이름을 지정하고 YAML 파일로 내보낸다면 CSR 설정들을 확인할 수 있다.

```
# kubectl get csr (CSR NAME) -o yaml
```

현재 지정된 권한이나 상태를 확인할 수 있다.
