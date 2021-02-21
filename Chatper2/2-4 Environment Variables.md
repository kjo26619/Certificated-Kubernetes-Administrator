# Environment Variables

Dockerfile에서 ENV로 환경 변수를 지정할 수 있다. 환경 변수는 Key-Value로 이루어진 값으로 프로그램의 동작 방식, 동작해야할 위치 등을 지정하는데 사용한다.

Kubernetes에서 환경 변수를 지정하는 방법은 YAML파일의 containers에서 env를 추가해주면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    env:
      - name: APP_NAME
        value: simpleWeb
```        
   
그런데 환경 변수라는 것은 꽤 많을 수도 있기 때문에 이렇게 YAML파일에서 모두 지정하는 것은 비효율적일 수 있다.

그래서 Kubernetes에서는 ConfigMap이나 Secret을 사용한다.

# ConfigMap

ConfigMap은 Key-Value 쌍을 저장하는데 사용하는 API 오브젝트이다. Pod를 구성할 때 환경 변수, 커맨드라인 인수 등에 사용할 수 있다.

하지만 ConfigMap은 모두 플레인 즉, 암호화되지 않고 저장되며 암호화가 필요하면 Secret을 사용해야 한다.

ConfigMap은 명령어를 사용하거나 YAML 파일을 통해서 만들 수 있다.

kubectl create configmap 명령어를 사용해서 만들 수 있으며 Key-Value를 직접 설정하거나 파일을 가져올 수 있다.

```
# kubectl create configmap (CONFIG NAME) --from-literal=(KEY)=(VALUE)

# kubectl create configmap (CONFIG NAME) --from-file=(FILE PATH)
```

YAML 파일을 이용할 경우 kind가 ConfigMap이 되며, spec 대신에 data 라는 섹션을 사용한다.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-2
data:
  APP_NAME: "simpleWeb"
  APP_VERSION: "1.10"
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/env1.PNG)

ConfigMap을 통해 만들면 각 Container의 여러 조건을 YAML파일에 나열할 필요 없이 다 따로 지정해줄 수 있다. 

Configmap을 확인하기 위해서는 kubectl get configmaps 명령어와 kubectl describe configmaps 명령어를 사용하면 된다.

```
# kubectl get configmaps

# kubectl describe configmaps (CONFIG NAME)
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/env2.PNG)

ConfigMap을 만들었다면 Pod를 구성할 때 env에서 Key-Value를 구성하지 않고 ConfigMap으로 구성해줄 수 있다.

contaienrs의 env는 envFrom으로 바뀐다.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    envFrom:
      - configMapRef:
          name: app-config
```

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/env3.PNG)

kubectl describe 명령어에서 Environment Variables from 을 확인해보면 app-config가 있는 것을 확인해볼 수있다.

이러한 ConfigMap은 로컬 환경에만 유지되므로 Volume에 지정해줄 수도 있다. 자세한 사항은 Volume에서 설명한다.

