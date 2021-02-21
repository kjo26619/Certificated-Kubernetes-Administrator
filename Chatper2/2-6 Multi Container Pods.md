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

Multi Container Pods를 구성할 때는 대체로 3가지 디자인 패턴을 사용한다. 이는 Sidecar, Adapter, Ambassador이다.

# Init Containers

Init Containers는 Pod의 Container들이 실행되기 전에 먼저 실행되는 특수 Container이다.

이는 서비스의 실행 순서, 처음에 코드 또는 바이너리를 가져오기 등을 위해 사용된다. 

예를 들어, DB 서비스가 먼저 실행된 이후에 웹 서버가 실행되어야 하는데 Pod의 순서는 어떻게 실행될 지 모르니까 웹 서버 Pod는 Init Containers를 사용해 기다리게끔 한다.

Init Containers는 spec 섹션에 containers 말고 initContainers를 추가해주면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

Init Containers는 Multi Container Pods라고 보면 되며, Init Containers도 여러개를 지정할 수 있다.

여러 개로 지정된 Init Containers는 위에서 아래 순서대로 실행되고 완료된다. 그리고 모든 Init Containers가 완료되면 Pod의 Container가 시작된다.

위 구성은 myservice와 mydb가 실될 때까지 Init Container가 기다리며 실행됐을 경우 Init Containers를 완료한다.

더욱 자세한 구성은 https://kubernetes.io/docs/concepts/workloads/pods/init-containers/ 를 참조하면 된다.
