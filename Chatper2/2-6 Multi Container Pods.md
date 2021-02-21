# Multi Container Pods

Multi Container Pods는 MSA(Micro Service Architecture)에서 서로 관련된 Container를 Pod로 묶는 것이다.

MSA는 크게 하나의 프로그램으로 이루어져있던 서비스(Monolithic Architecuture)를 하위 구성 요소로 잘게 쪼개서 서비스로 만드는 구조로 서비스의 수정, 확장 및 관리에 용이하다.

그러나 매우 밀접하게 관련되는 서비스들이 있다. 예를 들어, 웹 서버와 웹 서버의 Logging 서비스의 조합이다.

이럴 경우 MSA로 구성은 하지만 Kubernetes에서는 Pod에 여러 Container를 넣어서 관리하도록 한다.

Multi Container Pods의 Containers는 같이 Scaling 되며 같은 Lifecycle, Storage, Network를 갖는다.

Multi Container Pods는 기존의 YAML파일에서 containers 가 배열로 구성되는 이유이다.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers: (Array)
    - name: simple-webapp 
      image: nginx
```

즉, containers에 필요한 Container를 추가로 넣어주면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers: (Array)
    - name: simple-webapp 
      image: nginx
    - name: log-agent
      image: log-agent
```

# Init Containers

