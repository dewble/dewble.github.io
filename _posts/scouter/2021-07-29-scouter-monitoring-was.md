---
title: \[Scouter\]\(Monitoring\)Monitoring - WAS
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Scouter
- Monitoring
- Java Agent
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
WAS에서 서비스되는 어플리케이션에 대한 실시간 서비스 처리현황 및 관련된 성능 데이터를 모니터링

---

# WAS Performance Counter

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-1.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-2.png" width="150%" height="150%"></center>

---

## Active Service EQ( 퀄라이저)

파랑색: 3 초 미만

주황색: 3~8초

빨간색: 8초 이상

막대바를 더블클릭하면 Active Service List를 확인할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-3.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-4.png" width="150%" height="150%"></center>

Hang이 걸려 CPU 리소스가 올라 가는 경우 Active Service를 Stop하여 해결하는 경우가 종종 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-4-1.png" width="150%" height="150%"></center>
## Perm

Class 메타 정보, 메소드 실행 코드, 상수 Pool등이 저장되는 영역

동적으로 class를 만들어낼 경우   Perm 메모리가 상승할 수 있다.

## Process Cpu

JVM 프로세스 CPU 사용률 - JAVA가 사용하는 CPU

전체 CPU와 비교해서 보면 JAVA가 많이 사용하는지 다른 요소가 영향을 주는지 확인할 수 있다.

## TPS - 초당 트랜잭션 처리건수

TPS에 맞게 부하테스트를 해야한다.

TPS가 높을 수록 좋은 시스템

## RequestProc

Request에 대해 Port 기준으로 처리시간, 수신/전송량, 에러 건수를 모니터링 할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-5.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-6.png" width="150%" height="150%"></center>

## DataSource

WAS에서 사용중인 DataSource의  Active/Idle Connection을 모니터링 할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-7.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-8.png" width="150%" height="150%"></center>

### DataSource - Conn Active

특정 DataSource에 대해 사용중인 JDBC Connection의 개수를 표시

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-9.png" width="150%" height="150%"></center>
### DataSource - Conn Active

DataSource에 대해 사용 가능한 JDBC Connection의 개수를 표시