---
title: \[Theory\]What is GitOps?
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Theory
- Gitops
categories:
- Theory
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 깃옵스란?
애플리케이션의 배포와 운영에 관련된 모든 요소를 코드화하여 깃(Git)에서 관리(Ops)하는 것이 깃옵스의 핵심이다. 

기본 개념은 코드를 이용하여 인프라를 프로비저닝 하고 관리하는 IaC(Infrastructure as Code)에서 나온 것으로 깃옵스는 이를 인프라에서 전체 애플리케이션 범위로 확장하였다.

---

# 핵심 개념

1. 배포에 관련된 모든 것을 선언형 기술서(Declarative Descriptions) 형태(yaml) 로 작성하여 Config Repository에서 관리한다. → Git
2. Config Repository의 선언형 기술서와 운영 환경 간 상태 차이가 없도록 유지시켜주는 자동화 시스템을 구성한다. → ArgoCD

---

# 아키텍처

<center><img src="/assets/images/posts/theory/gitops/gitops-1.png" width="150%" height="150%" ></center>  

### code 저장소: **어플리케이션 자체의 소스 코드를 저장.**

어플리케이션 소스코드와 빌드를 위한 Dockerfile, Jenkinsfile 관리

→ 소스코드에 gitops 폴더를 복사하여 프로젝트 구성

<center><img src="/assets/images/posts/theory/gitops/gitops-2.png" width="150%" height="150%" ></center>  

### config 저장소: **Kubernetes 배포용 매니페스트 저장.**

배포를 위한 yaml 파일 관리, Helm chart 관리

<center><img src="/assets/images/posts/theory/gitops/gitops-3.png" width="150%" height="150%" ></center>  

---

# 장점

1. 인프라를 포함한 애플리케이션 배포와 운영에 관련된 모든 것을 코드로 관리할 수 있고, 이 코드를 이용하여 언제든 똑같은 환경을 다시 만들어내거나 부분 소실 시 복원할 수 있다는 점입니다.
2. Git을 단일 진실 공급원으로 지정하고 오직 이 곳에서만 관리하도록 한다.  모든 운영 활동의 시작은 깃이므로 사람 혹은 시스템 간의 혼선을 최소화 할 수 있다.  
3. Git 에서  누릴 수 있었던 깃의 기능을 운영 단계에서도 활용할 수 있게 된다. 

    → (History, Commit, Merge Request/ Review, Revert 등)

> [https://www.s-core.co.kr/insight/view/데브옵스의-확장-모델-깃옵스gitops-이해하기-2/](https://www.s-core.co.kr/insight/view/%EB%8D%B0%EB%B8%8C%EC%98%B5%EC%8A%A4%EC%9D%98-%ED%99%95%EC%9E%A5-%EB%AA%A8%EB%8D%B8-%EA%B9%83%EC%98%B5%EC%8A%A4gitops-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-2/)