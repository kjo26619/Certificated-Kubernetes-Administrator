# Monitoring

Kubernetes에서는 PoD가 사용하는 CPU, Memory, Disk를 확인하기 위해서는 Monitoring 도구가 필요하다.

Monitoring은 Container, PoD, Cluster 등이 동작하는 것을 감시하고 어플리케이션이 제대로 동작하는지 검사할 수 있다.

Monitoring 도구에는 Metric Server, Prometheus, Elastic Stack, Datadog, Dynatrace 등이 있다.

# Metric Server

Metric Server는 Heapster 라고 하는 기존 Kubernetes에서 사용하던 Monitoring 도구에서 경량화된 버전이다.

Metric Server는 Kubelet을 통하여 Node, Pod에서 사용하는 CPU, Memory 같은 Metric을 모으는 도구이다.

Kubelet에서는 cAdvisor라는 Container Advisor가 있으며 cAdviosr가 성능 Metric을 검색할 수 있다.

Metric Server를 설치하는 방법은 git을 이용한다.

정확한 설치법은 https://github.com/kubernetes-sigs/metrics-server 에 나와있으며 다음과 같은 명령어를 사용하면 된다.

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml 
```

Metric Server의 설치가 끝나면 kubectl top 명령어를 사용할 수 있으며 Node와 Pod가 차지하는 CPU, Memory를 확인할 수 있다.

```
kubectl top (node or pod or ...)
```

그 외에 다른 Montioring 도구는 공식 홈페이지에서 확인하면 된다.

# Logging

Log는 문제가 발생했을 때 원인을 찾고 지금까지의 활동 내역을 확인할 수 있기 때문에 매우 중요하다.

Kubernetes에서는 기본적으로 Pod 수준의 Log 명령어를 지원해준다. 이는 kubectl logs 명령어이다.

```
kubectl logs (POD)
```

원하는 정보를 출력하거나 Log 파일로 만들기 위해서는 https://kubernetes.io/docs/concepts/cluster-administration/logging/ 에서의 지침을 따르면 된다.

위 링크에서도 나오지만, Kubernetes에서는 클러스터 수준의 Logging 아키텍처를 공식적으로 지원하지 않는다.

대신 Kubernetes와 호환되는 여러 Logging 도구들이 있으니 따로 설치해서 사용해야 한다.
