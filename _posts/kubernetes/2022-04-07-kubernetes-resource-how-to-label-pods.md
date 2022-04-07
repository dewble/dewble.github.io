---
title: \[Kubernetes\]\(Concept\)How to label pods
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Label
- Pod
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 라벨(Label)
개발이 진행되면 쿠버네티스 클러스터 안에는 파드가 많이 생성된다.

이것들을 클러스터에서 관리할 때 동일한 그룹으로 모아서 조작할 수 있다면 편리할 것이다.

하지만 폴더와 같은 계층 구조로 만들어 버리면, 결제 처리를 하는 백엔드 처리에서 스테이징 환경과 같이 여러 속성을 갖고 있는 경우등을 관리해야 하기 때문에 운용이 복잡해진다.

그래서 나온 개념이 라벨 이다.

# 포드에 라벨 설정

라벨은 Key-Value 형으로 설정한다. 

Key와 Value 둘다 문자열로 지정한다. 

Key 명은 필수이며, 63자 이내로 지정. 맨 앞과 맨 뒤는 반드시 알파벳이나 숫자로 지정한다. 그 외에는 '-', '_', '.' 도 사용할 수 있다.

Key 에는 슬래시로 구분한 prefix도 지정할 수 있다. prefix를 지정할 때는 DNS 서브도메인에서 길이 253자의 제한이 있다.

Nginx가 움직이는 심플한 파드 매니페스트 파일을 작성하여 동작을 확인해 보자.

## 라벨 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-a
  labels:
    env: test
    app: photo-view
spec:
  containers:
  - image: nginx
    name: photoview-container
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-b
  labels:
    env: test
    app: imagetrain
spec:
  containers:
  - image: nginx
    name: photoview-container
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-c
  labels:
    env: test
    app: prediction
spec:
  containers:
  - image: nginx
    name: photoview-container
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-d
  labels:
    env: stage
    app: photo-view
spec:
  containers:
  - image: nginx
    name: photoview-container
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-e
  labels:
    env: stage
    app: imagetrain
spec:
  containers:
  - image: nginx
    name: photoview-container
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-f
  labels:
    env: prod
    app: photo-view
spec:
  containers:
  - image: nginx
    name: photoview-container
```

라벨은 여러 개 설정할 수 있다.

### --show-labels 명령어로 설정한 label을 확인할 수 있다

```bash
kubectl apply -f Label/label-pod.yaml 
pod/nginx-pod-a created
pod/nginx-pod-b created
pod/nginx-pod-c created
pod/nginx-pod-d created
pod/nginx-pod-e created
pod/nginx-pod-f created

kubectl get pod --show-labels      
NAME          READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod-a   1/1     Running   0          49s   app=photo-view,env=test
nginx-pod-b   1/1     Running   0          49s   app=imagetrain,env=test
nginx-pod-c   1/1     Running   0          49s   app=prediction,env=test
nginx-pod-d   1/1     Running   0          49s   app=photo-view,env=stage
nginx-pod-e   1/1     Running   0          49s   app=imagetrain,env=stage
nginx-pod-f   1/1     Running   0          49s   app=photo-view,env=prod
```

## 라벨을 변경해본다

라벨을 변경하려면 매니페스트 파일을 변경한다. 작성한 포드 'nginx-pod'a'의 env 값을 'test' 에서 'stage'로 변경한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-a
  labels:
    env: stage
    app: photo-view
spec:
  containers:
  - image: nginx
    name: photoview-container
```

다음 명령을 실행하여 포드의 라벨 변경, 라벨 확인

```bash
kubectl apply -f label-pod.yaml
pod/nginx-pod-a configured
pod/nginx-pod-b unchanged
pod/nginx-pod-c unchanged
pod/nginx-pod-d unchanged
pod/nginx-pod-e unchanged
pod/nginx-pod-f unchanged

kubectl get pod --show-labels
NAME          READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod-a   1/1     Running   0          31m   app=photo-view,env=stage
nginx-pod-b   1/1     Running   0          31m   app=imagetrain,env=test
nginx-pod-c   1/1     Running   0          31m   app=prediction,env=test
nginx-pod-d   1/1     Running   0          31m   app=photo-view,env=stage
nginx-pod-e   1/1     Running   0          31m   app=imagetrain,env=stage
nginx-pod-f   1/1     Running   0          31m   app=photo-view,env=prod
```

kubectl label deployments 명령을 실행하여 라벨을 덮어 쓸 수도 있지만 구성 관리의 관점에서 볼 때 매니페스트를 변경하는 것을 권장한다.

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 