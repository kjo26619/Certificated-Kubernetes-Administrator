# KubeConfig

Kubernetes 클러스터에서 정보를 얻을 때에는 kube-apiserver로 키와 인증서를 보낸 뒤 kube-apiserver가 이를 검증하고 결과를 반환한다.

정확하게는 curl 명령어나 kubectl 명령어를 이용하여 kube-apiserver에게 명령어, 키, 인증서를 보내는 것이다.

```
# curl https://my-kube-apiserver:6443/api/v1/pods \
  --key admin.key --cert admin.crt --cacert ca.cert
  
# kubectl get pods \
  --server my-kube-apiserver:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```

하지만 이러한 키, 인증서를 명령어에 계속 붙이는 것은 사용자 입장에서는 매우 불편하다.

그래서 Kubernetes에서는 이러한 정보들을 저장하고 가져와서 사용할 수 있는 Configuration 파일을 지원한다.

이것이 바로 KubeConfig이다.

# Default KubeConfig

기본적인 KubeConfig는 $HOME/.kube/config 로 구성이 되어있다.

이는 kubeadm으로 구성했으면 ls -al 명령어로 찾을 수 있다.

내용을 확인해보면 3가지 섹션이 존재한다는 것을 알 수 있다.

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter4/Image/kubeconfig1.PNG)

3가지 섹션은 cluster, context, user이다.

cluster는 Kubernetes 클러스터를 의미한다. 예를 들어, Development cluster, Production cluster 등등.. 각 클러스터에 대한 정보가 적혀있다.

user는 클러스터에 접속하는 사용자를 의미한다. 즉, 키와 인증서를 의미한다. 접속하려는 클러스터에서 사용하는 인증서와 키의 내용이 적혀있다.

마지막으로 context는 user와 cluster를 잇는 방법이다. 어느 사용자의 키와 인증서를 어느 클러스터에 보낼 것인지 적혀있다.

그래서 context의 이름은 (user name)@(cluster name)으로 적는다.

모든 KubecConfig 설정이 끝나면 이제 클러스터에 명령을 보낼 때 따로 옵션을 추가할 필요가 사라진다.

정확한 KubeConfig의 YAML 파일은 다음과 같다.

```
apiVersion: v1
kind: Config
current-context: my-kube-admin@my-kube-apiserver
clusters:
- name: my-kube-apiserver
  cluster:
    certificate-authority: ca.crt
    server: https://my-kube-apiserver:6443
contexts:
- name: my-kube-admin@my-kube-apiserver
  context:
    cluster: my-kube-apiserver
    user: my-kube-admin
users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin-key
```

clsuter, context, user가 모두 Array로 구성되어 있다. 그러므로 여러 개의 클러스터를 구성하고 여러 명의 사용자를 만든 뒤, 여러 개의 context를 구성할 수 있다.

그리고 기본적으로 사용할 context를 지정할 때에는 current-context에 context 이름을 적어주면 된다. 그렇다면 모든 kubectl 기본 명령은 이 context를 기반으로 실행된다.

그리고 인증서와 키를 적을 때, certificate-authority, client-certificate, client-key 를 사용하면 Control Plane의 파일 시스템 내에 인증서 경로를 적어주면 된다.

하지만 인증서 경로를 사용하고 싶지 않다면 certificate-authority-data, client-certificate-data, client-key-data 와 같이 뒤에 data를 추가한다음 인증서나 키 내용 자체를 넣어주면 된다.

현재 kubectl이 사용하는 KubeConfig를 보고 싶다면 kubectl config view를 이용하면 된다.

```
# kubectl config view
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter4/Image/kubeconfig2.PNG)

그리고 현재 kubectl이 사용하는 KubeConfig를 변경하기 위해서는 kubectl config use-context 명령어를 이용하면 된다.

```
# kubectl config use-context (CONTEXT NAME)
```

만약 명령어에서 아예 다른 KubeConfig를 사용하고 싶다면 뒤에 --kubeconfig 옵션을 붙여주면 된다.

```
# kubectl get pods --kubeconfig (KUBECONFIG FILE)
```

# KubeConfig for Multiple Namespace 

Kubernetes의 클러스터들은 모두 내부에서 Namespace 라는 이름으로 가상 클러스터로 나뉘어질 수 있다.

KubeConfig에서는 이러한 기본 Namespace를 지정할 수 있는 기능을 가지고 있다.

context 섹션에 namespace 를 추가해주면 된다.

```
apiVersion: v1
kind: Config
current-context: my-kube-admin@my-kube-apiserver
clusters:
  ...
contexts:
- name: my-kube-admin@my-kube-apiserver
  context:
    cluster: my-kube-apiserver
    user: my-kube-admin
    namespace: (NAMESPACE NAME)
users:
  ...
```

