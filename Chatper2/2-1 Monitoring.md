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
