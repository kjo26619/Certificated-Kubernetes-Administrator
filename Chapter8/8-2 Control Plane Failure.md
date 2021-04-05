# Control Plane Failure

Control Plane은 Kubernetes의 핵심인 만큼 고장에 대해서 매우 민감해야 한다.

특히, 여러 Control Plane 서버로 구성하는 HA 환경이 아닐 경우 1개의 Control Plane이 고장나면 Kubernetes 클러스터 전체가 고장난다.

제대로된 확인을 위해서는 먼저, Control Plane의 구성 요소에 대해서 알아야 한다.

Control Plane의 중심이 되는 5가지의 필수 구성 요소 kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, kube-proxy 를 확인해야 한다.

# Service Check

먼저 이들은 모두 Static Pod로 구성되므로 kubectl get pods를 이용하면 확인할 수 있다.

구성요소의 Namespace는 kube-system으로 구성된다.

```
# kubectl get pods -n kube-system
```

![image1]()

이들 중 문제가 생긴 Pod를 확인하고 수정하면 된다.

로그를 확인하고 싶으면 kubectl logs 명령을 사용하면 된다.

```
# kubectl logs -n kube-system (POD NAME)
```

꽤 길어서 확인하기 힘들 수 있으니 반드시 파일로 빼내서 확인하는 것이 좋다.

그리고 journalctl 명령을 사용해도 된다.

```
# journalctl -u (COMPONENT NAME)
```

 journalctl은 Systemd에서 수집한 로그를 journal 이라는 바이너리 형식으로 저장하고 이를 검색하고 확인하는 명령어이다.
