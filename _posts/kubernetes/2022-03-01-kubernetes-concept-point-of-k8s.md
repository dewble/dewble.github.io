---
title: \[Kubernetes\]\(Concept\)쿠버네티스 개념 - 지향점
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Point
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 지향점

1. 시스템 구축을 위한 수작업을 줄인다.
2. 시스템을 셀프 서비스로 운용한다.

# Immutable Infrastructure

한번 구축한 인프라는 변경할 필요 없이 파기하고 새로운 것을 구축해버리면 되므로 지금까지 부하가 컸던 인프라의 변경 이력을 관리할 필요가 없어졌다. 

그리고 인프라의 변경 이력을 관리하는 것이 아니라 지금 현재 움직이고 있는 인프라의 상태를 관리하면 되도록 바뀌어 왔다. 이러한 인프라를 Immutable Infrastructure 라고 한다.


# 선언적 설정

기존의 시스템 변경 이력을 기록하는것이 아니라 시스템의 상태를 관리하는 방식이다.

선언적 설정에서는 시스템이 본래 되어 있어야 할 모습을 정의. 클러스터 안에서 몇 개 가동시킬지, 최소로 필요로 하는 CPU나 메모리 리소스는 얼마 일지 등 시스템을 어떻게 구성하고 싶은지를 정의 파일에 적는다.

쿠버네티스 클러스터는 이 정의를 보고 그 모습이 되도록 자율적으로 움직인다.


# 자기 복구 기능

Immutable Infrastructure와 선언적 설정을 사용하여 시스템을 효율적으로 운용관리하는 구조를 취하고 있는데, 이러한 처리를 가능한 한 사람에 의한 수작업이 아니라 컴퓨팅 리소스가 자율적으로 운용할 수 있도록 소프트웨어로 실현하고 있다.

쿠버네티스가 어플리케이션의 상태를 항상 감시하며 이상이 있으면 미리 '시스템 본래의 모습'으로 설정된 상태가 되도록 쿠버네티스 자신이 자동으로 API를 사용하여 재시작시키거나 클러스터에서 장애를 제거하여 시스템을 복구 한다.