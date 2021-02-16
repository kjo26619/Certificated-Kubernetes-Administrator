# DaemonSets

DaemonSets은 Deployment와 유사하나 그 기능이 다르다. DaemonSets는 특정한 Node 혹은 모든 Node에서 실행되고 있어야할 기본 Pod를 유지한다.

즉, DaemonSets은 특정 Pod를 Kubernetes Cluster에 참여하는 Node들에게 하나씩 모두 배치하고 유지될 수있도록 한다.

DaemonSets은 모니터링을 위해 Log를 수집하거나 기본적으로 모든 노드에서 실행되어야할 기본 Pod 등을 위해 사용된다.

무조건 노드 당 한 개의 Pod를 유지하기 때문에 Replica의 수도 없으며 Scheduler에 의해 Node 배치를 받지도 않는다.

