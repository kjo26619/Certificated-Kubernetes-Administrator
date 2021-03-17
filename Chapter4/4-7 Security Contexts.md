# Security Contexts

Docker에는 Container를 실행할 때 사용된 사용자의 ID를 수집하거나 Container에서 추가 또는 제거할 수 있는 Linux 기능 등의 Security Standard를 지원한다.

역시, Kubernetes에서도 이 기능을 지원하며 Pods를 생성할 때 지정할 수 있다.

이러한 설정은 Pod Level과 Container Level로 나뉘는데 

Pod Level로 지정할 경우 Pod 내의 모든 Container에게 보안이 설정된다.

그리고 Container Level로 지정할 경우 하나의 Container에게만 보안이 설정된다.

설정은 YAML파일에서 지정해주면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

spec에 securityContext를 추가해주면 되며 runAsUser는 User의 ID를 의미한다.

이는 Pod Level의 설정이며 모든 Container에게 적용된다.

Container Level 적용도 유사하나 containers에 추가하면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

정확한 Security Context 는 https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ 에서 확인할 수 있다.

밑에 붙은 capabilities는 Linux 기능을 추가하는 것으로, Pod Level에서는 지원하지 않고 Container Level에서만 사용할 수 있다.
