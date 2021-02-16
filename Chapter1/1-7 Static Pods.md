# Static Pods

Staitc Pods는 kube-apiserver에 의해 관리되지 않고 Node에 있는 Kubelet에 의해서 직접 관리되는 Pods를 말한다.

즉, Cluster에 의해서 관리되지 않고 각 Node들이 따로 관리하고 있는 Pods이다.

이러한 Pods는 kube-apiserver에 의해서 확인해 볼 수는 있다. 왜냐하면 Kubelet에서 Static Pods에 대해서 kube-apiserver에게 Mirror Pod를 생성해서 내용을 보내주기 때문이다.

즉, 복제본을 API 서버가 받아서 확인할 수 있는 것이다. 그렇기 때문에 API 서버가 제어할 수는 없다.

이러한 Static Pods는 처음 Cluster를 구성하면서 Control Plane을 의한 controller-manager, kube-apiserver, etcd 등을 만들 때 사용한다. 

정확한 사항은 https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/ 에서 확인할 수 있다.

# Multiple Scheduler

기본적으로 Kubernetes에서는 kube-scheduler를 제공해주지만 자신이 원하는대로 변경한 Scheduler를 여러 개 사용할 수 있도록 지원하고 있다.

이는 Static Pods를 통해 만들며 각 Pod들을 만들 때 사용할 Scheduler를 지정해줄 수 있다.

정확한 사항은 https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/ 에서 확인할 수 있다.
