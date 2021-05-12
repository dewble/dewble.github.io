---
title: \[Linux\]\(Command\)scp — secure copy (remote file copy program)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Linux
- Command
- SCP
categories:
- Linux
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
scp copies files between hosts on a network.

# Option
-p : 원본 권한 속성 유지
-P : ssh port num
-c : 압축 복사
-v : 과정 출력 복사
-a : 아카이브 모드 복사
-r : 디렉토리 내 모든 파일 or 디렉토리 복사

# 1. 원격지로 파일 전송

write 권한이 있어야 전송가능

원격지에서 받고 나면 목적지에서는 rw-rw-r-- 와 같은 권한을 가진다.

# 기본 문법

scp [option] [소스디렉터리] [원격지ID]@[원격지IP]:[원격지디렉터리]

-P: ssh 연결 포트

-r : 하위 디렉터리까지 전송

## 단일 파일 전송

```bash
scp -P 2211 /home/asmanager/scpfile1  asmanager@10.50.100.24:/home/asmanager/flow/schedule/
```

## 다중 파일 전송

```bash
scp -P 2211 scpfile1 scpfile2 asmanager@10.50.106.25:/DATA/
```

## 디렉터리 전송

```bash
scp -P 2211 -r /DATA/diractory [원격지ID]@[원격지IP]:/[원격지디렉터리]
```

# 2. 원격지에서 파일 가져오기

# 기본 문법

scp [option] [원격지ID]@[원격지IP]:[원본파일] [목적지위치]

-P: ssh 연결 포트

-r : 하위 디렉터리까지 전송

## 단일 파일 전송

```bash

scp -P 2211 logmanager@10.50.106.138:/home/logmanager/scp-source-test/testfile1 /data/scp-destination-test/
testfile1                                                                        100%    0     0.0KB/s   00:00
[asmanager@jv0540 scp-destination-test]$ ll
total 0
-rw-rw-r--. 1 asmanager asmanager 0 May 11 11:08 testfile1
[asmanager@jv0540 scp-destination-test]$
```

## 복수 파일 전송

원본파일을 " "를 사용해서 묶어준다

```bash
scp -P 2211 [원격지ID]@[원격지IP]:"[원본파일1] [원본파일2]" [목적지위치]
```

## 디렉터리 전송

```bash
scp -P 2211 -r [원격지ID]@[원격지IP]:[디렉터리위치] [목적지위치]
```