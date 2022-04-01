---
title: \[Kubernetes\]\(Ingress\)Custom nginx ingress errors
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Ingress
- Custom error pages
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
인그레스로 연결된 서비스에 없는 페이지를 호출할 경우 아래에 작성한 커스텀 에러 페이지를 표출한다

---

# git clone

아래 git에서 소스를 다운 받는다

```bash
git clone https://github.com/kenmoini/custom-nginx-ingress-errors.git
```

---

# error page 작성

/www 하위 경로에 error page를 작성

```go
/root/workspace/custom-nginx-ingress-errors/www
```

---

# Docker image build

작성한 error page와 함께 새로운 docker image를 생성한다

```go
docker build -t harbor.ta.net/ta/error-pages:v1.0 .
```

---

# Docker image push

custom image를 k8s에 배포하기 위해 registry에 push한다

```go
docker push harbor.ta.net/ta/error-pages:v1.0
```

---

# deploy, svc 생성

위에서 생성한 docker image로 pod를 배포 한다

k8s-deployment.yaml 에서 container image를  registry에 push한 내용으로 변경

→ image: harbor.ta.net/ta/error-pages:v1.0

```go
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-errors
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: nginx-errors
    app.kubernetes.io/part-of: ingress-nginx
spec:
  selector:
    app.kubernetes.io/name: nginx-errors
    app.kubernetes.io/part-of: ingress-nginx
  ports:
  - port: 80
    targetPort: 8080
    name: http
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-errors
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: nginx-errors
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx-errors
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx-errors
        app.kubernetes.io/part-of: ingress-nginx
    spec:
      containers:
      - name: nginx-error-server
        image: harbor.ta.net/ta/error-pages:v1.0
        ports:
        - containerPort: 8080
        # Setting the environment variable DEBUG we can see the headers sent
        # by the ingress controller to the backend in the client response.
        # env:
        # - name: DEBUG
        #   value: "true"
```

## 확인

nginx-errors라는 deployment가 생성된걸 확인할 수 있다

```go
root@jv0535 [~/workspace/custom-nginx-ingress-errors]k get all -n ingress-nginx
NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-68dfd94fd4-pp9tz   1/1     Running   0          36d
pod/ingress-nginx-controller-68dfd94fd4-z2bfq   1/1     Running   0          36d
pod/nginx-errors-85789dc55d-xpml2               1/1     Running   0          17s

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.102.76.250   <none>        80:30080/TCP,443:30443/TCP   51d
service/ingress-nginx-controller-admission   ClusterIP   10.106.24.115   <none>        443/TCP                      51d
service/nginx-errors                         ClusterIP   10.110.171.30   <none>        80/TCP                       17s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   2/2     2            2           51d
deployment.apps/nginx-errors               1/1     1            1           17s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-68dfd94fd4   2         2         2       36d
replicaset.apps/ingress-nginx-controller-6cb6fdd64b   0         0         0       51d
replicaset.apps/nginx-errors-85789dc55d               1         1         1       17s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           7s         51d
job.batch/ingress-nginx-admission-patch    1/1           10s        51d
```

---

# Edit nginx-controller deployment

kubectl edit deployment/ingress-nginx-controller -n ingress-nginx

arg 추가

        - --default-backend-service=$(POD_NAMESPACE)/nginx-errors

```go
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app.kubernetes.io/component
                operator: In
                values:
                - controller
            topologyKey: kubernetes.io/hostname
      containers:
      - args:
        - /nginx-ingress-controller
        - --election-id=ingress-controller-leader
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
## add
        - --default-backend-service=$(POD_NAMESPACE)/nginx-errors
```

---

# Edit configmap

k edit cm ingress-nginx-controller -n ingress-nginx

```go
apiVersion: v1
data:
  use-forwarded-headers: "true"
  custom-http-errors: "400,403,404,500,503"
```

---

# Test custom error page

없는 페이지 호출시 

www/404.html 에 작성한 페이지가 호출된다.

<center><img src="/assets/images/posts/kubernetes/ingress/ingress-custom-error-pages.png" width="150%" ></center>  
