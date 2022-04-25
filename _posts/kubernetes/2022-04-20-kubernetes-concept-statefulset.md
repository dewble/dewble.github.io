---
title: \[Kubernetes\]\(Concept\)What is Kubernetes StatefulSet(스테이트풀세트)
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
- StatefulSet
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
StatefulSet은 ***상태가 있는 파드들을 관리***하는 컨트롤러이다.

ReplicaSet, Deployment는 모두 ***상태가 없는 파드들을 관리***하는 용도였다.

StatefulSet을 사용하면, 볼륨을 사용해서 특정 데이터를 저장한 후 파드를 재시작했을 때 ***해당 데이터를 유지할 수 있다.***

Deployment와 다르게 StatefuleSet은 각 파드의 독자성을 유지한다.

→ 이 파드들은 동일한 스펙으로 생성되지만, 서로 교체는 불가능하다. 각각은 재스케줄링 간에도 지속적으로 유지되는 식별자를 가진다.

---

# When we use ****StatefulSets****

StatefulSet은 아래에 해당하는 애플리케이션에 유용하다.

- 안정되고 지속성을 가지는 고유한 네트워크 식별자
- 안정되고 지속성을 가지는 스토리지
- 순차적인 배포와 스케일링이 필요한 경우
- 순차적인 배포의 롤링 업데이트가 필요한 경우

---

# Example - StatefulSet

statefulset.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-statefulset-service
  labels:
    app: nginx-statefulset-service
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-statefulset-service
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web # Nmae of StatefulSet 
spec:
  selector:
    matchLabels:
      app: nginx-statefulset
  serviceName: "nginx-statefulset-service"
  replicas: 3 
  template:
    metadata:
      labels:
        app: nginx-statefulset
    spec:
      terminationGracePeriodSeconds: 10 # graceful의 대기 시간을 설정 
      containers:
      - name: nginx-statefulset
        image: nginx
        ports:
        - containerPort: 80
          name: web
```

```bash
$ kubectl apply -f statefulset.yaml
service/nginx-statefulset-service created
statefulset.apps/web created
```

### 서비스 확인

```bash
$ kubectl get svc,statefulset,pods
NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
service/nginx-statefulset-service   ClusterIP   None            <none>        80/TCP                   44s

NAME                   READY   AGE
statefulset.apps/web   3/3     44s

NAME                                              READY   STATUS             RESTARTS   AGE
pod/web-0                                         1/1     Running            0          85s
pod/web-1                                         1/1     Running            0          78s
pod/web-2                                         1/1     Running            0          71s
```

- UUID 형식의 접미사가 붙는 것이 아니라 web-[num] 이 된다.
- 실행은 0 → 1 → 2 순으로 되며 삭제는 2 → 1 → 0 순으로 이루어진다.

### 실행중인 스테이트풀세트의 .spec.replicas 필드 값을 높은 숫자부터 줄인 숫자 개수만큼 삭제한다.

```bash
spec:
  podManagementPolicy: OrderedReady
  replicas: 3 -> 2

[dewble@instance-1 yaml]$ kubectl edit statefulset/web
statefulset.apps/web edited
[dewble@instance-1 yaml]$ kubectl get svc,statefulset,pods | grep web
statefulset.apps/web   2/2     5m24s
pod/web-0                                         1/1     Running            0          5m23s
pod/web-1                                         1/1     Running            0          5m16s
```

web-2 가 삭제 되었다.

---

# 파드를 순서 없이 실행하거나 종료하기

- .spec.podManagementPolicy 필드를 이용해 순서를 없앨 수도 있다. (기본값은 OrderedReady)

statefulset-parallel.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-statefulset
  labels:
    app: nginx-statefulset
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-statefulset
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-parallel
spec:
  podManagementPolicy: Parallel # OrderedReady -> 순서대로 관리
  selector:
    matchLabels:
      app: nginx-statefulset
  serviceName: "nginx-statefulset-service"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-statefulset
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx-statefulset
        image: nginx
        ports:
        - containerPort: 80
          name: web
```

파드가 한꺼번에 만들어짐을 확인

```bash
$ kubectl apply -f statefulset-parallel.yaml
service/nginx-statefulset-service unchanged
statefulset.apps/web-parallel created

$ kubectl get pods | grep parallel
web-parallel-0                                1/1     Running            0          12s
web-parallel-1                                1/1     Running            0          12s
web-parallel-2                                1/1     Running            0          12s
```

---

# 특정 파드에 서비스 연결

```bash
$ kubectl describe pod web-0
Name:         web-0
Namespace:    kube-system
Priority:     0
Node:         instance-5/10.146.0.11
Start Time:   Sun, 20 Sep 2020 03:14:52 +0000
Labels:       app=nginx-statefulset
              controller-revision-hash=web-65969dfd8f
              **statefulset.kubernetes.io/pod-name=web-0**
```

- statefulset.kubernetes.io/pod-name=web-0

이 레이블을 이용하면 스테이트풀세트가 관리하는 전체 파드 중에서 특정 파드에만 서비스를 연결할 수 있다.

---

> [https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)
>