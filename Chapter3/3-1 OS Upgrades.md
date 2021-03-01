# OS Upgrades

Kubernetes에서는 Node가 중단되었을 경우 (kubelet과 통신이 되지 않음) 일정한 시간을 대기한 후 다른 Node로 Pod를 이동시킨다.

기본적인 시간은 5분이며 재부팅이 되어서 다시 연결이 되면 Pod를 다시 실행한다.

하지만 Node가 사용하는 OS를 업그레이드할 경우에는 5분 안에 다시 온라인 상태가 될 지는 미지수이다.

그래서 Kubernetes에서는 실행 중인 Pod를 이동시키는 drain 명령어를 지원한다.

```
# kubectl drain (NODE NAME)
```

![image1]()

![image2]()

kubectl drain 명령어를 통해서 Node에 있는 Pod를 다른 Node로 이동시킨 뒤에 재부팅을 수행하면 된다.

하지만, drain 명령어를 사용해보면 Node에서 Pod를 삭제하지 못하는 경우가 생긴다. 

2번째 이미지에 나오며 Pod가 DaemonSet, ReplicaSet 등에 의해 관리되지 않을 경우와 Daemonset에 의해서 관리되는 Pod가 있을 경우이다.

이 두가지의 경우에는 --force와 --ignore-daemonsets 옵션을 추가해주면 된다. 그러나 --force를 사용할 경우 그 Pod는 삭제된다.

![image3]()

이를 방지하기 위해서는 ReplicaSet으로 Pod를 구성해서 관리해주면 된다.

![image4]()

ReplicaSet에 의해서 Pod가 관리되고 확인해보면 각각 존재하는 Node에 Pod를 할당한 것을 확인할 수 있다.

![image5]()

여기서, drain 명령을 사용하면 node2는 evicted되며 존재하는 Pod가 다른 Node로 이동하고 더이상 Scheduling 받지 않는다.

![image6]()

![image7]()

Pod의 상태를 확인해보면 다른 Node로 이동한 것을 확인할 수 있다.

![image8]()

Node의 상태를 확인해보면 SchedulingDisabled가 되어 있는 것을 확인할 수 있다.

다시 Node가 Pod를 할당 받을 준비가 되면 uncordon 명령어로 drain을 취소하면 된다.

```
# kubectl uncordon (NODE NAME)
```

![image9]()

drain 명령을 수행하면 Kubernetes Controller는 그 Node를 Scheduling에서 제외한다. 그리고 uncordon 명령을 수행하면 다시 Scheduling에 포함한다.

그렇기에 새로 만든 Pod만 Scheduling할 때 영향을 받으며, drain할 때 이미 다른 Node로 옮겨진 Pod는 바로 옮겨지지 않는다.

이렇게 drain 명령을 이용하면 OS를 업그레이드 하거나 오류 없이 안전하게 재부팅할 수 있다.

# Cordon, Uncordon

uncordon이 있다면 cordon 명령어도 존재한다. cordon은 drain 명령어와는 달리 수행 중인 기존 Pod는 제거하지 않고 더이상 Scheduling 하지 않는다.

```
# kubectl cordon (NODE NAME)
```

cordon은 Scheduling을 막기 위해 사용한다.

![image10]()

확인해보면 기존에 존재하는 Pod들은 삭제되지는 않지만 Node의 상태에는 SchedulingDisabled가 되어 있다.
