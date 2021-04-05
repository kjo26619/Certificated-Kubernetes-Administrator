# Application Failure

만들고 배포한 애플리케이션이 작동하지 않으면 두 가지 사항을 확인해볼 필요가 있다.

첫 번째로 Service 부분과 두 번째로 Pod 부분이다.

# Service Troubleshooting

외부에서 Pod로 접속하기 위해서는 Service가 필요하다. 내부에서도 Pod 들간에 통신이 용이하기 위해서는 Service를 이용해서 구축해야 한다.

Serivce를 확인하는 가장 쉬운 방법은 curl을 이용하는 방법이다.

내부에서 curl http://(SERVICE IP):(SERVICE PORT) 로 curl을 보내서 응답하는 지 확인하면 된다.

```
# curl http://(SERVICE IP):(SERVICE PORT)
```

만약, curl이 잘못된 응답을 내놓거나 연결이 되지 않으면 Pod와 Service 간에 설정이 문제가 있는 것이다.

이 때는 kubectl describe 명령을 이용해서 Service에 대해서 정확히 확인해볼 필요가 있다.

```
# kubectl describe service (SERVICE NAME)
```

![image1]()

Service 에서 확인해볼 것은 Selector와 Endpoint이다.

Selector는 Pod를 설정하는 것으로 제대로 원하는 Pod를 설정했는지 확인해야 한다.

Endpoint는 Pod의 IP 부분으로 원하는 Pod의 IP와 포트가 맞는지 확인해야 한다.

그 외에도 자신이 설정한 Node Port 부분이 제대로 이루어져 있는지 확인해보면 된다.

# Pod Trobuleshooting

대체로 Pod의 문제는 상태를 확인한 뒤 describe 에서 확인할 수 있는 로그를 통해서 고치면 된다.

kubectl get pods 명령을 이용해서 현재 상태를 확인한다.

```
# kubectl get pods
```

![image2]()

현재 STATUS가 Running이 아닐 경우 Pod는 제대로 동작하지 않는 것이다. 그러나 Running 임에도 RESTARTS 가 높아진다면 문제가 생긴 것이다.

Pod가 실행이 되다가 갑자기 종료되고 다시 시작되는 것이므로 확인해볼 필요가 있다.

확인을 위해서는 kubectl describe pod 명령이나 kubectl log 명령을 사용하면 된다.

```
# kubectl describe pod (POD NAME)

# kubectl logs (POD NAME)
```

![image3]()

![image4]()

describe 명령은 Events 부분을 보면 문제점을 확인해볼 수 있다.

logs 명령을 사용하면 쌓이는 log 대부분을 확인해볼 수 있다.

Kubernetes 설정이 제대로 되었음에도 애플리케이션이 제대로 동작하지 않으면 애플리케이션 자체나 의존도 문제일 가능성이 높다.
