# OS Upgrades

Kubernetes에서는 Node가 중단되었을 경우 (kubelet과 통신이 되지 않음) 일정한 시간을 대기한 후 다른 Node로 Pod를 이동시킨다.

기본적인 시간은 5분이며 재부팅이 되어서 다시 연결이 되면 Pod를 다시 실행한다.

하지만 Node가 사용하는 OS를 업그레이드할 경우에는 5분 안에 다시 온라인 상태가 될 지는 미지수이다.

그래서 Kubernetes에서는 실행 중인 Pod를 이동시키는 drain 명령어를 지원한다.

```
# kubectl drain (NODE NAME)
```

![image1]()

kubectl drain 명령어를 통해서 Node에 있는 Pod를 다른 Node로 이동시킨 뒤에 재부팅을 수행하면 된다.

재부팅이 끝난 뒤 다시 Node가 Pod를 할당 받을 준비가 되면 uncordon 명령어로 drain을 취소하면 된다.

```
# kubectl uncordon (NODE NAME)
```

![image2]()

drain 명령을 수행하면 Kubernetes Controller는 그 Node를 Scheduling에서 제외한다. 그리고 uncordon 명령을 수행하면 다시 Scheduling에 포함한다.

이를 통해서 OS를 업그레이드 하거나 오류 없이 안전하게 재부팅할 수 있다.

uncordon이 있다면 cordon 명령어도 존재한다. cordon은 drain 명령어와는 달리 수행 중인 기존 Pod는 제거하지 않고 더이상 Scheduling 하지 않는다.

```
# kubectl cordon (NODE NAME)
```

cordon은 Scheduling을 막기 위해 사용한다.
