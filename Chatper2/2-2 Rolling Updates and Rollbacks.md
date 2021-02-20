# Rolling Updates and Rollbacks

Kubernetes에서 Deployments는 어플리케이션을 업그레이드하면서 Pod의 설정을 업데이트 할 때 Rolling Update를 지원한다.

Rolling Update는 Pod 하나씩 업데이트를 진행하여 어플리케이션이 멈추지 않도록 하는 업그레이드 방법이다.

이 Rolling Update의 반대는 Recreate로 모든 Pod를 종료한 뒤 한꺼번에 업그레이드를 진행하는 방법이다. 

# Rollout Command

Rollout 명령어는 Deployment의 배포 업데이트를 할 수 있는 명령어이다.

즉, Pod의 업데이트 상황을 볼 수 있고 이전에 해왔던 업데이트 기록을 확인할 수 있다.

명령어는 kubectl rollout 명령어이며 하위 명령에 따라서 여러 결과를 볼 수 있다.

현재 Rollout 상태를 확인하기 위해서는 kubectl rollout status 명령어를 사용하면 된다.

```
# kubectl rollout status deployment (DEPLOYMENT NAME)
```

![image1]()

그리고 Rollout의 기록을 보고 싶다면 kubectl rollout history 명령어를 사용하면 된다.

```
# kubectl rollout history (DEPLOYMENT NAME)
```

![image2]()

# RollingUpdate and Recreate

Deployments의 두가지 업데이트 방법은 YAML 파일을 통해서 지정할 수 있다.

spec 섹션에서 strategy 특성을 추가한 다음 RollingUpdate나 Recreate를 지정한다.

![image3]()

Max Unavailable과 Max Surge는 각각 업데이트 중 사용할 수 없는 최대 포드 수와 업데이트 프로세스를 진행할 수 있는 최대 포드 수이다.

어느 정도의 Pod를 유지해야하는 지를 나타내는 지표이며 백분율 혹은 절대값으로 지정할 수 있다.

Deployment를 만든 뒤 kubectl describe 명령어로 확인해보면 StrategyType에서 확인할 수 있다.

```
# kubectl describe deployment (DEPLOYMENT NAME)
```

![image4]()

Recreate 같은 경우 RollingUpdate와는 다르게 몇 퍼센트의 Pod를 유지해야하는지 지정할 필요가 없다.

kubectl edit으로 Deployment를 수정하거나 YAML파일을 바꾼 뒤 kubectl apply 명령어를 사용하면 된다.

![image5]()

![image6]()

Rolling Update를 지정하면 Pod를 모두 종료하지 않고 일부만 종료하는 것을 확인해볼 수 있으며 Recreate로 지정하면 모든 Pod가 한번에 종료되고 다시 켜지는 것을 확인할 수 있다.

# Rollbacks

Rollback은 어플리케이션을 업그레이드한 이후에 잘못된 것이 있어서 되돌릴 경우 사용한다.

Rollback을 하는 방법은 Rollout 명령어 중에 undo 하위 명령을 사용하면 된다.

```
# kubectl rollout undo deployment (DEPLOYMENT NAME)
```

![image7]()

이 명령어를 통해 Rollout 하기 이전으로 돌아간다.

