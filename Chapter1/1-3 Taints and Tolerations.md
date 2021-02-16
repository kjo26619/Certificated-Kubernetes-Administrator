# Taints and Tolerations

Taints와 Tolerations는 Scheduling 기법 중 하나이다.

Taints는 Node에 적용할 수 있는 Scheduling 기법이고 Tolerations는 Pod에 적용할 수 있는 Scheduling 기법이다.

Taints가 설정된 Node는 같은 Tolerations가 설정된 Pod만을 Scheduling 받을 수 있다.

즉, 둘을 같이 사용하여 Pod가 원하지 않는 Node로 Scheduling 되는 것을 방지할 수 있다.

# Taints

Taints는 마치 입장 게이트와 같은 의미이다. Taints가 설정이 되어있으면 보통의 Pod는 Scheduling 받지 않는다.

Tolerations가 설정된 즉, 출입증이 있는 Pod만이 Node에 Scheudling 될 수 있다.

Taints는 <key>=<value>:<taint-effect> 로 구성된다. Key와 Value는 관리자가 원하는대로 지정할 수 있다.
  
그리고 Effect는 NoSchedule, PreferNoSchedule, NoExecute 3가지로 나뉜다. 

1. NoSchedule : 이 Taint가 설정되면 Tolerations가 설정되지 않은 Pod를 받지 않는다. 기존에 있던 Pod는 지속됨.

2. PreferNoSchedule : 이 Taint가 설정되면 NoSchedule과 마찬가지로 Pod를 받지 않는다. 하지만, 이것은 시스템 상 가능할 경우에만 적용됨. (보장되지 않음)

3. NoExecute : 이 Taint가 설정되면 Tolerations가 설정되지 않은 기존에 있던 Pod까지 모두 제거되며 Pod를 받지 않는다.

Taints 설정은 kubectl taint nodes 명령어를 이용하면 된다.

```
# kubectl taint nodes (NODES NAME) (KEY)=(VALUE):(TAINT-EFFECT)
```

![image1]()

Taints가 설정되면 kubectl describe node 명령어로 확인할 수 있다.

명령어 사용 시 파이프라인을 이용해서 grep Taint하면 Taint만 확인해 볼 수 있다.

```
# kubectl describe node (NODES NAME) | grep Taint
```

![image2]()

참고로 Control Plane은 이미 NoSchedule Taints가 설정이 되어 있다.

설정된 Taints를 삭제하기 위해서는 kubectl taint nodes 명령어에서 뒤에 Taint에 대하여 - 를 붙여주면 된다.

```
# kubectl taint nodes (NODES NAME) (KEY)=(VALUE):(TAINT-EFFECT)-
```

![image3]()

# Tolerations

Tolerations는 출입증과 같다. Taints가 설정된 Node에 Scheduling 되기 위해서는 반드시 Tolerations가 필요하다.

하지만 Tolerations가 설정된 Pod가 반드시 Taints가 설정된 Node에 Scheduling 되는 것은 아니다. Taints가 설정되지 않은 Node로 Scheduling 될 수 있다.

Taints와 Tolerations는 잘못된 Scheduling을 방지하는 것이지 이 Pod는 반드시 이 Node에 설정되어야한다 라는 의미가 아니다.

Tolerations는 Pod의 YAML파일에서 spec 섹션에 tolerations 특성을 추가하여 설정한다.

![image4]()

Taints를 설정하면서 사용했던 Key, Value, Effect를 사용하면 된다. 

Operator는 Equal과 Exist가 있다. Equal은 Key와 Value가 같은 Taints에 대하여 Tolerations를 설정하는 것이다. 

Exist는 Value를 따로 지정할 필요가 없이 Key만 지정한다. 같은 Key인 Taints에 대한 Tolerations를 설정하는 것이다.

당연하게도 Taints와 Tolerations는 여러 개를 지정할 수 있다. 같은 Key 다른 Value일 수도 있고 다른 Key 다른 Value일 수도 있다.

![image5]()

