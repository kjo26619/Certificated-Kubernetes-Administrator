# Worker Node Failure

Control Plane가 클러스터 자체를 움직이는 노드라면 Worker 노드는 Pod를 할당받고 구성하는 노드이기 때문에 메모리나 CPU 등의 문제와 관련이 많다.

Worker 노드에 문제가 생긴다면 리소스 혹은 네트워크의 문제라고 보면 된다.

# Check Node

노드의 상태를 확인하기 위해서는 kubectl get nodes 명령을 사용하면 된다.

```
# kubectl get nodes
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter8/Image/worker1.PNG)

만약, Ready가 아니라 NotReady 상태라면 이유를 확인해볼 수 있다.

그 이유는 kubectl describe node 명령에서 확인할 수 있다.

```
# kubectl describe node (NODE NAME)
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter8/Image/worker2.PNG)

이 describe 결과에서 핵심은 밑에 있는 Conditions이다.

만약 노드에 문제가 생겼다면 이 상태를 확인하고 True 값을 반환해준다.

OutofDisk, MemoryPressure, DiskPressure, PIDPressure 가 있으며 이유는 Messasge에 나와 있는 것과 같다.

그래서 만약 노드에 메모리가 부족하여 MemoryPressure가 나온다면 MemoryPressure의 Status가 True로 바뀌고 Ready가 False로 바뀐다.

그런데 이 결과들이 모두 UnKnown으로 바뀔 수 있다.

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter8/Image/worker3.PNG)

이는 네트워크에서 이 노드와 연결을 할 수 없다는 의미이다. 대부분 네트워크에서 사라졌거나 노드 자체가 다운된 경우이다.

그 외에도 Worker 노드에서 kubelet 설정이 잘못되어 있을 수도 있다. 
