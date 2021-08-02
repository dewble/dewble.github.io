---
title: \[Scouter\]\(Monitoring\)Monitoring - WAS (Object request)
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
사용자가 필요한 시점에 요청을 보내 데이터를 얻을 수 있다.

---

# WAS Object Request

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-1.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-2.png" width="150%" height="150%"></center>

## Active Service List

WAS에서 현재 수행되고 있는 Active Service List를 조회

Active Service EQ의 Stack Bar를 클릭하여도 해당 기능 수행

## Load Class List

WAS에 로딩된 모든 class(interface 포함) 리스트와 메타정보 조회

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-3.png" width="150%" height="150%"></center>

- Copy: Clipboard 로 선택한 line 복사
- Export Class: 선택한 class 를 클라이언트 pc로 export (download)
- Export Jar: 선택한 class를 포함하고 있는 jar를 pc로 export (download)
- Description: 선택한 class의 변수와 메소드 목록을 보여줌 (더블클릭 가능)
- Redefine class: 선택한 class를 reload

## Thread Dump

WAS의 현시점의 Thread Dump를 출력

재시작전에 반드시 저장해두고 이후에 분석

```bash
## command 사용
jstack $pid 또는 kill -3 $pid
```

## Socket

현재 WAS에서 오픈한 Socket에 대한 정보를 조회할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-4.png" width="150%" height="150%"></center>

## System.GC

WAS에 강제로 Garbage Collection을 수행한다.

GC가 수행되는 동안 서비스중단이 발생할 수 있으니  주의 필요

## Heap Dump

hprof 형식의 Heap Dump를 생성하고, 다운로드 할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-5.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-6.png" width="150%" height="150%"></center>

System.GC 수행 하고 Heap Dump를 수행하는것이 좋다.

```bash
## command  사용
jmap -dump:live,format=b,file=${fileName}.hprof $pid
```

## File Dump

Dump 파일을 생성하고, 조회한 후 로컬에 저장할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-7.png" width="150%" height="150%"></center>

- List Dump Files: Dump 된 파일리스트를 보여주고 더블클릭하면 해당 내용을 보여준다
- Dump Active Service List : Active Service List Dump 파일 생성
- Dump Thread Dump : Thread Dump 파일 생성
- Dump Thread List : Thread List 파일 생성
- Dump Heaphisto : Heap Histogram 파일 생

---

# WAS Summary -오브젝트 분석 및 통계

## Stack Frequency Analyzer

주기적인 쓰레드 덤프를 트리거 시켜 다수의 쓰레드 덤프를 만들고 분석하여 가장 빈번히 나타나는 쓰레드 스택을 확률적으로 찾아 감으로써 자원을 차지하는 시간이 상대적으로 긴 워킹 스레드의 업무를 추적하는 분석 기법

## Summary

통계 범위에 따라 아래 3가지 메뉴가 제공된다.

1. 전체 통계
2. 같은 WAS 타입
3. 하나의 WAS서버

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-8.png" width="150%" height="150%"></center>
## Kinds of Summary

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-was-object-request-9.png" width="150%" height="150%"></center>

### 통계를 활용하여 아래 순으로 성능 튜닝을 진행한다.

1. 서비스 top10 - Total Elapsed
    1. 서비스 URL별 통계를 조회
    2.  Total Elapsed를 통해 Top Application에 대해 성능을 판단
2. SQL top10 
    1. SQL별 통계를 조회
    2. Top SQL을 추출하고 성능을 판단
3. Exception top10
    1. 예외에 대한 통계를 조회
    2. 시스템 품질에 대한 척도로 활용 가능