---
title: \[Kubernetes\]\(Concept\)What is a workload?
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Workloads
- Deployment
- ReplicaSet
- DaemonSet
- StatefulSet
- Job
- CronJob
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
워크로드는 쿠버네티스에서 구동되는 애플리케이션이다.  
사용자를 대신하여 파드 집합을 관리한다.  
지정한 상태와 일치하도록 올바른 수의 올바른 파드 유형이 실행되고 있는지 확인하는 컨트롤러를 구성한다.  

---

# Type of workloads

## 디플로이먼트(Deployment), 레플리카세트(ReplicaSet)

오랜 시간 동안 계속 실행되어야 하는 파드들을 관리하는 워크로드

## 데몬세트(DaemonSet)

클러스터의 전체 노드에 같은 파드를 실행하는 워크로드

## 스테이트풀세트(StatefulSet)

보통 상태가 없는 앱을 실행하는데 사용하는 컨테이너를 ***상태가 있는 앱을 실행***할 때 사용하도록하는 워크로드

## 잡(Job)

1회성 작업(단 한번의 작업)에 사용하는 워크로드

## 크론잡(CronJob)

주기적인 배치 작업을 실행할 때 사용하는 워크로드

---

> [https://kubernetes.io/ko/docs/concepts/workloads/](https://kubernetes.io/ko/docs/concepts/workloads/)
>