# Commands

Docker에서 Image가 시작하면서 실행할 명령을 지정할 수 있다. 이는 Dockerfile에서 CMD를 통해서 지정할 수 있다.

예를 들어, Nginx Image는 CMD에 nginx 명령어가 설정되어 있고, MySQL Image는 CMD에 mysqld 명령어가 설정되어 있다. Ubuntu 같은 경우에는 bash로 설정되어 있다.

이러한 것을 Commands라고 하며, Docker에서 Container를 시작할 때 지정해 줄수도 있다.

```
# docker run [COMMAND]
```

하지만 이 방법의 경우 만들 때마다 Commands를 지정해 주어야 하므로 Dockerfile을 이용해서 정의해주는 편이 용이하다.

Dockerfile에서 Ubuntu Image를 가지고 Commands를 지정하는 방법은 다음과 같다.

```
FROM ubuntu:18.04
...
CMD sleep 5
```

CMD의 경우에는 실제로는 ["executable", "param1", "param2"] 과 같은 형식을 가진다. 즉, sleep 5는 ["sleep", "5"]로 쓸 수 있다.

이러한 CMD는 위에서 말한 것 처럼 docker run을 할 때 수정해줄 수 있다. 만약, sleep 10으로 바꾸고 싶으면 docker run (IMAGE) sleep 10 이렇게 지정하면 된다.

이렇게 sleep 5에서 sleep 10으로 바꾸는 것처럼 파라미터만 바꾸고 싶을 때에는 ENTRYPOINT를 사용해도 된다.

```
FROM ubuntu:18.04
...
ENTRYPOINT ["sleep"]
```

ENTRYPOINT는 Container가 시작되었을 때 지정되는 스크립트 또는 명령이다. 대신 이 ENTRYPOINT에서 지정된 명령은 옵션을 이용해서 덮어씌우지 않는한 변경할 수 없다.

ENTRYPOINT를 지정하게 되면 docker run을 할 때 파라미터만 지정하면 sleep 명령에 파라미터가 전달된다.

```
docker run (BUILD IMAGE) 10
```

이렇게 변경되지 않는 실행 명령은 ENTRYPOINT로 지정하고 이 실행 명령에 기본 옵션 인자를 지정할 때 CMD를 같이 사용한다.

```
FROM unbuntu:18.04

ENTRYPOINT ["sleep"]

CMD ["5"]
```

docker run을 할 때 --entrypoint 옵션을 이용하면 ENTRYPOINT를 덮어씌울 수 있다.

# Kubernetes Commands and Arguments

Kubernetes에서 Docker처럼 실행될 때 사용할 명령을 지정해줄 수 있다. 이는 YAML파일에서 각 Container를 지정할 때 추가해주면 된다.

Kubernetes에서는 Docker에서 사용하는 ENTRYPOINT와 같은 역할을 하는 것을 command라고 한다.

그리고 ENTRYPOINT와 CMD를 같이 사용할 때 처럼 기본 옵션 인자를 지정할 때는 args를 사용한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

위에서 ENTRYPOINT가 command가 되는 것이고 CMD가 args가 되는 것이다.

이렇게 지정하면 기존에 Dockerfile에 지정된 ENTRYPOINT와 CMD가 command와 args에서 지정한 값으로 대체된다.
     
