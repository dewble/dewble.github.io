---
title: \[Kubernetes\]\(Concept\)데몬세트(DaemonSet)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Controller
- DaemonSet
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
<center><img src="/assets/images/posts/kubernetes/daemonset/daemonset.png" width="150%"></center>  
DaemonSet은 보통 로그 수집기를 실행하거나 노드를 모니터링하는 모니터링용 데몬 등 ***클러스터 전체에 항상 실행***시켜두어야 하는 파드에 사용한다.

클러스터 전체 노드에 특정 파드를 실행할 때 사용하는 컨트롤러

클러스터 안에 새롭게 노드가 추가되었을 때 데몬세트가 자동으로 해당 노드에 파드를 실행시킨다.

반대로 노드가 클러스터에서 빠졌을 때는 해당 노드에 있던 파드는 그대로 사라질 뿐 다른곳으로 옮겨가서 실행되지 않는다.

---

# Example - DaemonSet

로그 수집기를 실행하는 데몬세트 설정

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system 
  labels:
    k8s-app: fluentd-logging 
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  updateStrategy:
    type: RollingUpdate # DaemonSet 업데이트 전략
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: fluent/fluentd-kubernetes-daemonset:elasticsearch
        env:
        - name: testenv
          value: value
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

### DaemonSet 업데이트 전략

데몬세트의 파드를 업데이트는 이 필드를 설정한다.

- OnDelete/RollingUpdate 두 가지 값중 하나를 선택할 수 있다.
    - RollingUpdate → 템플릿을 변경했을 때 바로 변경 사항을 반영. 지정한 개수만큼
    - OnDelete → 데몬셋 템플릿을 업데이트한 후, 이전 데몬셋 파드를 수동으로 삭제할 때만 새 데몬셋 파드가 생성된다.
- .spec.minReadySeconds 필드를 추가로 설정하면 한번에 교체하는 파드 개수를 조정할 수 있다.
    - 실행하는 파드가 준비 상태가 되는 최소 시간. 기본 값은 0 이라 파드가 준비되는 대로 사용할 수 있다.
- .spec.updateStrategy.rollingUpdate.maxUnavailable 필드는 한번에 삭제하는 파드 개수.

```bash
$ kubectl apply -f daemonset.yaml
daemonset.apps/fluentd-elasticsearch created
```

kube-system 네임스페이스에 데몬세트가 만들어 졌다.

```bash
$ kubectl get daemonset -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
calico-node             5         5         5       5            5           <none>                   5d10h
fluentd-elasticsearch   2         2         1       2            1           <none>                   43s
kube-proxy              5         5         5       5            5           kubernetes.io/os=linux   5d10h
nodelocaldns            5         5         5       5            5           <none>                   5d10h
```

---

# DaemonSet의 파드 업데이트 방법 변경하기

### RollingUpdate 로 변경

kubectl edit 으로 DaemonSet 변경

```bash
kubectl edit daemonset fluentd-elasticsearch -n kube-system
# 중간 생략
- env:
        - name: testenv
          value: value -> value01

```

변경 확인

```bash
kubectl describe daemonset/fluentd-elasticsearch -n kube-system
# 중간 생략
Environment:
      testenv:  value01
    Mounts:     <none>
  Volumes:      <none>

```

RollinUpdate이므로 파드를 즉시 재시작한다.

### OnDelete 로 변경

```yaml
# 중략
spec:
      containers:
      - env:
        - name: testenv
          value: value02

# 중략
updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: OnDelete
```

변경 이후 value03으로 수정 후 확인

```bash
# 중략
spec:
      containers:
      - env:
        - name: testenv
          value: value03
```

```bash
$ kubectl get daemonset/fluentd-elasticsearch -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-elasticsearch   2         2         0       0            0           <none>          12m
```

UP-TO-DATA 항목이 0이므로 변경은 했지만 최신 설정으로 변경되지 않았다는 뜻이다.

### 변경 확인

```bash
$ kubectl delete pods fluentd-elasticsearch-ftfw8 -n kube-system
pod "fluentd-elasticsearch-ftfw8" deleted

$ kubectl get daemonset/fluentd-elasticsearch -n kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-elasticsearch   2         2         1       1            1           <none>          15m
```

실행중인 pod을 삭제하면, 

UP-TO-DATA 항목이 1로 변경되어서 데몬세트가 최신 상태로 변경되었음을 알 수 있다.

```bash
$ kubectl get pods -n kube-system | grep fluentd-elasticsearch
fluentd-elasticsearch-x248m                   1/1     Running            4          2m26s
```

새로운 파드가 만들어져 실행 중인 것을 확인할 수 있다.

---

> [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
[https://kubernetes.io/ko/docs/tasks/manage-daemon/update-daemon-set/](https://kubernetes.io/ko/docs/tasks/manage-daemon/update-daemon-set/)
>