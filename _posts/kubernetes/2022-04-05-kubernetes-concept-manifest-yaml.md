---
title: \[Kubernetes\]\(Concept\)매니페스트 파일(yaml)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Manifest
- Yaml
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
쿠버네티스에서는 선언적 설정을 할 때 매니페스트 파일(yaml)을 사용한다.

---

# 매니페스트 파일의 기본

### 매니페스트 파일 예시

Nginx 가 움직이는 컨테이너 이미지를 바탕으로 한 웹 프론트 서버를 클러스터 안에서 10개 실행시켜 두고 싶은 경우

webserver.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver
spec:
  replicas: 10
  selector:
    matchLabels:
      app: webfront
  template:
    metadata:
      labels:
        app: webfront
    spec:
      containers:
      - image: nginx
        name: webfront-container
        ports:
          - containerPort: 80%
```

### 매니페스트 파일 리소스 등록

이 매니페스트 파일을 클러스터에 등록 ( API 에게 송신 )

```bash
kubectl apply -f webserver.yaml 
replicaset.apps/webserver created
```

클라이언트의 명령을 받은 쿠버네티스 마스터의 API Service는 파일의 내용을 클러스터의 구성 정보인 ***etcd에 저장한다***. 

쿠버네티스는 이 etcd에 기록된 정보를 바탕으로 리소스를 관리한다.

### 매니페스트 파일 리소스 삭제

매니페스트 파일로 정의한 리소르를 클러스터에서 삭제하려면 파일명을 -f 옵션으로 지정하여 다음 명령을 실행한다.

```bash
kubectl delete -f webserver.yaml      
replicaset.apps "webserver" deleted
```

---

# 매니페스트 파일 기본 구조

```yaml
apiVersion: [1. API의 버전 정보]
kind: [2. 리소스의 종류]
metadata:
  name: [3. 리소스의 이름]
spec:
[4. 리소스의 상세 정보]
```

### 1. API의 버전 정보

호출할 API 버전을 지정. 버전에 따라 안정성과 지원 레벨이 다르다.

- alpha
v1alpha1
- beta
v2beta3
- 안정판
v1 과 같은 번호가 붙는다

### 2. 리소스의 종류

쿠버네티스의 리소스 종류를 지정. 

- 어플리케이션의 실행 : Pod/RelpcaSet/Deployment
- 네트워크 관리 : Service/Ingress
- 어플리케이션 설정 정보의 관리  : ConfigMap/Secrets
- 배치 잡의 관리 : Job/CronJob

### 3. 리소스의 이름

리소스의 이름을 설정한다. kubectl 명령 등으로 조작할 때 식별에 사용하므로 짧고 알기 쉬운 이름이 좋다.

### 4. 리소스의 상세 정보

2. 에서 설정한 리소스의 종류에 따라 설정할 수 있는 값이 달라진다.

---

# YAML의 문법

### YMAL 3가지 특성

1. 인덴트로 데이터의 계층 구조를 나타낸다.
데이터의 계층 구조를 인덴트(들여쓰기)로 나타낸다. 인덴트에는 스페이스를 사용한다.
2. 종료 태그가 필요없다.
3. 플로 스타일과 블록 스타일
데이터의 구조를 표기할 때는 인덴트로 데이터 구조를 나타내는 스타일을 플로우 스타일
[]. {} 등으로 데이터 구조를 나타내는 스타일을 블록 스타일이라고 한다.
YAML에서는 어떤 표기 방법이든 상관없으며 둘을 섞어서 표기할 수 도 있다.

### 알아두면 좋은 YAML의 문법

1. 주석 행
'#' 이후는 주석
여러행 주석처리는 없기 때문에 각행에 주석 처리를 해야한다.
2. 데이터형
YAML은 다음과 같은 데이터형을 자동으로 판별한다.
정수, 부동소수점수, 진리값(true/false, yes/no) 
이 외의 것은 문자열로 취급한다. 홑따옴표/겹따옴표로 둘러싸면 강제적으로 문자열로 취급한다.
3. 배열
데이터 앞에 '-' 를 붙여서 배열을 나타낸다. '-' 뒤에는 반드시 스페이스를 넣어야 한다.
4. 해시
YAML에서는 해시를 나타낼때  key와 Value를 ':' 로 구분한다. ':' 다음에는 반드시 스페이스를 넣어야한다.
해시도 배열과 마찬가지로 인덴트를 사용하여 내포 구조를 나타낼 수 있다.

---

> **완벽한 IT 인프라 구축의 자동화를 위한 Kubernetes(쿠버네티스) -** [Asa Shiho](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956748412&orderClick=LAG&Kc=#)
 지음
[https://dewble.com/book/kubernetes-with-azure/](https://dewble.com/book/kubernetes-with-azure/)
> 