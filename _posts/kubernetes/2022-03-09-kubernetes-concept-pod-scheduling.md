---
title: \[Kubernetes\]\(Concept\)쿠버네티스 개념 - Pod의 스케줄링 구조
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Scheduling
- Pod
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Pod가 어떻게 배치되는가?
- 스케줄링: 컨테이너 어플리케이션을 구체적으로 어떤 노드에서 움직이게 할지를 알고리즘에 기초하여 배치하는 것이다.
- 이 스케줄링 구조를 이해하는 데 있어서 중요한 역할을 하는 것이 쿠버네티스의 마스터에서 움직이는 API Server이다.
- API Server는 클러스터 안의 리소스를 작성, 참조, 갱신, 삭제하기 위해 RESTful 인터페이스를 갖고 있다.
- 클러스터의 상태 데이터를 갖고 있는 etcd에 대한 액세스도 이 API Server를 통해 한다.

# Pod 스케줄링 순서

### 1. kubectl 명령 실행 - 사용자

kubectl 명령으로 포드를 작성하는 매니페스트(yaml file)를 apply 하면 Kubectl 명령은 마스터에서 움직이는 API Server를 호출한다.

### 2. Pod 정보 기록 - ETCD

그 다음 마스터의 API Server는 Pod가 작성되었다는 정보를 etcd에 기록한다.

### 3. Pod 정보의 변경을 감지 - 컨트롤러 매니저, 스케줄러

스케줄러는 클러스터 상에 포드를 배치하는 것을 정하는 컴포넌트이다. 스케줄러는 클러스터 구성에 변경이 없는지를 확인하기 위해 API Server(Pod API)를 항상 감시하고 있다.  그래서 스케줄러가 (2) 의 변화를 감지한다.

### 4. Pod를 할당 & 노드를 정한다 - 스케줄러

Pod가 작성되었다는 정보를 감지한 스케줄러는 어떤 노드에 포드를 배치할지를 결정한다.

### 5. Pod의 할당 위치 갱신 - ETCD

스케줄러가 포드를 배치할 노드를 정하면 API Server를 호출하고 etcd 상의 포드정보에 배치 노드를 갱신한다. 여기서 스케줄러의 일은 끝난다.

### 6. 포드 정보의 변경을 감지 - kubelet ↔  API

클러스터의 노드에서 움직이는 kubelet도 클러스터 구성에 변경이 없는지를 확인하기 위해 API Server를 항상 감시하고 있다. 그래서 kubelet이 '자신의 노드의 상태에 변경이 있었다' 라는것을 감지한다.

### 7. 포드 작성

구성 정보를 감지한 Kubelet은 컨테이너 런타임(e.g. docker)에 Pod 작성을 지시한다.

# 정리
흐름을 보면 알 수 있듯이 쿠버네티스에서는 각 컴포넌트가 자율적으로 움직여 마스터의 API Server를 통해 서로 연계하면서 처리를 수행하고 있다.

---
> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 