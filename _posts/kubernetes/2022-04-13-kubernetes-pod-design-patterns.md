---
title: \[Kubernetes\]\(Pod\)PODs Design Patterns - POD 구성 패턴
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Pod
- Design Patterns
- Sidecar
- Ambassador
- Adapter
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Design Patterns type
- 사이드카 패턴(Sidecar)  
- 앰배서더 패턴(Ambassador) 
- 어댑터 패턴(Adapter)  

---

# 사이드카 패턴(Sidecar)

원래 사용하려던 기본 컨테이너의 ***기능을 확장하거나 강화***하는 용도의 컨테이너를 추가하는 것

기본 컨테이너는 원래 목적의 기능에만 충실하도록 구성하고, 나머지 공통 부가 기능들은 사이드카 컨테이너를 추가해서 사용한다.

### e.g. 웹 서버 컨테이너 - 로그 수집 컨테이너

<center><img src="/assets/images/posts/kubernetes/pod/design-patterns-1.png" width="150%"></center>  

- 웹 서버 컨테이너는 웹 서버 역할만 하고 로그는 파일로 남긴다.
- 그럼 사이드카 역할인 로그 수집 컨테이너는 파일 시스템에 쌓이는 로그를 수집해서 외부의 로그 수집 시스템으로 보낸다.
- 이렇게 구성하면 웹 서버 컨테이너를 다른 역할을 하는 컨테이너로 변경했을 때도 로그 수집 컨테이너는 그대로 사용할 수 있다.
- 즉, 공통 역할을 하는 컨테이너의 재사용성을 높일 수 있다.

---

# 앰배서더 패턴(Ambassador)

파드 안에서 프록시 역할을 하는 컨테이너를 추가하는 패턴. 

파드 안에서 외부 서버에 접근할 때 내부 프록시에 접근하도록 설정하고 실제 외부와의 연결은 프록시에서 알아서 처리한다.

### e.g. 웹 서버 컨테이너 - 프록시 컨테이너

<center><img src="/assets/images/posts/kubernetes/pod/design-patterns-2.png" width="150%"></center>  


- 웹 서버 컨테이너는 캐시에 localhost로만 접근하고 실제 외부 캐시 중 어디로 접근할지는 프록시 컨테이너가 처리한다.
- 이런 방식으로 파드의 트래픽을 더 세밀하게 제어할 수 있다.

앰버서더 패턴을 이용한 파드 트래픽 제어

<center><img src="/assets/images/posts/kubernetes/pod/design-patterns-3.svg" width="150%"></center>  
  
  
파드마다 프록시를 추가해서 트래픽을 처리하도록 구성하는 것임을 알 수 있다.  

---

# 어댑터 패턴(Adapter)

- 파드 외부로 노출되는 정보를 표준화하는 어댑터 컨테이너를 사용한다.
- 주로 어댑터 컨테이너로 파드의 모니터링 지표를 표준화한 형식으로 노출시키고, 외부의 모니터링 시스템에서 해당 데이터를 주기적으로 가져가서 모니터링하는데 이용한다.

### e.g. 웹 서버 컨테이너 - 모니터링 컨테이너

<center><img src="/assets/images/posts/kubernetes/pod/design-patterns-4.png" width="150%"></center>  


---

> [https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)
> 
> [https://istio.io/latest/docs/ops/deployment/architecture/](https://istio.io/latest/docs/ops/deployment/architecture/)
>