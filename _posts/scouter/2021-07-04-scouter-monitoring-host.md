---
title: \[Scouter\]\(Monitoring\)Monitoring - Host object
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
categories:
- Scouter
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
monitoring real time resources provided by OS

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-1.png" width="150%" height="150%"></center>

### Scouter에는 모니터링 항목을 크게 두가지로 구분한다.

1. Performance Counter: 시간에 따라 변하는 값을 실시간 차트 형태로 보여준다.
2. Object Request: 사용자가 특정 성능 정보를 요청하여 조회하는 기능

### CPU, Memory, Net graphs
<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-0.png" width="150%" height="150%"></center>

---

# CPU monitoring

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-2.png" width="150%" height="150%"></center>

## 안정적인 시스템 운영을 위한 CPU 사용률 기준

CPU 사용률: 70% 이하

CPU Run Queue: CPU 코어당 3개 이하

---

# Memory monitoring

## 용어정의

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-3.png" width="150%" height="150%"></center>

## Performance Counter

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-4.png" width="150%" height="150%"></center>
---

# Network monitoring

### Socket state transition diagram

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-5.png" width="150%" height="150%"></center>
> [https://commons.wikimedia.org/wiki/File:Tcp_state_diagram_fixed_new.svg](https://commons.wikimedia.org/wiki/File:Tcp_state_diagram_fixed_new.svg)

## Performance Counter

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-6.png" width="150%" height="150%"></center>

---

# Object Request

실행되고 있는 HOST AGENT에 TCP 프로토콜로 명령을 수행하여 결과를 표현할 수 있다.

요청하는 순간의 값들을 보여준다. (새로고침 필요)

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-7.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-8.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-9.png" width="150%" height="150%"></center>

---

# Configure

Host Agent의 **설정 값**을 조회하고 변경할 수 있다.

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-10.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-11.png" width="150%" height="150%"></center>

오른쪽에 값을 더블클릭하면 config를 추가할 수 있다.  

경우에 따라 재시작이 요구된다.

# Properties

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-12.png" width="150%" height="150%"></center>

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-13.png" width="150%" height="150%"></center>

- main counter

아래의 Perf를 의미한다

<center><img src="/assets/images/posts/scouter/monitoring/monitoring-host-14.png" width="150%" height="150%"></center>