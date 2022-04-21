---
title: \[Kubernetes\]\(Deployment\)Pausing and Resuming a rollout of a Deployment - 디플로이먼트 배포 일시 중지, 배포 재개, 재시작하기
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
- Update Strategy
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
Deployment를 업데이트할 때 해당 Deployment에 대한 롤아웃을 일시 중지할 수 있다.

변경 사항을 적용할 준비가 되면, Deployment 롤아웃을 재개한다.

이러한 방법으로, 불필요한 롤아웃을 트리거하지 않고 롤아웃 일시 중지와 재개 사이에 여러 수정 사항을 적용할 수 있다. 

---

# 배포 일시 중지 - rollout pause

```bash
$ kubectl rollout pause deployment.apps/nginx-deployment
deployment.apps/nginx-deployment paused
```

```bash
$ kubectl set image deploy/nginx-deployment nginx-deployment=nginx:1.12
deployment.apps/nginx-deployment image updated
```

```bash
$ kubectl patch deployment.apps/nginx-deployment -p "{\"metadata\":{\"annotations\":{\"kubernetes.io/change-cause\":\"version 1.12\"}}}"
deployment.apps/nginx-deployment patched
```

```bash
$ kubectl rollout history deploy/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
6         <none>
10        <none>
11        version 1.10.1
12        version 1.11
```

rollout history 로 확인해보면 배포가 진행되지 않았다는 것을 알 수 있다.  
→ annotations에 작성한 version 1.12 이 기록되지 않는다.

업데이트를 pause  시켰으므로 디플로이먼트 작업이 진행되지 않는것이다.

---

# 배포 재개 - resume

```bash
$  kubectl rollout resume deployment.apps/nginx-deployment
deployment.apps/nginx-deployment resumed
$ kubectl rollout history deploy/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
6         <none>
10        <none>
11        version 1.10.1
12        version 1.11
13        version 1.12
```

annotations에 작성한 version 1.12 이 기록됐다.

---

# 재시작 - restart

```bash
$ kubectl rollout restart deployment.apps/nginx-deployment
deployment.apps/nginx-deployment restarted
$ kubectl rollout history deploy/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
6         <none>
10        <none>
11        version 1.10.1
12        version 1.11
13        version 1.12
14        version 1.12
15        version 1.12
```

annotations에 작성한 version 1.12 이 기록됐다.

---

> [https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment)
>