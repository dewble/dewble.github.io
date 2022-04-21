---
title: \[Kubernetes\]\(StatuefulSet\)Update strategy
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- StatefulSet
- Update Strategy
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
스테이트풀셋의 `.spec.updateStrategy`  필드는 스테이트풀셋의 파드에 대한 컨테이너, 레이블, 리소스의 요청/제한 그리고 주석에 대한 자동화된 롤링 업데이트를 구성하거나 비활성화할 수 있다. 두 가지 가능한 전략이 있다.

### RollingUpdate

파드를 순차적으로 업데이트 한다.

### OnDelete

파드를 자동으로 업데이트하지 않는다. 사용자는 컨트롤러가 스테이트풀셋의 `.spec.template`를 반영하는 수정된 새로운 파드를 생성하도록 수동으로 파드를 삭제해야 한다.

---

# rollingUpdate (default)

```yaml
#중간생략
spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx-statefulset
        image: nginx
        ports:
        - containerPort: 80
          name: web
        env:
        - name: testenv
          value: testvalue01
```

env 내용을 추가하고 재배포한다.

```bash
$ kubectl apply -f statefulset-env.yaml
service/nginx-statefulset-service unchanged
statefulset.apps/web configured
```

### 재시작 순서 확인

```bash
$ kubectl get pods | grep web
web-0                                         1/1     Running            0          10s
web-1                                         1/1     Running            0          29s
web-2                                         1/1     Running            0          42s
web-parallel-0                                1/1     Running            0          12m
web-parallel-1                                1/1     Running            0          12m
web-parallel-2                                1/1     Running            0          12m
```

2→1→0 순으로 파드가 재시작 된 것을 알 수 있다.

## partition option

```yaml
$ kubectl edit statefulset web
statefulset.apps/web edited
## 중략
updateStrategy:
    rollingUpdate:
      partition: 1
    type: RollingUpdate
```

- `updateStrategy.rollingUpdate.partition`: 1 추가

### 업데이트 테스트를 위한 환경변수 변경

```yaml
metadata:
      creationTimestamp: null
      labels:
        app: nginx-statefulset
    spec:
      containers:
      - env:
        - name: testenv
          value: testvalue01->04
```

### 환경변수 확인

```bash
[dewble@instance-1 yaml]$ kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{.spec.containers[0].env}{'\n'}{end}" |grep web
web-0[{"name":"testenv","value":"testvalue01"}]
web-1[{"name":"testenv","value":"testvalue04"}]
web-2[{"name":"testenv","value":"testvalue04"}]
```

- `.spec.updateStrategy.rollingUpdate.partition` 필드 값 1보다 작은 번호의 파드들의 변경 사항을 ***업데이트하지 않는다.***
- partition 필드값이 replicas 필드값 보다 크면 `.spec.template` 필드에 변경 사항이 있더라도 업데이트하지 않는다.

---

# OnDelete

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
  name: web-ondelete
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
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx-statefulset
        image: nginx
        ports:
        - containerPort: 80
          name: web
        env:
        - name: testenv
          value: testvalue01
  **updateStrategy:
    type: OnDelete**
```

```bash
[dewble@instance-1 yaml]$ kubectl apply -f statefulset-ondelete.yaml
service/nginx-statefulset-service unchanged
statefulset.apps/web-ondelete created
```

env 변경

```yaml
[dewble@instance-1 yaml]$ kubectl edit statefulset web-ondelete
statefulset.apps/web-ondelete edited
containers:
      - env:
        - name: testenv
          value: testvalue02
```

env 변경 확인

```bash
[dewble@instance-1 yaml]$ kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{.spec.containers[0].env}{'\n'}{end}" |grep web
web-ondelete-0[{"name":"testenv","value":"testvalue01"}]
web-ondelete-1[{"name":"testenv","value":"testvalue01"}]
web-ondelete-2[{"name":"testenv","value":"testvalue01"}]
```

test value가 변경되지 않았다.

pod 삭제후 확인

```bash
[dewble@instance-1 yaml]$ kubectl delete pods web-ondelete-2
pod "web-ondelete-2" deleted
[dewble@instance-1 yaml]$ kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{.spec.containers[0].env}{'\n'}{end}" |grep web
web-ondelete-0[{"name":"testenv","value":"testvalue01"}]
web-ondelete-1[{"name":"testenv","value":"testvalue01"}]
web-ondelete-2[{"name":"testenv","value":"testvalue02"}]
```

web-ondelete-2 의 env 값이 변경된 것을 알 수 있다.

---

> [https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)
>