# Rolling Updates and Rollbacks

Kubernetes에서 Deployments는 Pod를 업데이트 할 때 Rolling Update를 지원한다.

Rolling Update는 Pod 하나씩 업데이트를 진행하여 어플리케이션이 멈추지 않도록 하는 업데이트 방법이다.

이 Rolling Update의 반대는 Recreate로 모든 Pod를 종료한 뒤 한꺼번에 업데이트를 진행하는 방법이다. 

# Rollout Command

Rollout 명령어는 Deployment의 배포 업데이트를 할 수 있는 명령어이다.

즉, Pod의 업데이트 상황을 볼 수 있고 이전에 해왔던 업데이트 기록을 확인할 수 있다.

명령어는 kubectl rollout 명령어이며 하위 명령에 따라서 여러 결과를 볼 수 있다.

현재 Rollout 상태를 확인하기 위해서는 kubectl rollout status 명령어를 사용하면 된다.

```
# kubectl rollout status (DEPLOYMENT NAME)
```

![image1]()

그리고 Rollout의 기록을 보고 싶다면 kubectl rollout history 명령어를 사용하면 된다.

```
# kubectl rollout history (DEPLOYMENT NAME)
```

![image2]()

