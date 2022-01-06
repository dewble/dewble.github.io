---
title: \[Nginx\]\(Config\)파일업로드 크기 제한 - client_max_body_size
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Nginx
- Config
categories:
- Nginx
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
대용량 파일 업로드가 필요한 경우 아래 옵션에서 size 설정이 필요하다.

# nginx.conf 에 내용 추가

## http

```bash
 http {
## 중략
   client_max_body_size 2000M;
 }
```

# server

```bash
 server {
## 중략
   client_max_body_size 2000M;
 }
```

## location

```bash
 location /uploads {
## 중략
 client_max_body_size 2000M;
 }
```