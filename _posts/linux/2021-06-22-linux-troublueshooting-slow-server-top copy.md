---
title: \[Linux\]\(Troubleshooting\)Slow servers - 3. Check for symptoms and troubleshoot overload with "sar" command
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
- sar
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose

Collect, report, or save system activity information.

현상 확인 후 과부하 문제 해결하기

---

# Install sysstat

성능 데이터를 기록할 수 있는 도구를 서버에 설치 - sysstat

```bash
yum install -y sysstat
```

---

# set sysstat config

sysstat이 활성화 되면 시스템 통계 정보를 10분마다 **/var/log/sa/** 에 기록한다.

## 1. vim /etc/sysconfig/sysstat

sysstat config (default)

```bash
# sysstat-10.1.5 configuration file.

# How long to keep log files (in days).
# If value is greater than 28, then log files are kept in
# multiple directories, one for each month.
HISTORY=28

# Compress (using gzip or bzip2) sa and sar files older than (in days):
COMPRESSAFTER=31

# Parameters for the system activity data collector (see sadc manual page)
# which are used for the generation of log files.
SADC_OPTIONS="-S DISK"

# Compression program to use.
ZIP="bzip2"
```

## 2. vim /etc/cron.d/sysstat (default)

```bash
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A
```

---

# Check CPU Statistics Information

```bash
sar  
```

기본적으로 당일의 CPU 통계를 보여준다

<center><img src="/assets/images/posts/linux/troubleshooting/sar/sar-1.png" width="150%" height="150%"></center>


- user :사용자모드에서 CPU가 소비된 시간의 비율
- nice: nice로 스케줄링의 우선도를 변경한 프로세스가 사용자 모드에서 CPU를 소비한 시간의 비율
- system: 시스템 모드에서 CPU가 소비된 시간의 비율
- iowait: CPU가 디스크 I/O 대기를 위해 Idle상태로 소비한 시간의 비율
- steal: Xen등 OS의 가상화를 이용하고 있을 경우 ㅡ 다른 가상 CPU의 계산으로 대기된 시간의 비율
- idle: CPU가 디스크I/O 대기등으로 대기되지 않고, Idle상태로 소비한 시간의 비율

---

# Check RAM Statistics Information

```bash
sar -r
```

<center><img src="/assets/images/posts/linux/troubleshooting/sar/sar-2.png" width="150%" height="150%"></center>

- kbmemfree : 물리 메모리 남은양
- kbmemused : 사용중인 물리 메모리
- kbbuffers : 커널내 버퍼로 사용되고 있는 물리 메모리량
- kbcached : 커널내 캐시용 메모리로 사용되고 있느 물리 메모리량
- kbswpfree : 스왑 영역의 남은 용량
- kbswpued : 사용중인 스왑 사용량

---

# Check DISK Statistics Information

```bash
sar -b 
```

<center><img src="/assets/images/posts/linux/troubleshooting/sar/sar-3.png" width="150%" height="150%"></center>

- tps: 초당 트랜잭션 수
- rtps: 읽기 트랜잭션
- wtps: 쓰기 트랜잭션
- bread/s: 초당 읽은 평균 바이트 수
- bwrtn/s: 초당 쓴 평균 바이트 수

```bash
sar -A 
```

모든 정보를 한번에 확인

---

# Get spcific time data

```bash
sar -br -s 09:00:00 -e 10:00:00
sar -s 09:00:00 -e 10:00:00
```

<center><img src="/assets/images/posts/linux/troubleshooting/sar/sar-4.png" width="150%" height="150%"></center>

---

# Get another day data

```bash
sar -f /var/log/sa/sa23 -br -s 09:00:00 -e 10:00:00
```

<center><img src="/assets/images/posts/linux/troubleshooting/sar/sar-5.png" width="150%" height="150%"></center>