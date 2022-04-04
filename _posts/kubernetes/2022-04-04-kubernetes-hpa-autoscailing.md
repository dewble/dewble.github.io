---
title: \[Kubernetes\]\(HPA\)What is Horizontal Pod Autoscaler (HPA)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- HPA
- What is
- Autoscaling
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
쿠버네티스에서, *HorizontalPodAutoscaler*는 워크로드 리소스(예: [디플로이먼트](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)
 또는 [스테이트풀셋](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/))를 자동으로 업데이트하며, 워크로드의 크기를 수요에 맞게 자동으로 스케일링하는 것을 목표로 한다.

→ 부하 증가에 대해 [파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)를 더 배치하는 것을 뜻한다.

부하량이 줄어들고, 파드의 수가 최소 설정값 이상인 경우 HPA는 스케일 다운을 지시한다.

---

# 알고리즘

```bash
원하는 레플리카 수 = ceil[현재 레플리카 수 * ( 현재 메트릭 값 / 원하는 메트릭 값 )]
```

### 예시 - 1

현재 메트릭 값 200m, 원하는 값 100m인 경우 200/100 = 2

레플리카 수는 2배가 된다.

### 예시 - 2

현재 메트릭 값 50m,  원하는 값 100m인 경우 50/100 = 0.5

레플리카 수는 1/2배가 된다.

---

# yaml 파일 작성

## autoscaling.yaml

```bash
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50 ## pod cpu average
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 1.5Gi ## pod memory average
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 120
```

### hpa describe

```bash
Name:                                                  hpa
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 26 Jul 2021 14:57:05 +0900
Reference:                                             Deployment/hpa
Metrics:                                               ( current / target )
  resource memory on pods:                             816898048 / 1536Mi
  resource cpu on pods  (as a percentage of request):  4% (4m) / 50%
Min replicas:                                          2
Max replicas:                                          5
Behavior:
  Scale Up:
    Stabilization Window: 0 seconds
    Select Policy: Max
    Policies:
      - Type: Pods     Value: 4    Period: 15 seconds
      - Type: Percent  Value: 100  Period: 15 seconds
  Scale Down:
    Stabilization Window: 120 seconds
    Select Policy: Max
    Policies:
      - Type: Percent  Value: 100  Period: 15 seconds
Deployment pods:       2 current / 2 desired
```

### Scale up

HPA가 정상 상태에 도달 할 때까지 15초 마다 4개의 파드 또는 현재 실행 중인 레플리카의 100%가 추가된다.

### Scale Down

```bash
  Scale Down:
    Stabilization Window: 120 seconds
    Select Policy: Max
    Policies:
      - Type: Percent  Value: 100  Period: 15 seconds
```

Down sacle 안정화 윈도우를 2분동안 제공

```bash
behavior:
  scaleDown:
    policies:
    - type: Percent
      value: 10
      periodSeconds: 60
```

HPA에 의해 파드가 제거되는 속도를 분당 10%로 제한

---

# 부하 테스트

60초 동안 100명의 사용자가 50번 접속 시도

<center><img src="/assets/images/posts/kubernetes/hpa/hpa1.png" width="150%" ></center> 

<center><img src="/assets/images/posts/kubernetes/hpa/hpa2.png" width="150%" ></center> 


### resources monitoring by prometheus

<center><img src="/assets/images/posts/kubernetes/hpa/hpa3.png" width="150%" ></center> 


### check hpa data

kubectl get hpa

```bash
root@jv0535 [~]k get hpa
NAME   REFERENCE        TARGETS                      MINPODS   MAXPODS   REPLICAS   AGE
hpa    Deployment/hpa   1004662784/1536Mi, 72%/50%   2         5         5          126m
```

kubectl describe hap

```bash
Events:
  Type    Reason             Age                  From                       Message
  ----    ------             ----                 ----                       -------
  Normal  SuccessfulRescale  20m                  horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  15m                  horizontal-pod-autoscaler  New size: 4; reason: All metrics below target
  Normal  SuccessfulRescale  14m                  horizontal-pod-autoscaler  New size: 3; reason: All metrics below target
  Normal  SuccessfulRescale  9m42s                horizontal-pod-autoscaler  New size: 3; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  5m56s (x3 over 71m)  horizontal-pod-autoscaler  New size: 2; reason: All metrics below target
```

---

> [https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/)
>