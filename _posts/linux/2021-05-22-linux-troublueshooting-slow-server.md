---
title: \[Linux\]\(Troubleshooting\)Slow servers - 1. Check load average
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Linux
- Troubleshooting
- Slow servers
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Check load average 1, 5, 15 minutes. It will be fundamental data to resolve problem that slow servers

# Check load average - 시스템 부하 확인
시스템의 평균 부하는 느려진 시스템의 문제해결을 시작할 때 기초적인 지표가 될 것이다.
```bash
[root@jv0472 ~]# uptime
 17:42:20 up 329 days,  2:48,  1 user,  load average: 0.23, 0.21, 0.27
```
평균 부하(load average) 다음에 나오는 세 개의 숫자는 시스템에서 1분, 5분, 15분 동안의 평균 부하를 각각 나타 낸다.  

평균 부하는 **실행 가능한 상태** 혹은 **중단 불가능한 상태**의 프로세스들의 평균 개수와 같다.

- 실행 가능한 프로세스: 현재 CPU를 사용하고 있거나 CPU 사용을 위해 대기 중인 상태
- 중단 불가능한 프로세스: I/O를 기다리고 있는 프로세스.

### Example
1) 단일 CPU 환경의 시스템에서 평균 부하 1
   -  한 개의 CPU가 끊임없이 일을 하고 있었다는 뜻이다.

2) 단일 CPU 환경의 시스템에서 평균 부하가 4 
   -  처리할 수 있는 부하보다 4배의 부하가 시스템에 걸려 있다는 것.  
   - 4개의 프로세스 중 3개의 프로세스는 CPU 자원을 사용하기 위해 대기 중이라는 뜻이다.  
  


# What information can get from the average load?
중요한 것은 load average 를 확인 하여 우선적으로

1. 부하가 CPU에 집중된 것인지(프로세스가 CPU 자원을 대기하고 있는 것) 
2. RAM에 집중된 것인지(특히, 높은 RAM 사용율로 인해 프로세스가 RAM 영역이 아닌 스왑영역으로 옮겨졌을 경우)
3. I/O에 집중된 것인지  
   
파악 하는 것이다.  

만약 load average가 높은데 RAM 사용률이 낮다면 CPU 업그레이드를 고려해 볼 수 있다.