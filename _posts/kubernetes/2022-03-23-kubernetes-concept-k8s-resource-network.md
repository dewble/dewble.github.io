---
title: \[Kubernetes\]\(Concept\)쿠버네티스 리소스 - 네트워크 관리(Service/Ingress)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Network
- Service
- Ingress
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 쿠버네티스 리소스
쿠버네티스는 유연한 어플리케이션 실행 환경 관리를 소프트웨어로 수행하기 때문에 다양한 것들을 추상화하고 있다. 이렇게 추상화한 것을 쿠버네티스에서는 리소스라고 부른다.

--- 

# 네트워크 관리(Service/Ingress)
클러스터 안의 파드에 대해 클러스터 내부 및 외부 네트워크로부터 액세스하기 위한 리소스가 Service 및 Ingress이다.

---

## 서비스(Service)

- K8s 클러스터 안에서 실행된 파드에 대해 접근할 때는 서비스(Service)를 정의한다.
- 서비스는 K8s의 네트워크를 관리하는 것으로, 몇 가지 종류가 있다.
- 그 중에서도 로드밸런서는 서비스에 대응하는 IP 주소 + 포트 번호에 액세스하면 여러 포드에 대한 L4 레벨 부하분산을 수행한다.
- 서비스에 의해 할당되는 IP 주소에는 Cluster IP와 External IP가 있다.
- Cluster IP는 클러스터 안의 파드끼리 통신하기 위한 프라이빗 IP이다. 클러스터 안의 파드에서 Cluster IP로 가는 패킷은 ***노드 상의 Proxy 데몬이 받아, 수신처 포드로 전송***된다.
- External IP는 클러스터 ***외부에 공개하는 IP*** 주소이다. 파드를 새로 실행시키면 기존 서비스의 IP주소와 포트 번호는 환경변수로 참조할 수 있다.

---

## 인그레스(Ingress)

- 인그레스는 클러스터 외부의 네트워크로부터 액세스를 받기 위한 오브젝트로, HTTP/HTPPS의 엔드포인트로서 기능한다.
- 로드밸런서, URL 경로에 대응하는 백엔드 서비스에대한 라우팅, SSL, 네임스페이스의 버추얼 호스팅 등 L7의 기능을 제공한다.

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 