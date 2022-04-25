---
title: \[Kubernetes\]\(Concept\)What is Kubernetes Pod
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Pod
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
쿠버네티스는 파드라는 단위로 컨테이너를 묶어서 관리하므로 보통 컨테이너 하나가 아닌 여러 개 컨테이너로 구성된다. 

파드 안에는 컨테이너가(1개 이상) 있고 파드 안에 있는 컨테이너들은 IP 하나를 공유한다. 

외부에서 파드 안 컨테이너와 통신할 때는 컨테이너마다 다르게 설정한 포트를 사용한다. 

### tip. 불러온 pod 저장하기

```bash
kubectl run redis --image=redis --dry-run=client -o yaml > pod_redis.yaml
```

--dry-run=client -o yaml: 명령어를 실행시키지 않고 내용을 yaml 형태로 out한다.

---

# 파드 사용하기

pod-sample.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
# 파드 이름 설정
  name: nginx-pod
# 오브젝트를 식별하는 레이블
  labels:
    app: nginx-pod
spec:
  containers:
# 컨테이너의 이름 설정
  - name: nginx-pod
# 컨테이너에 사용할 이미지 설정
    image: nginx:latest
    ports:
# 컨테이너에 접속할 포트 설정
    - containerPort: 80
```

## pod apply

```bash
kubectl apply -f pod-sample.yaml
pod/kubernetes-simple-pod created

kubectl get pods | grep simple-pod
kubernetes-simple-pod                         1/1     Running   0          2m53s
```

---

# 파드 생명 주기

```bash
[dewble@instance-1 yaml]$ kubectl describe pods nginx-pod
Name:         nginx-pod
Namespace:    kube-system
Priority:     0
Node:         instance-5/10.146.0.11
Start Time:   Tue, 15 Sep 2020 13:34:16 +0000
Labels:       app=nginx-pod
Annotations:  <none>
Status:       Running
IP:           10.233.118.9
IPs:
  IP:  10.233.118.9

Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
```

### Status

- Pending → k8s에 파드 생성중
- Running → 파드 안 모든 컨테이너가 실행 중
- Succeeded → 파드 안 모든 컨테이너가 정상 실행 종료된 상태로 재시작되지 않는다.
- Failed → 파드 안 모든 컨테이너 중 정상적으로 실행 종료되지 않은 컨테이너가 있는 상태
- Unknown → 파드의 상태를 확인할 수 없는 상태

### Condtitions.Type

- Initialized  → 모든 초기화 컨테이너가 성공적으로 시작 완료
- Ready → 파드는 요청들을 실행할 수 있고 연결된 모든 서비스의 로드밸런싱 풀에 추가되어야 한다.
- ContainersReady → 파드 안 모든 컨테이너가 준비 상태
- PodScheduled → 파드가 하나의 노드로 스케줄을 완료
- Unschedulable → 스케줄러가 자원의 부족이나 다른 제약 등으로 지금 당장 파드를 스케줄 할 수 없다.