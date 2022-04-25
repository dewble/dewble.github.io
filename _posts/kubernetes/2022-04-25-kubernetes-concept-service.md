---
title: \[Kubernetes\]\(Concept\)What is a Service
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Service
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
파드 집합에서 실행중인 애플리케이션을 네트워크 서비스로 노출하는 추상화 방법

여러개 파드에 접근할 수 있는 고유한 IP를 제공한다. 

파드 집합에 대한 단일 DNS명을 부여한다.

다양한 기능을 제공하지만 본질적으로 ***로드밸런서*** 역할을 한다.

---

# 개념

동적으로 변하는 파드들에 고정적으로 접근할 때 사용하는 방법이 ***쿠버네티스 서비스*** 이다.

서비스를 사용하면 파드가 클러스터 안 어디에 있든 ***고정 주소***를 이용해 접근할 수 있다.

인그레스로도 접근 할 수 있고 아래와 같은 차이가 있다.

서비스 → L4 영역 통신에서 사용

인그레스 → L7 영역 통신에서 사용  

<center><img src="/assets/images/posts/kubernetes/service/concept-of-service.png" width="150%" ></center>  


---

# 서비스 타입

## ClusterIP

기본 서비스 타입. 쿠버네티스 클러스터 안에서만 사용할 수 있다.

클러스터 안 ***노드나 파드에서***는 클러스터 IP를 이용해서 서비스에 연결된 파드에 접근한다.

## NodePort

서비스 하나에 ***모든 노드의 지정된 포트***를 할당한다. 

e.g. Node1:30080, Node2:30080 처럼 노드에 상관 없이 서비스에 지정된 포트만 사용하면 파드에 접근할 수 있다.

특이한점은 node1에만 실행되어 있는 파드가 있을때도 node2:30080 으로 접근했을 때 node1에 실행된 파드로 연결한다.

클러스터 외부에서 클러스터 안 파드로 접근할 때 사용할 수 있는 가장 간단한 방법이다.

## LoadBalancer

퍼블릭 클라우드 서비스, 프라이빗 클라우드, 쿠버네티스를 지원하는 로드밸런서 장비에서 사용.

클라우드에서 제공하는 로드밸런서와 파드를 연결한 후 로드밸런서의 IP를 이용해 클러스터 외부에서 파드에 접근할 수 있도록 한다. 

EXTERNAL-IP 항목에 로드밸런서 IP를 표시한다. 이 IP를 사용해 클러스터 외부에서 파드에 접근한다.

## ExternalName

서비스를 .spec.externalName 필드에 설정한 값과 연결한다. 클러스터안에서 외부에 접근할 때 주로 사용.

이 서비스로 클러스터 외부에 접근하면 설정해둔 CNAME값을 이용해 클러스터 외부에 접근할 수 있다.

클러스터 외부에 접근할 때 사용하는 값이므로 설정할 때 셀렉터(.spec.selector 필드)가 필요 없다.

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
>