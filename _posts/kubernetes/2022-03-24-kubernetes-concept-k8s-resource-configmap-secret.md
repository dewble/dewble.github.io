---
title: \[Kubernetes\]\(Concept\)쿠버네티스 리소스 - 어플리케이션 설정 정보 관리(ConfigMap/Secrets)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- ConfigMap
- Secret

categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 어플리케이션 설정 정보 관리(ConfigMap/Secrets)
K8s를 사용하면 컨테이너 어플리케이션이 클러스터안의 어디에서 움직이고 있는지를 의식할 필요가 없다.

그런데 어플리케이션이 ***공통으로 이용하는 환경변수*** 등을 컨테이너 안에 넣어 버리면 환경이 바뀔 때마다 이미지를 다시 만들어야 하는 상황이 생긴다.

K8s는 이런 어플리케이션 설정 정보를 일원 관리하는 서비스가 있다 → ConfigMap, Secrets

---

## 컨피그맵(ConfigMap)

- 어플리케이션의 설정 정보, 구성파일, 명령 인수, 포트 번호, 어플리케이션 고유의 식별 정보 등을 포드에서 참조할 수 있도록 해주는 장치.
- Key-Value형으로 정보를 관리할 수 있다. 예를 들어 Nginx의 설정 파일 등 각 컨테이너에서 공통으로 만들고 싶은 것을 등록하여 일원 관리한다.
- 컨피그맵의 정보는 볼륨으로서 마운트할 수 있으므로 컨테이너 어플리케이션에서 봤을 때는 보통의 파일로 취급할 수 있다. 또 환경변수로서 취급할 수도 있다.

---

## 시크릿(Secrets)

- 컨피그맵과 마찬가지로 구성 정보를 컨테이너 어플리케이션에 전달하기 위한 것이지만, DB와 연결할 때의 비밀번호나 OAuth 토큰과 같은 기밀성이 높은 정보를 관리할 때 이용한다.
- 바이너리 데이터도 저장할 수 있도록 데이터를 ***base64로 인코딩하여 등록***해야 한다.
- 시크릿의 데이터는 메모리 상(tmpfs)에 전개되어 디스크에는 기록되지 않는다는 특징을 갖고 있다.
- 또, K8s1.7 이후 버전에서는 암호화되어 etcd로 관로된다.
- 프라이빗한 컨테이너 이미지로부터 이미지를 다운로드할 때 인증 정보를 전달하는 경우에 사용한다.

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 