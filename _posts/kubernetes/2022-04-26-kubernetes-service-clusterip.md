---
title: \[Kubernetes\]\(Service\)How to use Kubernetes ClusterIP
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Service
- ClusterIP
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
<center><img src="/assets/images/posts/kubernetes/service/service-clusterip.png" width="150%" ></center>  

사용자는 서비스IP(ClusterIP)로 파드에 바로 접근할 수 없고 같은 클러스터안의 ***파드나 노드에서*** 접근할 수 있다.

기본 서비스 타입. 쿠버네티스 클러스터 안에서만 사용할 수 있다.

---

# 기본 구조

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP 
  clusterIP: 10.0.10.10 
  selector: 
    app: MyApp
  ports: 
  - protocol: TCP
    port: 80
    targetPort: 9376
```

`.spec.type`: 서비스 타입 설정

`.spec.ClusterIP`: 클러스터 IP를 직접 설정, 설정하지 않으면 자동 할당한다.

`.spec.selector`: 서비스와 연결할 파드에 설정한 .labels 필드 값을 설정한다.

`.spec.port`s: 서비스에서 한꺼번에 포트 여러 개를 외부에 제공할 때는 이 하위 필드에 설정한다.

---

# Example - ClusterIP

### 파드 생성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-for-svc # pod name
  labels:
    app: nginx-for-svc
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-for-svc # .spec.template.labels.app 과 맞춘다. 
  template:
    metadata:
      labels:
        app: nginx-for-svc
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f service.yaml

$ kubectl get po | grep nginx-for-svc
nginx-for-svc-66b6c48dd5-bjndc                1/1     Running     0          5m6s
nginx-for-svc-66b6c48dd5-c5452                1/1     Running     0          5m6s
```

`.metadata.name`: pod name

`.metadata.labels.app`: 서비스에 사용할 레이블. key : app, value: nginx-for-svc

### ***ClusterIP 서비스 생성***

```bash
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP 
  selector:
    app: nginx-for-svc 
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

`.spec.selector.app`:  앞에서 실행한 nginx-for-svc 파드를 선택하도록 했다.


```bash
$ kubectl apply -f service-clusterip.yaml
$ kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
clusterip-service           ClusterIP   10.233.54.164   <none>        80/TCP                   11s
```

CLUSTER-IP 가 10.233.39.192 으로 클러스터 IP가 생성되었다.

```bash
$ kubectl describe service clusterip-service
Name:              clusterip-service 
Namespace:         kube-system
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-for-svc 
Type:              ClusterIP
IP:                10.233.54.164
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.233.118.224:80,10.233.125.121:80 
Session Affinity:  None
Events:            <none>
```

Name :  서비스 이름

Selector : 레이블이 app=nginx-for-svc 인 파드 선택

Endpoints : 실제로 이 서비스에 연결된 파드

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/#publishing-services-service-types](https://kubernetes.io/ko/docs/concepts/services-networking/service/#publishing-services-service-types)
>

