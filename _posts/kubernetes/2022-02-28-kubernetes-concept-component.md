---
title: \[Kubernetes\]\(Concept\)쿠버네티스의 주요 컴포넌트
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Component
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 쿠버네티스의 주요 컴포넌트
쿠버네티스 컴포넌트는 3 가지로 구분

클러스터를 관리하는데 필수인 ***마스터용 컴포넌트***

***노드용 컴포넌트***

필수는 아니지만 추가로 사용할 수 있는 ***애드온용 컴포넌트***

# 마스터 컴포넌트

### etcd

코어 OS에서 개발한 고가용성을 제공하는 키-값 저장소. 분산 시스템에서 노드 사이의 상태를 공유하는 합의 알고리즘 중 하나인 raft 알고리즘을 구현한 것.

쿠버네티스에서는 필요한 모든 데이터를 저장하는 데이터베이스 역할을 한다.

### kube-apiserver

쿠버네티스 클러스터의 API를 사용할 수 있도록 하는 컴포넌트

클러스터론 요청이 유효한지 검증한다.

수평적으로 확장할 수 있도록 설계했으므로 서버 여러 대에 여러 개의 kube-apiserver를 실행해 사용할 수 있다.

### kube-scheduler

현재 클러스터 안에서 자원 할당이 가능한 노드 중 알맞은 노드를 선택해서 새롭게 만든 파드를 실행한다.

### kube-controller-manager

쿠버네티스는 파드들을 관리하는 컨트롤러가 있다. 컨트롤러 각각을 실행하는 컴포넌트

스케일아웃 관리

### cloud-controller-manager

쿠버네티스의 컨트롤러들을 클라우드 서비스와 연결해 관리하는 컴포넌트.

---

# 노드 컴포넌트

### kubelet

모든 노드에서 실행되는 에이전트

파드 컨테이너들의 실행을 직접 관리

### Kube-proxy

쿠버네티스는 클러스터 안에 별도의 가상 네트워크를 설정하고 관리한다.

호스트의 네트워크 규칙을 관리하거나 연결을 전달할 수도 있다.

### Container Runtime

실제로 컨테이너를 실행시킨다.

docker, containerd, runc 등

---

# 에드온 컴포넌트

### 네트워킹 에드온

클러스터안에 가상 네트워크를 구성해 사용할 때 kuby-proxy 이외에 네트워킹 에드온을 사용한다.

### DNS 애드온

클러스터 안에서 동작하는 DNS 서버. 쿠버네티스 서비스에 DNS레코드를 제공한다. 쿠버네티스 안에 실행된 컨테이너들은 자동으로 DNS 서버에 등록된다.

kube-dns, CoreDNS 등

### 대시보드 애드온

[대시보드](https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/)는 쿠버네티스 클러스터를 위한 범용의 웹 기반 UI다. 사용자가 클러스터 자체뿐만 아니라, 클러스터에서 동작하는 애플리케이션에 대한 관리와 문제 해결을 할 수 있도록 해준다.

### 컨테이너 자원 모니터링

클러스터 안에서 실행 중인 컨테이너의 상태를 모니터링하는 애드온

### 클러스터 로깅

클러스터 안 개별 컨테이너의 로그와 쿠버네티스 구성 요소의 로그들을 중앙화한 로그 수집 시스템에 모아서 보는 에드온

---

> [https://kubernetes.io/ko/docs/concepts/overview/components/](https://kubernetes.io/ko/docs/concepts/overview/components/)
>