# Backup and Restore

Kubernetes 클러스터를 구성은 어려운 작업이기 때문에 항시로 설정을 백업해두는 것이 좋다.

만약 잘못된 설정을 했는데 되돌리기 어려울 경우 백업한 설정을 가지고 와서 Restore 시키는 것이다.

Kubernetes의 모든 설정을 가져오는 것은 어렵지 않다.

Kubernetes는 대체로 YAML 파일로 구성이 되어 있기 때문에 이 YAML 파일들을 수시로 백업해놓고 사용해도 된다.

혹은 Kube-system 내에 존재하는 k8s Pods의 설정들까지 백업해 놓고 싶다면

kubectl get 명령어를 통해서 YAML 파일을 가져오면 된다.

```
# kubectl get all --all-namespaces -o yaml > (YAML FILE NAME).yaml
```

혹은 Verelo 라는 Kubernetes 클러스터 리소스와 볼륨을 마이그레이션하는 백업 애플리케이션이 있다.

https://velero.io/

대부분의 kubernetes를 사용하는 클라우드에서도 지원이 된다.

# ETCD Backup

ETCD의 경우에는 etcdctl 이라는 ETCD 전용 컨트롤 API가 존재한다.

정확한 것은 https://etcd.io/docs/v3.4/ 에서 확인해보면 된다.

그리고 Kubernetes 에서 ETCD를 백업할 때에는 암호화해서 데이터를 저장해주는 snapshot을 이용하면 된다.

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

snapshot은 etcdctl에 내장되어 있다.
