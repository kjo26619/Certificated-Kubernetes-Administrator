# Secrets

Kubernetes를 사용하다보면 Docker와 마찬가지로 비밀번호, 토큰 등의 정보를 암호화할 필요가 있다.

그래서 Kubernetes에서는 Secrets라는 보안 개념을 지원한다. Secrets를 통해서 정보를 평문이 아닌 based64로 인코딩된 문자열로 저장해둘 수 있다.

Secrets는 Volume 내의 파일로 사용되거나 환경 변수 등에 사용할 수 있다. 

하지만 Secrets는 base64로 인코딩되며 누구나 쉽게 디코딩할 수 있다. 이에 따라서 Secrets를 암호화라고 보기에는 조금 애매하다.

그래서 Kubernetes 문서에도 Secrets에 대한 암호화를 활성화 방법을 제시한다. 참고 : https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

Secrets를 만드는 방법은 kubectl create 명령어를 사용한다. 방법은 ConfigMap과 유사하며 Key-Value를 명령어에서 지정하는 방법과 파일을 지정하는 방법이 있다.

```
# kubectl create secret generic (SECRET NAME) --from-literal=(KEY)=(VALUE)

# kubectl create secret generic (SECRET NAME) --from-file=(FILE PATH)
```

![image1]()

그리고 YAML파일을 이용해서 만들 수 있다. YAML파일의 형식도 ConfigMap과 유사하나 kind가 Secret으로 바뀐다.

그러나 가장 유의해야하는 점이 있다. 바로 data 섹션에서 Key-Value를 지정할 때 Value가 이미 base64로 인코딩된 문자열로 지정해야되는 점이다.

즉, passwrd 같은 경우에는 cGFzc3dyZA== 로 변환한 뒤 적어주어야 한다. 변환하는 법은 echo 를 이용하면 된다.

```
# echo -n (VALUE) | base64

# echo -n (ENCODED VALUE) | base64 --decode
```

밑에 있는 명령어는 base64로 인코딩된 문자열을 평문으로 디코딩하는 것이다.

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/secret1.PNG)

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret-2
data:
  DB_HOST: bXlzcWw=
  DB_USER: cm9vdA==
  DB_PASSWORD: cGFzc3dyZA==
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/secret2.PNG)

만들어진 Secrets를 확인하기 위해서는 kubectl get secret 명령어를 사용하면 된다.

그리고 Secrets에 대한 정확한 정보는 kub3ectl describe secret 명령어를 사용하면 된다.

```
# kubectl get secrets

# kubectl describe secrets
```

![image3](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/secret3.PNG)

Pod를 만들 때 Secrets를 환경 변수로 지정하는 방법 역시 ConfigMap과 유사하다.

containers에서 envFrom을 지정하면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
sepc:
  containers:
  - name: simple-webapp
    image: nginx
    envFrom:
      - secretRef:
        name: app-secret-2
```

![image4](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chatper2/Image/secret4.PNG)

Secrets를 Volume으로 지정할 수 있으며 이는 Volume에서 자세히 다룰 예정이다.

만약 Volume으로 지정하면 Container 내부에서는 평문으로 저장된다.
