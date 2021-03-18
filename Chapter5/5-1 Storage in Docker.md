# Storage in Docker

Docker가 설치되면 로컬 파일 시스템의 /var/lib/docker에 관련 파일들이 저장된다.

대표적으로 aufs, containers, image, volumes 가 있다.

Union mount 관련은 aufs에 저장되고 Container 관련은 containers에 저장되고 Image 관련은 images에 저장되고 Volume 관련은 volumes에 저장된다.

그렇다면 저장될 때 Image와 Container는 어떻게 저장할 것인가? 이것은 Docker의 Layered Architecture를 이해해야 한다.

# Layered Architecture

Docker Image를 구성하는 Dockerfile은 Layer로 구성되어 있다.

각각 하나의 Layer 당 하나의 명령으로 구성된다.

정확한 명령의 구성은 https://github.com/kjo26619/Docker/blob/main/Chapter3/3-1%20Container%20Image.md 에서 정리했다.

Dockerfile의 예제를 보면 Layer를 확인할 수 있다.

```
FROM Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FALSK_APP=/opt/source-code/app.py flask run
```

각 FROM, RUN, COPY, ENTRYPOINT가 하나의 명령이자 Layer이다.

Docker는 이 Dockerfile을 보면서 각 Layer에 필요한 파일들을 저장해놓는다.

예를 들어, FROM Ubuntu의 경우에는 기본 Ubuntu가 필요하므로 이를 저장한다.

그리고 RUN 명령에서 apt를 통해 python을 저장하고 pip를 통해 flask를 저장한다.

그리고 로컬 시스템의 현재 디렉토리를 Container 내부에 /opt/source-code에 옮기면 된다.

만약, 새로운 Dockerfile을 만들었다고 가정한다.

```
FROM Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FALSK_APP=/opt/source-code/app2.py flask run
```

app.py 대신에 app2.py를 실행해야되서 Dockerfile을 바꾸었다.

이 때 Layered Architecture의 장점을 알 수 있는데, 이전 Dockerfile과 같은 Layer들은 이미 저장이 됐기 때문에 따로 실행하지 않는다.

이전 Dockerfile과 같은 3개의 Layer(FROM, RUN, RUN)는 캐시에서 가져와서 사용하며 2개의 Layer만 작동한다.

이렇게 함으로써 Docker는 디스크를 효율적으로 사용하면서 빠르게 Container를 만들 수 있는 것이다.

환경은 똑같고 소스 코드만 다르다면 소스 코드만 변경함으로써 높은 효율을 보일 수 있는 것이다.

이렇게 Dockerfile에 지정된대로 빌드가 끝나면 Image가 되며 더이상 수정할 수 없게 된다.

이것은 Image를 사용하는 모든 Container가 Image Layer를 공유하고 있기 때문이다.

대신 읽기/쓰기 작업이 필요하기 때문에 Image를 통해서 Container가 만들어지면 Container Layer가 생성이 된다.

이 Layer는 읽기/쓰기 모두 가능하며 Container 내부에서 만든 로그나 파일들을 저장한다. 

만약, Container가 실행 중인 동안 Image Layer에 존재하는 app.py, app2.py를 변경하려면 어떻게 해야할까?

이 파일들을 Container Layer로 복사하고 수정한 뒤 사용한다. 즉, Image Layer들은 재 빌드되지 않는 이상 바뀌지 않는다.

이러한 메커니즘을 COPY-ON-WRITE 메커니즘이라고 한다.

Container Layer에서 수정되고 바뀐 파일들은 Container의 생명 주기동안만 존재한다. 즉, Container가 종료되는 순간 사라진다.

그렇다면 이 바뀐 파일들을 어떻게 유지할 수 있을까? 예를 들어, DB Container 내부의 정보들은 계속 유지가 되어야 한다.

이럴 때 필요한 것이 Volumes 이다.

# Volumes

Container 실행 중 바뀐 파일들은 Container가 종료되면 사라지므로 영구적으로 유지해야하는 데이터를 Container 내부에 넣기에는 문제가 있다.

이를 위해서 Volumes이 등장했으며 위에서 언급한 바와 같이 /var/lib/docker/volumes에 이 영구 데이터들을 저장하고 관리한다.

Docker에서는 Volumes을 만든 뒤 Container에게 마운트시킬 수 있다. 자세한 사항은 https://github.com/kjo26619/Docker/blob/main/Chapter4/4-2%20Data%20Volumes.md 에서 확인 가능하다.

Volumes는 특정 위치에 데이터 저장소를 만들고 이를 Container와 연결해준다.

그리고 Container Layer에서 이 데이터 저장소를 읽고 쓸 수 있게끔 한다.

그러므로 이 Volumes는 Container가 생성될 때 연결되고 삭제될 때 연결이 사라지게 된다. Volumes 자체는 사라지지 않으며 유지된다.

Container를 구성할 때 이러한 Volumes를 쓰는 것이 가장 추천하는 방법이다. 

하지만, 불가피하게 Image Layer를 바꾸고 사용할 때도 있어야 한다. 그러나 많은 Container가 사용하고 있는 Image Layer를 함부로 바꾸었다간 모두 꼬이는 사태가 벌어진다.

그래서 Docker에서는 Union File System을 지원한다. 이러한 Union File System을 위한 Storage Driver를 지원한다.

이는 Docker Docs에서 자세히 설명하고 있다. https://docs.docker.com/storage/storagedriver/select-storage-driver/ 
