---
title: \[Linux\]\(Script\)Find and Delete old log files
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Linux
- Script
- Log
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Delete old log files

---
# Find and Delete old log files

## 1. 몇일 전

```bash
# 사용법 1
find -name '/경로/*.log' -mtime +(원하는날짜 -1) |xargs rm

# 사용법 2
find /home/searchuser/sf-1/log/*/*.log -type f -mtime +30 | xargs rm -f
find /home/searchuser/sf-1/log/*/*.log -type f -mtime +30 -exec rm -f {} \;
```

현재 위치 하위에서 .log 파일을 찾아서 mtime(수정시간)이 30일 이상 된 것을 지움(xargs rm)

## 2. 특정 월

```bash
## 대상 파일 확인
ll --time-style full-iso | awk '{print $6" "$9}' | grep 2021-04

## 대상 파일 삭제
ll --time-style full-iso | awk '{print $6" "$9}' | grep 2021-04 | awk '{print $2}' | xargs rm -f
```

## 3. script를 crontab 에 등록하여 삭제

### /box/log 밑에 있는 로그를 1주일 간격으로 삭제

```bash
0 0 * * * (find /box/log/*/*.log.* -mtime +7 -exec rm -rf {} \;)
30 0 * * * (find /box/log/*/*.log.* ! -name '*.gz' -mtime +1 -exec gzip {} \;)
```

### 매일 4시, 14일 지난 로그 삭제 (script 실행)

```bash
00 04 * * * /root/scheduled_job/log_delete.sh
```

log_delete.sh

```bash
#!/bin/sh

LOG=/box/java_logs

find $LOG -type f -mtime +14 |xargs rm -f
```