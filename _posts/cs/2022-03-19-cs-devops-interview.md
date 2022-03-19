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
# 기술

기술 인터뷰의 경우 이력서에 작성한 내용이 주를 이뤘다. 그 외에 대한 내용을 아래에 정리

---

## DevOps

- 배포 전략에 대하여 설명하라
    - 각 배포방식에서 사용자한테 영향을 미치고 않고 배포는 효율적으로 할 수 있는 방법을 설명하라
- 젠킨스 파이프라인에서 배포를 위해 할 수 있는 설정들을 설명하라
- 넥서스 VS 하버의 차이를 설명하라

---

## Kubernetes

- 구축했던 클러스터 구성 설명하라
    - 워커 노드를 용도별로 구분 했을때 파드를 어떻게 배포할 수 있는지 설명하라
        - Taints와 Tolerations을 사용해서 설명하라
    - 각 플랫폼들의 선택 이유와 비교할 수 있는 플랫폼들과의 차이에 대한 설명 요구 → 플랫폼 선정에 대한 이유를 궁금해 하신듯
- kubeadm vs kubespray vs RKE 차이점을 설명하라
- ETCD를 구성할 수 있는 방법을 설명하라
- ETCD의 역할을 설명하라
- CRI, CNI 종류와 특징을 설명하라
    - Docker VS 그외 CRI의 차이점을 설명하라
    - Containerd에서 이미지 빌드, 푸쉬하는 과정을 설명하라
- K8s와 도커스웜의 차이

---

## Cloud

- 클라우드의 Paas 플랫폼의 장점 → HPC(High Perpormance Computing)에대한 설명을 원했던것 같음
- 클라우드에서 K8s를 운영할때 어떤 이점을 가질 수 있는가
- AWS 서비스 종류를 설명하라

---

## OS

- WAS 운영 할때 OS에서 어떤 튜닝을 할 수 있는지 설명하라 (e.g. 커널)
- 폐쇄망에서 프록시 서버를 사용하는 방법을 설명하라
- OS 관리를 위한 framework을 만들때 관리 항목들을 말하라

---

## Network

- 로드밸런싱 종류에 대하여 설명하라
    - HASH 알고리즘이 어떻게 작동하는지 사이트를 예를 들어 설명하라
- 스위치 종류와 역할에 대하여 자세히 설명하라

---

# 컬쳐

- 이직하려는 이유 → 꼬리의 꼬리를 무는 경우가 많다..
- 협업할 때 중요하게 생각하는것이 어떤 것인지? 상황을 예를 들어 설명하라
- 최근에 협업하며 들었던 긍정적인 내용, 부정적인 내용은 어떤 것이 있는가
- 회사에서 기술적으로 어떤 위치에 있는것 같은가
- 회사에서 신뢰받는 사람인것 같은가 그리고 그 이유는
- 야근에 대해서 어떻게 생각하는가 → ...ㅎ
- 커리어적인 목표를 얘기해보라 (e.g. 3년 후, 5년 후)
- 최근 읽은 기술 서적 3가지
- 최근 공부했던 내용을 IT를 모르는 사람에게 설명하라
- 지원했던 직무의 앞으로 비전이 어떤것 같은가