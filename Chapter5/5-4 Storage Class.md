# Storage Class

Kubernetes에서 클라우드의 PV가 필요할 경우 먼저, 어플리케이션이 클라우드 플랫폼에게 디스크를 수동으로 프로비저닝 하게끔 한다.

스토리지 프로비저닝이 끝나면 같은 이름의 PV를 생성해야 한다. 생성할 때 클라우드의 디스크임을 명시하고 사용해야 한다. 

이를 정적 프로비저닝이라고 한다.

Kubernetes가 관리의 자동화를 위해 사용하는데 저장소를 구성할 때 수동으로 프로비저닝 하고 있으면 비효율적이다.

그래서 Kubneretes에서 동적 프로비저닝을 지원하기 시작했다. 

동적 프로비저닝은 PV를 따로 만들지 않아도 PVC에서 Storage Class를 지정하면 자동으로 PV가 생성된다.

# Dynamic Volume Provisioning

동적 볼륨 프로비저닝은 위에서 언급한 바와 같이 PV를 자동으로 생성하는 방법이다.

동적 볼륨 프로비저닝을 위해서는 Storage Class가 필요하며 다음과 같이 YAML 파일로 정의한다.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

Storage Class는 내가 어떠한 저장소를 사용할 것인지 설정하는 것이다.

provisioner는 https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner 에서 어떤 종류들을 지원하는지 확인할 수 있다.

parameters는 자신이 사용하는 저장소 종류에 따라 다르며 추가적으로 저장소에 대해 설정할 파라미터를 지정하는 것이다.

예를 들어, GCP의 standard, ssd 같은 스토리지 옵션이 있다.

자세한 사항은 https://kubernetes.io/docs/concepts/storage/storage-classes/#parameters 에서 확인할 수 있다.

reclaimPolicy는 PV의 Reclaim Policy와 같은 것으로 Delete나 Retain이 있다.

allowVolumeExpansion은 사용자가 PVC를 통해 Volume의 크기를 조절할 수 있게끔 하는 것이다. 

이는 지원하는 Storage Class와 Kubernetes Version이 다르므로 https://kubernetes.io/docs/concepts/storage/storage-classes/#allow-volume-expansion 에서 확인해야 한다.

mountOptions는 동적으로 생성된 PV에게 지정할 마운트 옵션이다.

volumeBindingMode는 Volume Binding과 동적 프로비저닝의 시작 시기를 제어한다.

Immediate는 즉시 발생이지만 모든 노드에서 전역적으로 접근 불가능한 스토리지 백엔드의 경우 Pod의 스케줄링 없이 PV가 바인딩 되거나 프로비저닝 될 수 있다. 

자세한 사항은 https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode 에서 확인할 수 있다.

Storage Class를 만들었으면 PVC를 생성할 때 Storage Class로 지정해주면 된다.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

PVC에서 지정하고 Pod에 PVC를 지정하면 자동으로 PV가 프로비저닝 된다.

대신 사용하는 저장소와 반드시 Storage Class가 같아야 하며 Kubernetes에서 지원이 되어야 한다.

