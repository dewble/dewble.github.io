---
title: \[Scouter\]\(Monitoring\)Monitoring - WAS (XLog)
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
- XLog
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
응답시간 분포도(XLog) 및 트랜잭션 프로파일링 기능 제공

# XLog

Active Service가 종료되면 XLog차트에 하나의 점으로 표시된다.

## XLog화면에서의 다양한 기능


<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-1.png" width="150%" height="150%"></center>

## Filter

필터링 기능을 통해 다양한 방법으로 서비스를 선별할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-2.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-3.png" width="50%" height="50%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-4.png" width="150%" height="150%"></center>

## Y Axis

서비스 분포도의 기준을 변경할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-5.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-6.png" width="150%" height="150%"></center>

## Summary

화면에 찍힌 XLog의 분포에 대한 요약정보를 볼 수 있다. 

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-7.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-8.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-9.png" width="150%" height="150%"></center>

## Load

지난 Xlog 이력을 스카우터 서버로 부터 불러와 XLog를 다시 그린다.

로드 후 마우스 드래그, 필터링, Summary기능 이용이 가능하다

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-10.png" width="150%" height="150%"></center>

## Search

다양한 조건으로 XLog조회

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-11.png" width="50%" height="50%"></center>

Normal Search: 조건에 맞게 순차로 검색하여 결과를 보여준다.

Quick Search: 인덱싱 되어 있는 txid 혹은 gxid로 결과를 보여준다.

## XLog 마우스로 선택하여 분석

응답시간이 길거나, 에러로 판별된 XLog를 분석하는데 효과적이다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-12.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-13.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-14.png" width="150%" height="150%"></center>

## XLog 서비스 프로파일링

XLog 리스트 중 하나를 선택하면 프로파일링 뷰를 볼 수 있다.

이를 통해 한 트랜잭션이 처리되는 동안 아래 정보를 확인할 수 있다.

- DB Connection Get Time, SQL 및 SQL 수행시간, ResultSet Fetch Time
- Socket 접속, 외부 API Call (REST 서비스 호출, http 호출)

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-15.png" width="150%" height="150%"></center>

### SQL Statistics

프로파일 내의 SQL 중복 카운팅을 하고, SQL 별로 Bind 변수를 확인 할 수 있다.

### Bind SQL Parameter

포로파일 뷰에서 SQL 파라미터를 SQL문에 대입시켜 보여준다.

### Save Full Profile

프로파일 스텝 수가 10240(10블럭)을 넘어가면 10240 라인만 화면에 나오고 ** Profile Truncated ** 이라고 나오는데, 전체 프로파일을 보고 싶을 때 이 기능을 사용한다.

workspace에 저장하고, 스텝 별로 요약 정보를 볼 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-16.png" width="150%" height="150%"></center>

## Service Flow 다이어그램 보기

하나의 트랜잭션에 대한 흐름을 도식화하여 다이어그램으로 볼 수 있다.

- xtid: 현재 프로파일의 Flow Diagram
- gxid: 서버와 서버간의 연계 추적된 Diagram

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-17.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-18.png" width="150%" height="150%"></center>

## 메소드 상세 프로파일링 - 개발 환경에서 수행

### Advanced Option

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-19.png" width="150%" height="150%"></center>

---

# XLog 패턴

## 부하 테스트 사용자 추가하며 확인

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-20.png" width="150%" height="150%"></center>

## 타임 이벤트 발생

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-21.png" width="150%" height="150%"></center>

## 폭포수 현상 - 병목 패턴

동기화와 관련 

lock 매커니즘

update 문에서 주로 발생 

트랜잭션의 시작시간은 다른데, 종료시간이 같을 때 나타나는 패턴

트랜잭션들이 동일 자원에 대한 변경을 시도할 때 나타나는 패턴

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-xlog-22.png" width="150%" height="150%"></center>