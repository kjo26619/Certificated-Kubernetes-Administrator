# Cluster Roles and Role Bindings

4-5에서 정리한 Role과 RoleBinding은 모두 Namespace 단위의 자원들에 대한 권한을 부여하는 것이다.

Namespaced Resource는 Pods, Replicasets, Jobs, Deployments, Services, Secrets, ConfigMaps 등등이 있다.

이들은 각 Namespace마다 자원들을 가지므로 Namespace마다 권한에 대한 Role을 설정해주어야 한다.

하지만 Namespaced가 아닌 Cluster scoped Resource들에게도 권한 부여를 할 수 있어야 한다.

그래서 있는 것이 Cluster Role이다.

# Cluster Roles

Cluster scoped Resource들은 Nodes, Persistent Volume, Certificate Signing Requests(CSR), Namespaces 등등이 있다.

이를 확인하기 위해서는 kubectl api-resources 명령어를 통해서 확인할 수 있다.

```
(Namespaced Resource)
# kubectl api-resources --namespaced=true

(Cluster scoped Resource)
# kubectl api-resources --namespaced=false
```

각 사용자에게 Role을 설정하는 것은 Namespaced와 크게 다르지 않다.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

kind를 ClusterRole로 바꾸고 resources를 선택할 때 Cluster scoped Resource들로 바꾸면 된다.

RoleBinding도 마찬가지로 크게 달라지지 않는다.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

kind를 ClusterRoleBinding으로 바꾸면 되며 나머지 구성은 크게 다르지 않다.

대신, 이 구성은 모든 Namespace에게 적용된다.

Cluster Role을 확인하는 방법은 kubectl get clusterrole을 사용하면 된다.

```
# kubectl get clusterrole

# kubectl get clusterrolebindings
```
