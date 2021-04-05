# Network Troubleshooting

네트워크 관련된 Troubleshooting은 세 가지이다. 세 가지는 CNI 플러그인, Core DNS, Kube-proxy이다.

CNI 플러그인은 자신이 원하는 플러그인을 선택해서 설치하는 만큼 Troubleshooting도 다양해진다.

그래서 설치한 CNI 플러그인의 Docs로 들어가서 확인해야 한다.

CNI 플러그인은 kubelet에 설치되므로 kubelet 설정을 확인하면 된다. 

ps aux | grep kubelet 혹은 ps aux | grep cni 명령을 통해서 CNI 플러그인이 어떻게 설정되어 있는지 확인할 수 있다.

# Core DNS Troubleshooting

Core DNS는 6-4에서 설명한 바와 같이 중앙에 위치한 DNS 서버이다.

새로운 네트워크를 구성하고 서비스 이름으로 통신을 시도했는데 실패한다면 DNS 문제일 가능성이 높다.

Core DNS 관련 설정은 Kubernetes 클러스터 내에 구성되어 있다.

coredns 라는 이름의 pod로 구성되어 있으며 Namespace는 kube-system 이다.

이 pod는 Deployment로 구성되어 있다.

![image1]()

그래서 문제가 발생했을 경우 Pod가 종료되며 describe 명령을 통해서 확인할 수 있다.

그리고 Config Map이 존재한다. 역시 coredns라는 이름이다.

![image2]()

그리고 Service가 있으며 이름은 kube-dns이다.

![image3]()

만약 Service 설정이 잘못되어 있으면 Core DNS로 진입이 안될 수 있다.

![image4]()

그리고 Cluster Role과 Cluster Role Binding도 존재한다. 이는 coredns와 kube-dns 모두 존재한다.

이들을 확인해보면서 문제 되는 부분을 수정하면 된다.

대체로 문제는 다음과 같다.

1. CNI 플러그인을 설치 안했을 경우 발생한다.

2. 이전 버전의 Docker와 함께 SELinux를 사용하는 노드가 있을 경우 coredns가 시작하지 않는 문제가 있다고 한다.

  이 문제는 Docker를 업데이트 하거나 SELinux를 비활성화 한다. 혹은 coredns 설정에서 allowPrivilegeEscalation 을 true로 수정하면 된다.
  
3. Kubelet이 잘못된 resolv.conf를 가지고 있을 경우 발생한다. 

# Kube-Proxy

kube-prxoy 역시 Static Pods로 구성된다.

그래서 kubectl get pods 명령을 사용하여 상태를 확인할 수 있다.

그리고 kubectl logs 명령으로 확인하는 방법도 있다.

![image5]()

그리고 kube-proxy는 Config Map을 가지고 있다.

이 Config Map을 통해 설정을 확인할 수 있다.

![image6]()

kube-proxy의 config가 /var/lib/kube-proxy/config.conf 임도 확인할 수 있다.

문제가 될만한 설정을 확인하고 수정하면 된다.

