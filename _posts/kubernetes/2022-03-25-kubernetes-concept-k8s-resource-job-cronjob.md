---
title: \[Kubernetes\]\(Concept\)쿠버네티스 리소스 - 배치 잡 관리(Job/CronJob)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Job
- CronJob

categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 배치 잡 관리(Job/CronJob)
웹 서버와 같은 상주 서비스가 아니라 집계 등과 같은 배치 처리 또는 기계학습이나 수치해석과 같은 프로그램의 시작부터 종료까지 완료되는 잡을 실행하기 위한 리소스가 Job 또는 CronJob이다.

파드는 정지=이상 종료이지만 Job,CronJob은 정지=잡 종료라는 차이가 있다.

---

## Job

- 하나 또는 여러 개의 포드에서 처리되는 배치 잡을 실행하기 위한 리소스
- 예를 들어, DB의 마이그레이션과 같이 한번만의 잡으로 처리가 끝나는 것에 이용한다.
- 어플리케이션 오류나 예외 등으로 실패했을 때는 처리가 성공할때까지 Job 컨트롤러가 파드를 다시 만든다.

---

## CronJob

- 정해진 타이밍에 반복할 Job 실행에 사용하는 리소스
- 스토리지 백업, 메일 송신 등과 같은 처리에 사용
- Linux, Unix 시스템의 Cron과 비슷하여 매니페스트 파일에서의 지정도 Job 실행 시각이나 빈도 등을 설정할 수 있다.

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 