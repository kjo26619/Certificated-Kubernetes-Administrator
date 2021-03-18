# Persistent Volumes

Kubernetes 클러스터 내부에서 직접 관리하는 Volumes은 Persistent Volumes라는 이름을 갖는다.

sPersistent Volumes (PV)는 관리자가 프로비저닝 했거나 프로비저닝 도구(AWS EBS, AzureFile, Cinder, ...)로 프로비저닝한 저장소이다.

PV는 Volumes와 같은 기능을 갖는다. 

대신, PV는 클러스터에서 생성하고 관리하는 클러스터 내부의 자원이며 각 노드로 생성해주거나 관리해준다.

PV의 생성은 YAML 파일을 통해서 만들 수 있다.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

accessModes는 볼륨에 접근할 수 있는 방식으로 3이다.

  - ReadWriteOnce : 하나의 노드에서 볼륨을 읽기-쓰기 가능
  - ReadOnlyMany : 여러 노드에서 볼륨을 읽기 전용
  - ReadWriteMany : 여러 노드에서 볼륨을 읽기-쓰기 가능

capacity는 PV의 크기이다.

hostPath는 호스트에서의 PV의 위치이다. 

물론 hostPath를 프로비저닝 도구로 지정할 수 있다.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeId: <volume-id>
    fsType: ext4
```

만들어진 PV를 확인하기 위해서는 kubectl get persistentvolume 명령어를 사용하면 된다.

```
# kubectl get persistentvolume
```

![image1](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter5/Image/pv1.PNG)

# Persistent Volume Claims

Persistent Volume Claims (PVC)는 저장소 즉, Persistent Volume을 사용하기 위해 사용자가 보내는 요청이다.

PVC가 생성되면 Kubernetes는 설정 및 요청에 따라 알맞은 PV를 연결(Binding)해준다.

PVC의 설정은 YAML파일을 통해 설정할 수 있다.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

YAML 파일 설정은 PV와 크게 다르지 않다.

accessModes는 PV에 접근할 때의 모드를 지정하며 3가지가 존재한다.

resources는 요청한 크기이다. 이것을 확인하고 Kubernetes가 알맞은 PV에 연결해준다.

만들어진 PVC를 확인하기 위해서는 kubectl get persistentvolumeclaim 명령어를 사용하면 된다.

```
# kubectl get persistentvolumeclaim
```

![image2](https://github.com/kjo26619/Certificated-Kubernetes-Administrator/blob/main/Chapter5/Image/pv2.PNG)

PV를 모두 사용하고 나서 사용자가 PV를 반환하기 위해서는 PVC를 삭제해야 한다.

```
# kubectl delete persistentvolumeclaim (PVC NAME)
```

PVC를 삭제해도 PV는 그대로 남게 된다. 그런데 사용자가 쓰던 데이터 자체는 남아있으므로 다른 사용자의 접근을 막는다.

이러한 PV Reclaim Policy를 Retain이라고 한다.

수동으로 작업한 후 관리자가 PV를 삭제하면 차지하고 있는 리소스는 반환된다.

이러한 PV Reclaim Policy는 총 3가지로 Retain, Delete, Recycle이다.

Delete는 PVC가 사라짐과 동시에 PV와 데이터를 삭제한다.

Recycle은 볼륨 내의 데이터를 삭제하고 다시 사용할 수 있도록 한다.

하지만, Recycle은 현재 사용하지 않으며 동적 프로비저닝을 추천하고 있다.

PVC를 구성하여 사용자가 PV를 할당 받은 뒤 Pods에게 설정할 수 있다.

이는 Pod의 YAML파일에서 volumes를 추가할 때 PVC로 설정해주면 된다.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
