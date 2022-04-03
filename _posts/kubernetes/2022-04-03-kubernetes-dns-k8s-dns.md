---
title: \[Kubernetes\]\(DNS\)What is Kubernetes DNS
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- DNS
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Kubernetes DNS
K8s에서는 클러스터 안에서만 사용하는 DNS를 설정할 수 있다. 

그리하여 파드 사이에 통신할 때 IP가 아닌 ***도메인을 사용***할 수 있다.

특정 파드나 디플로이먼트를 도메인으로 접근하도록 설정했다면 문제가 생겨서 파드나 디플로이먼트를 재생성할 때  자동으로 변경된 파드의 IP를 도메인에 등록하고 자연스럽게 새로 실행한 파드로 연결할 수 있다.

DNS를 클라이언트나 API 게이트웨이가 호출할 서비스를 찾는 ***서비스 디스커버리*** 용도로 사용할 수도 있다.

---

# 클러스터 안에서 도메인 사용하기

## 특정 서비스에 접근하는 도메인

→ 서비스이름.네임스페이스이름.svc.cluster.local  으로 구성한다.

e.g.

namespace: aname

servcie: bservice 

domain: bservice.aname.svc.cluster.local

## 특정 파드에 접근하는 도메인

→ 파드IP주소.네임스페이스이름.pod.cluster.local

e.g.

namesapce: default, 

pod: cpod(10.10.10.10) 

domain: 10-10-10-10.default.pod.cluster.local

---

# DNS lookup in kubernetes pod

```bash
## To create a namespace
kubectl create ns apps

## To create a Pod
kubectl run nginx --image=nginx --namespace apps

## To get the additional information of the Pod in the namespace "apps"
kubectl get po -n apps -owide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          99s   10.244.1.3   node01   <none>           <none>

## To get the dns record of the nginx Pod from the default namespace
kubectl run -it test --image=busybox:1.28 --rm --restart=Never -- nslookup 10-244-1-3.apps.pod.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      10-244-1-3.apps.pod.cluster.local
Address 1: 10.244.1.3
pod "test" deleted

## Accessing with curl command
kubectl run -it nginx-test --image=nginx --rm --restart=Never -- curl -Is http://10-244-1-3.apps.pod.cluster.local
HTTP/1.1 200 OK
Server: nginx/1.19.2
```

namespace: apps

pod: 10.244.1.3

domain: 10-244-1-3.apps.pod.cluster.local