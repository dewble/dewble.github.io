---
title: \[Kubernetes\]\(Concept\)쿠버네티스 개념 - 스케줄링과 디스커버리
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
- Discovery
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 스케줄링과 디스커버리
기존 시스템에서는 어디에서 어떤 어플리케이션이 움직이고 있는지 알고 있어야 했다. 하지만 쿠버네티스에서는 어플리케이션이 배포되는 위치가 쿠버네티스에 의해 정해진다. 어플리케이션이 배포되면 쿠버네티스가 클러스터 상에서 비어 있는 위치를 찾아내서 자동으로 배치한다. 

그래서 ***어플리케이션을 적절한 곳에 배포하고***, ***배포된 어플리케이션이 어디에 있는지 찾아내는*** 장치가 있다.

# 스케줄링

어플리케이션을 적절한 곳에 배포하는 장치

다양한 요구사항을 매니페스트 파일로 정의하여 이를 바탕으로 클러스터 안의 적절한 위치에 어플리케이션을 자동으로 배치해 간다.

쿠버네티스에서 스케줄링은 Kubelet이 파드를 실행할 수 있도록 파드가 노드에 적합한지 확인하는 것을 말한다.



# 서비스 디스커버리

일반적인 웹 어플리케이션의 경우 리퀘스트를 받은 프론트엔드 어플리케이션이 사용자의 트랜잭션을 처리하기 위해 백엔드 서비스를 호출한다.

이때 배포된 어플리케이션이 어디에 있는지를 찾아내는 장치를 서비스 디스커버리라고 한다.

쿠버네티스에서는 클러스터 안에 구성 레지스트리를 갖고 있어서 이를 바탕으로 서비스 디스커버리를 동적으로 수행한다.

---

> [https://kubernetes.io/ko/docs/concepts/scheduling-eviction/kube-scheduler/](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/kube-scheduler/)
> 

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=)
>