---
title: \[Kubernetes\]\(Monitoring\)Install Metrics server for Kubernetes monitoring
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Monitoring
- Metrics
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
kubernetes resource monitoring

아래와 같이 클러스터 리소스를 확인하기 위해서는 metrics server가 우선적으로 설치되어야 한다.
```bash
## check resource 
kubectl top node
kubectl top pod

root@jv0536 [~]kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
jv0535   132m         3%     3611Mi          46%
jv0536   120m         3%     2744Mi          35%
jv0537   121m         3%     2723Mi          35%
jv0538   553m         13%    4009Mi          51%
jv0539   125m         3%     3954Mi          51%
```

---
# Install metrics-server

```bash
root@AJTV005 [~/workspace]git clone https://github.com/kubernetes-sigs/metrics-server.git
```

# Edit config

## deployment

### 1. edit args

```bash
root@AJTV005 [~/workspace/metrics-server/manifests/base]vim deployment.yaml
## 중략

containers:
      - name: metrics-server
        image: gcr.io/k8s-staging-metrics-server/metrics-server:master
        imagePullPolicy: IfNotPresent
        args:
          **- --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP**
```

### 2. edit deployment (add selector)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  **labels:
    k8s-app: metric-server**
spec:
  **selector:
    matchLabels:
      k8s-app: metric-server**
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    **metadata:
      labels:
        k8s-app: metric-server** 
```

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
commonLabels:
  k8s-app: metrics-server
resources:
  - apiservice.yaml
  - deployment.yaml
  - rbac.yaml
  - service.yaml
```

# commit

```bash
kubectl apply -k .
```

---

# Use metrics-server API 

## expose metrics-server

```bash
kubectl port-forward svc/metrics-server -n kube-system 30443:443
```

## default 계정의 토큰 정보 얻기

```bash
$ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d ' ‘)
```

## 사용가능한 API 목록 확인

```bash
$ curl -k -H "Authorization: Bearer $TOKEN" https://localhost:30443/
{
  "paths": [
    "/apis",
    "/apis/metrics.k8s.io",
    "/apis/metrics.k8s.io/v1beta1",
    "/healthz",
    "/healthz/healthz",
    "/healthz/ping",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/metrics",
    "/openapi/v2",
    "/swagger-2.0.0.json",
    "/swagger-2.0.0.pb-v1",
    "/swagger-2.0.0.pb-v1.gz",
    "/swagger.json",
    "/swaggerapi",
    "/version"
  ]
}%
```

## metric 정보 확인 (/metrics)

```bash
curl -k -H "Authorization: Bearer $TOKEN" https://localhost:30443/metrics
……
# TYPE metrics_server_scraper_last_time_seconds gauge
metrics_server_scraper_last_time_seconds{source="kubelet_summary:docker-for-desktop"} 1.535815717e+09
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 8.92
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 11
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 2.2048768e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.53580689462e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 4.9332224e+07

```