# Network Policy

기본적으로 Kubernetes의 네트워크 내에 Pod들은 모두 연결되어 있고 통신이 가능하다.

하지만, 이 통신을 격리하고 특정 Pod들끼리만 통신을 시키고 싶을 수 있다.

그럴 때 사용하는것이 Network Policy이다.

# Ingress and Egress

Ingress와 Egress는 Kubernetes에서의 네트워크 트래픽의 흐름을 이야기 한다.

Kubernetes에서 Ingress는 외부에서 서버로 들어오는 네트워크 트래픽을 의미한다.

그리고 Egress는 서버에서 외부로 나가는 네트워크 트래픽을 의미한다.

외부와 서버 외에도 내부에서는 Pod를 기준해서 들어오는 트래픽을 Ingress, 나가는 트래픽을 Egress라고 한다.

# Network Policy Configuration

Network Policy를 구성은 YAML 파일을 통해 구성한다.

만약, 프론트 엔드를 구성하는 Web Pod와 DB를 연결하고 데이터를 가져오는 API Pod 그리고 DB를 구성하는 DB Pod가 있다고 가정한다.

그랬을 때 DB Pod와 연결할 수 있는 것은 API Pod이며, Web Pod에서는 직접 DB Pod로 연결할 수 없게끔 구성한다고 가정한다.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

DB Pod를 기준으로 들어오는 트래픽이기 때문에 Ingress로 처리해준다. 

kind를 NetworkPolicy로 만든 다음 spec에 policyTypes를 Ingress로 설정한다.

그러고 ingress 섹션을 추가한다. ingress 에서 들어오는 Pod가 API Pod만이 가능하게끔 Selector를 지정한다.

ports는 DB가 사용하는 port를 지정해주면 된다.

이렇게 지정하면 Web Pod에서 들어오는 Ingress 트래픽은 모두 Network Policy에 의해 막히게 된다.

하지만 만약, Namespace 여러 군데에 API Pod가 존재한다면 Namespace를 지정해주어야 한다.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          name: prod
    ports:
    - protocol: TCP
      port: 3306
```

ingress 섹션에 namespaceSelector를 추가하여 원하는 Namespace를 지정하면 된다.

또한 Network Policy는 IP 주소로 지정이 가능하다.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          name: prod
    - ipBlock:
      cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

ingress 섹션에 ipBlock을 추가하여 원하는 IP 주소 범위를 지정해주면 된다.

그리고 하나의 Network Policy에 Ingress 규칙과 Egress 규칙을 모두 지정할 수 있다.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          name: prod
    - ipBlock:
      cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
      cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

policyTypes 섹션에 Egress를 추가한 다음 egress 섹션을 추가해주면 된다.

네트워크 트래픽을 내보낼 때 작동하는 것이므로 from이 to로 바뀐다.

그리고 포트가 다른 여러 개의 Pod에 대한 Network Policy를 지정한다면 to나 from을 추가하면 된다.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
      cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
  - to:
    - podSelector:
        matchLabels:
          name: web
      ports:
      - protocol: TCP
        port: 8080
```

# Network Policy Command

만들어진 Network Policy를 확인하기 위해서는 kubectl get networkpolicies 명령어를 사용하면 된다.

```
# kubectl get networkpolicies
```

자세한 사항을 확인하기 위해서는 kubectl describe 명령을 사용하면 된다.

```
# kubectl describe networkpolicies (NETWORK POLICY NAME)
```
