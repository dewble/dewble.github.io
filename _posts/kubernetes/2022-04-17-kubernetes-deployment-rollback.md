---
title: \[Kubernetes\]\(Deployment\)Rolling Back a Kubernetes Deployment - 디플로이먼트 롤백
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Deployment
- Rollback
- Rolling Update
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
때때로 디플로이먼트의 롤백을 원할 수도 있다. 예를 들어 디플로이먼트가 지속적인 충돌로 안정적이지 않은 경우. 기본적으로 모든 디플로이먼트의 롤아웃 기록은 시스템에 남아있어 언제든지 원할 때 롤백이 가능하다.

---

# Check Rollback Changes - 롤백 변경 내용 확인

Deployment를 변경한 내역은 아래 명령어로 확인할 수 있다.

```bash
kubectl rollout history deploy
```

```bash
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>
```

### 상세 내용 확인 —revision=[num]

```bash
$ kubectl rollout history deploy nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=7b779c9596
  Containers:
   nginx-deployment:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

$ kubectl rollout history deploy nginx-deployment --revision=3
deployment.apps/nginx-deployment with revision #3
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=58ddb55966
  Containers:
   nginx-deployment:
    Image:	nginx:1.10.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

Revision2,3 에서 nginx version이 변경된걸 확인할 수 있다.

---

# Deployment Rollback

```bash
$ kubectl rollout undo deploy nginx-deployment
deployment.apps/nginx-deployment rolled back
```

바로 직전 REVISION으로 Rollback을 진행한다.

```bash
[dewble@instance-1 yaml]$ kubectl get deploy,rs,rc,pods | grep nginx-deployment
deployment.apps/nginx-deployment             3/3     3            3           4h6m
replicaset.apps/nginx-deployment-58ddb55966             3         3         3       71m
replicaset.apps/nginx-deployment-69cfdf5bc7             0         0         0       4h6m
replicaset.apps/nginx-deployment-7b779c9596             0         0         0       3h50m
pod/nginx-deployment-58ddb55966-csc8s             1/1     Running   0          47s
pod/nginx-deployment-58ddb55966-vwdl8             1/1     Running   0          41s
pod/nginx-deployment-58ddb55966-wm2qk             1/1     Running   0          35s
```

58ddb55966 레플리카세트와 해당 레플리카 세트가 관리하는 파드들로 다시 되돌려졌다.

```bash
$ kubectl get deploy nginx-deployment -o=jsonpath="{.spec.template.spec.containers[0].image}{'\n'}"
nginx
```

REVISION 숫자가 변경 되었다.

```bash
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         <none>
6         <none>
7         <none>
```

특정 리비전으로 실행중인 파드로 되돌리려면 ***--to-revision=[num]*** 옵션 사용

---

# Rollback to a specific revision

```bash
$ kubectl rollout undo deploy nginx-deployment --to-revision=2
deployment.apps/nginx-deployment rolled back
```

```bash
[dewble@instance-1 yaml]$ kubectl get deploy,rs,rc,pods | grep nginx-deployment
deployment.apps/nginx-deployment             3/3     3            3           4h15m
replicaset.apps/nginx-deployment-58ddb55966             0         0         0       80m
replicaset.apps/nginx-deployment-69cfdf5bc7             0         0         0       4h15m
replicaset.apps/nginx-deployment-7b779c9596             3         3         3       3h59m
pod/nginx-deployment-7b779c9596-5nq4k             1/1     Running   0          36s
pod/nginx-deployment-7b779c9596-kgsw2             1/1     Running   0          25s
pod/nginx-deployment-7b779c9596-p2j69             1/1     Running   0          30s
```

```bash
$ kubectl get deploy nginx-deployment -o=jsonpath="{.spec.template.spec.containers[0].image}{'\n'}"
nginx:1.9.1
```

다시 해당 디플로이먼트의 이미지로 변경되었다.

```bash
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
6         <none>
7         <none>
8         <none>
```

리비전 숫자도 변경되었다.

- CHANGE_CAUSE → 해당 리비전의 주요 내용을 나타낸다.
- 해당 항목을 출력하려면 deployment.yaml에 ***.metadata.anotation*** 필드를 추가한다

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
  annotations:
    kubernetes.io/change-cause: version 1.10.1
```

### History 확인 → CHANGE-CAUSE

```bash
$ kubectl apply -f deployment-nginx.yaml
deployment.apps/nginx-deployment configured
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
6         <none>
10        <none>
11        version 1.10.1
```

+) annotation 값이 그대로 출력 된다면 아래와 같이 적용

kubectl apply -f deployment-nginx.yaml --record=false 

---

# Change Revision number Limit

.spec.revisionHistoryLimit (default = 10)

```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx-deployment
        image: nginx:1.10.1
        ports:
        - containerPort: 80
  revisionHistoryLimit: 5
```

---

# Deployment RollingUpdate option 설정

## RollingUpdateStrategy

→ .maxSurge와 .maxUnavailable 필드 설정이 필요하다

### maxSurge와 maxUnavailable 필드 설정

- .spec.strategy.rollingUpdate.maxSurge 필드 → 디플로이먼트를 이용해서 컨테이너를 배포할 때  기본 파드 개수에 여분의 파드를 몇개 더 추가할 수 있는지 설정(기존 Pod + 새 Pod의 합 개수 설정, default 25%)
- .spec.strategy.rollingUpdate.maxUnavailable → 디플로이먼트를 업데이트하는 동안 몇 개의 파드를 이용할 수 없어도 되는지 설정 (default 25%)

### type: Recreate 에서는 위 필드를 사용하지 않는다

---

> [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)
>