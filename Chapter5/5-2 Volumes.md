# Volumes

원래는 Pods가 생성되고 실행되면서 생기는 데이터들은 Pods가 사라지면서 같이 사라진다.

하지만 데이터를 유지할 필요가 있는 Pods가 존재한다. 

그래서 존재하는 기능이 바로 Volumes이다. 

Volumes는 Pods 외부에 데이터 저장소를 만든 뒤 이를 Pods와 연결한다. 그리고 Pods가 만든 데이터는 Volumes에 저장한다.

이렇게 하면 Pods가 사라져도 Volumes에 남아있는 데이터들은 모두 유지된다.

Volumes를 설정하는 법은 Pods를 만드는 YAML 파일에서 설정할 수 있다.

```
apiVersion: v1
kind: Pod
metadata:
  name: number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: directory
```

YAML 파일에서 spec 밑에 volumes 섹션을 추가하여 원하는 Volume을 만들어낸다.

그리고 연결할 Container의 설정에 volumeMounts를 추가한다.

그리고 원하는 Volume의 이름을 적어주면 된다.

여기서 path는 현재 호스트의 로컬 파일 시스템의 경로이고 mountPath는 Container 내부의 파일 시스템 경로이다.

# Multi-Node Volume

위와 같은 설정의 Volume에는 한 가지 문제점이 있다.

바로, 노드가 여러개일 경우 어느 호스트의 경로를 가져올 것인가? 이다.

Kubernetes에서는 노드가 여러 개일 경우, 모든 노드가 Volume에서 지정한 경로를 가지고 있다고 생각한다.

즉, 위와 같은 설정에서는 모든 노드가 /data를 가지고 있다고 가정한다. 그리고 이 /data는 모두 동일한 데이터를 가지고 있다고 가정한다.

그러다보니 로컬 파일 시스템의 경로로 Volumes을 지정하는 것은 Multi-Node 상황에서는 추천하지 않는다.

그래서 Kubernetes에서는 여러가지 외부 Storage Solution을 지원한다. 이는 https://kubernetes.io/docs/concepts/storage/volumes/ 에서 확인할 수 있다.

만약, AWS EBS를 사용한다고 가정하면 다음과 같이 설정한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
      
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: "<volume id>"
      fsType: ext4
```
