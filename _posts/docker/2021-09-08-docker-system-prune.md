---
title: \[Docker\]\(Command\)Docker system prune
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Docker
- Command
- Prune
categories:
- Docker
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Delete old data that is not used in docker 

---

# docker system prune

> [https://docs.docker.com/engine/reference/commandline/system_prune/](https://docs.docker.com/engine/reference/commandline/system_prune/)

Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.

### option

- -filter : Provide filter values
- f : Do not prompt for confirmation
- a : Remove all unused images not just dangling ones

until=120h : until (<timestamp>) - only remove containers, images, and networks created before given timestamp

---

# How to use

## crontab 등록

```bash
0 3 * * * /usr/bin/docker system prune --filter "until=120h" -fa 
```

위와 같이 crontab에 등록하여 주기적으로 사용하지 않는 docker와 관련된 데이터들을 삭제한다.