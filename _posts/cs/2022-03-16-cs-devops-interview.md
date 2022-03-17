---
title: \[CS\]\(Interview\)DevOps 면접 질문 정리
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- CS
- Concept
- Interview
- DevOps
- Kubernetes
categories:
- CS
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 받았던 질문
---
# DevOps
- 배포 전략에 대하여 설명하라
    - 각 배포방식이 사용자한테 영향을 미치고 않고 배포는 효율적으로 할 수 있는 방법을 설명하라
- 젠킨스 파이프라인에서 배포를 위해 할 수 있는 설정들을 설명하라

---
# Kubernetes
- 구축했던 클러스터 구성 설명하라
    - 워커 노드를 용도별로 구분 했을때 파드를 어떻게 배포할 수 있는지 설명하라
        - Taints와 Tolerations을 사용해서 설명하라
    - 각 플랫폼들의 선택 이유와 비교할 수 있는 플랫폼들과의 차이에 대한 설명 요구 → 플랫폼 선정에 대한 이유를 궁금해 하신듯
- ETCD를 구성할 수 있는 방법을 설명하라
- ETCD의 역할을 설명하라
- CRI, CNI 종류와 특징을 설명하라
    - Docker VS 그외 CRI의 차이점을 설명하라
    - Containerd에서 이미지 빌드, 푸쉬하는 과정을 설명하라
- EFK의 구성과 어떻게 활용할 수 있는지 설명하라

---
# Cloud
- 클라우드의 Paas 플랫폼의 장점 → HPC(High Perpormance Computing)에대한 설명을 원했던것 같음

---
# OS
- WAS 운영 할때 OS에서 어떤 튜닝을 할 수 있는지 설명하라 (e.g. 커널)
- 폐쇄망에서 프록시 서버를 사용하는 방법을 설명하라

---
# Network
- 로드밸런싱 종류에 대하여 설명하라
    - HASH 알고리즘이 어떻게 작동하는지 사이트를 예를 들어 설명하라
- 스위치 종류와 역할에 대하여 설명하라