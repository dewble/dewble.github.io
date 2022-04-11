---
title: \[Kubernetes\]\(Pod\)Init Containers - 초기화 컨테이너
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Pod
- Init Containers
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 초기화 컨테이너 (Init Containers)
앱 컨테이너가 실행되기 전 파드를 초기화한다. 보안상 이유로 앱 컨테이너 이미지와 같이 두면 안 되는 앱의 소스 코드를 별도로 관리할 때 유용하다.

## 특징

- 초기화 컨테이너가 여러 개 있다면 파드 템플릿에 명시한 순서대로 초기화 컨테이너가 실행된다.
- 초기화 컨테이너 실행이 실패하면 성공할 때까지 재시작한다. 즉, 필요한 명령들을 순서대로 실행하는 데 사용된다.
- 초기화 컨테이너가 모두 실행된 후 앱 컨테이너 실행이 시작 된다. 즉, 초기화 컨테이너가 외부의 특정 조건을 만족할 때까지 대기하고 있다가 조건이 충족된 후 앱 컨테이너를 실행한다.
- 앱 컨테이너와 비슷하지만, 파드가 모두 준비되기 전에 실행한 후 종료되는 컨테이너이기 때문에  readindessProbe 를 지원하지 않는다.

pod-init.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: k8s-app-pod
  labels:
    app: k8s-app-pod
spec:
  initContainers:
# 첫 번째 초기화 컨테이너
  - name: init-myservice
    image: busybox:latest
# command 필드 값으로 해당 컨테이너를 실행할 때 2초 대기한 후 echo 명령을 실행
    command: ['sh', '-c', 'sleep 2; echo init first container;']
# 두 번째
  - name: init-mydb
    image: busybox:latest
    command: ['sh', '-c', 'sleep 2; echo init second container;']
# 세 번째
  containers:
  - name: k8s-app-pod
    image: busybox:latest
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

위 pod 를 생성하기 전 초기화 컨테이너로 init-myservcie와 init-mydb를 실행한다.

---

> [https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)
>