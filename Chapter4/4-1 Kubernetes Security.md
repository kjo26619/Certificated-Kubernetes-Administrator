# Kubernetes Security

Kubernetes에서 중심이 되는 kube-apiserver의 보안을 지키기 위한 다양한 방법이 존재한다.

Authentication, Authorization, TLS Certificates, Network Policy 이다.

# Authentication

Authentication 은 API 서버에 대한 접속 권한이다. 

Kubernetes Cluster에 대한 접속 권한 설정은 Static Password Files, Static Token Files, Certificates, LDAP 등으로 이루어진다.

그런데 Kubernetes에서는 kubectl를 이용하여 직접 User를 생성하고 관리할 수는 없다. 외부 계정 시스템을 사용하거나 Service Account를 사용해야 한다.

먼저, 외부에서 사용하는 가장 쉬운 방법인 Static Password Files와 Static Token Files를 살펴본다.

# Static Password & Token Files

Static Password & Token Files는 그냥 외부 파일 시스템에 Password, User Name, User ID, Group 혹은 Token을 CSV파일로 저장한 뒤

API 서버에서 이 CSV 파일을 Authentication에 사용한다고 하면 된다.




