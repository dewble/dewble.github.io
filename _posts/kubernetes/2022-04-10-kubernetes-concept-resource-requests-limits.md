---
title: \[Kubernetes\]\(Concept\)Resource Management for Pods and Containers
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Resource
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
파드를 생성할때 컨테이너에 필요한 각 리소스 양을 선택적으로 지정할 수 있다. 

파드에서 컨테이너에 대한 ***리소스 요청(request)***을 설정하면, kube-scheduler는 이 정보를 사용하여 파드가 배치될 ***노드를 결정한다***. 그리고 kubelet은 컨테이너가 사용할 수 있도록 해당 시스템 리소스의 최소 요청량을 예약한다.

파드에서 컨테이너에 대한 ***리소스 제한(limit)***을 설정하면, kubelet은 실행 중인 컨테이너가 설정한 제한보다 많은 리소스를 사용할 수 없도록 해당 ***제한을 적용***한다.

limits 필드 값을 설정하지 않았다면 자원을 모두 사용할 수도 있다.

# 파드에 CPU와 메모리 자원 할당

## 최소 자원 요구량(requests)

.spec.containers.[].resources.requests.cpu

.spec.containers.[].resources.requests.memory

## 최대 자원 요구량(limits)

.spec.containers.[].resources.limits.cpu

.spec.containers.[].resources.limits.memory

pod-resource.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-simple-pod
  labels:
    app: kubernetes-simple-pod
spec:
  containers:
  - name: kubernetes-simple-pod
    image: nginx:latest
    resources:
      requests:
        cpu: 0.1
        memory: 200M
      limits:
        cpu: 0.5
        memory: 1G
    ports:
    - containerPort: 80
```

---

> [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
>