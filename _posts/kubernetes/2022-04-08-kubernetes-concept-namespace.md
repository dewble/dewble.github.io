---
title: \[Kubernetes\]\(Concept\)Kubernetes Namespace - 쿠버네티스 리소스 분리
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Namespace
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
쿠버네티스에는 리소스를 모아서 가상적으로 분리하는 ***네임스페이스*** 라는 기능이 있다. 

이를 사용하면 하나의 쿠버네티스 클러스터를 여러 프로젝트에서 이용할 수 있다. 

여러 리소스를 모아서 넣는 폴더와 같은 것이라고 이해하면 된다.

이 네임스페이스는 ***롤베이스로 권한을 설정***할 수 있다. 네임스페이스별로 필요한 액세스 권한을 설정하여 이용할 수 있는 사용자를 제한하면 보안성을 높일 수 있다.

쿠버네티스에서 ***동일한 네임스페이스*** 안에서는 ***리소스명을 고유하게*** 해야 한다.

---

# Namespace 사용법

네임스페이스의 사용법을 살펴보자. 클러스터 안의 네임스페이스 목록을 확인

```bash
kubectl get namespace
NAME              STATUS   AGE
default           Active   6d19h
kube-node-lease   Active   6d19h
kube-public       Active   6d19h
kube-system       Active   6d19h
```

AKS(Azure Kubernetes Service)를 사용하여 클러스터를 구축했을 때는 기본적으로 위와 같이 작성 된다.

kubectl 명령에서 —namespace(-n) 을 설정하면 지정한 네임스페이스로 관리되는 쿠버네티스 리소스를 확인할 수 있다.

```bash
kubectl get pod --namespace kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6c66fc4fcb-5fvpc               1/1     Running   0          6d19h
coredns-6c66fc4fcb-zp2tz               1/1     Running   0          6d19h
coredns-autoscaler-76968fdcf-5nc9s     1/1     Running   0          4d15h
kube-proxy-ftfgt                       1/1     Running   0          6d19h
kube-proxy-hzw77                       1/1     Running   0          6d19h
kube-proxy-w6sl7                       1/1     Running   0          6d19h
kubernetes-dashboard-9f5bf9974-c8j6j   1/1     Running   0          6d19h
metrics-server-86c6bcf7df-mdqd2        1/1     Running   0          6d19h
tunnelfront-5bd478759-wqxvl            1/1     Running   0          6d19h
```

---

# 새로운 Namespace 생성

새로운 네임스페이스를 만들 때는 매니페스트 파일을 작성, 생성 한다.

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: trade-system
```

```bash
kubectl create -f namespace.yaml 
namespace/trade-system created

kubectl get namespace      
NAME              STATUS   AGE
default           Active   6d19h
kube-node-lease   Active   6d19h
kube-public       Active   6d19h
kube-system       Active   6d19h
trade-system      Active   66s
```

새로운 namespace 가 생성된 것을 확인 할 수 있다.

namespace 옵션을 사용하여 명시적으로 지정하지 않을 경우 'default' 네임스페이스에 포함된다.

---

# Context 설정

다음 명령으로 네임스페이스가 'trade-system'인 my-context'를 설정할 수 있다.

```bash
kubectl config set-context my-context --namespace=trade-system
Context "my-context" created.
```

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 