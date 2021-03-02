# Kubernetes Security

Kubernetes에서 중심이 되는 kube-apiserver의 보안을 지키기 위한 다양한 방법이 존재한다.

Authentication, Authorization, TLS Certificates, Network Policy 이다.

# Authentication

Authentication 은 API 서버에 대한 접속 권한이다. Kubernetes Cluster에 대한 접속 권한 설정은 Static Files, Tokens, Certificates, LDAP 등으로 이루어진다.

그런데 Kubernetes에서는 kubectl를 이용하여 직접 User를 생성하고 관리할 수 없다. 외부 계정 시스템을 사용하거나 Service Account를 사용해야 한다.


