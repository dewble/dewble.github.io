---
title: \[Scouter\]\(Monitoring\)Monitoring - common
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Scouter
- Monitoring
- Host-agent
- Java-agent
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Explain chart context menu and essential graphs list

# Chart context menu
<center><img src="/assets/images/posts/scouter/monitoring/monitoring-common-1.png" width="150%" height="150%"></center>


### All: 오브젝트별로 보기
하나의 성능 지표에 대해 복수개의 모니터링 대상의 성능값을 각각의 선으로 나타내는 방법

지표: 모든 지표

### Total: 합쳐보기

하나의 성능 지표에 대해 복수개의 모니터링 대상의 성능값을 합하여 나타내는 방법

지표: TPS, Recent User

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-common-2.png" width="150%" height="150%"></center>


### Current: 실시간 최근 5분 데이터를 계속 유지
### Today: 실시간으로 오늘의 데이터를 로드
### Past: 과거 특정 시점의 데이터를 로드
### Daily: 과거 데이터를 일 단위

--- 

# Essential graphs list

## How to add graphs

Collector → 데몬 → 항목선택

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-common-3.png" width="150%" height="150%"></center>

## WAS graphs list

- GC Count
- GC Time
- Heap Used or Heap Memory
- TPS
- Active Service EQ
- Active Speed
- XLog

## OS graphs list

- CPU
- Memory
- NETwork TX/RX Byte
- Swap → 성능이 아주 중요한 서비스의 경우 Swap 영역으로 메모리가 넘어가면 안되기 때문
