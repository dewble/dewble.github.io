---
title: \[Kubernetes\]\(Registry\)Install harbor with helm chart
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Registry
- Harbor
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Install harbor to manage docker images  

---

# Install

## add repo

```python
helm repo add harbor https://helm.goharbor.io
helm repo list
```

## fetch & value edit

```python
helm fetch harbor/harbor
tar zxvf harbor-1.6.0.tgz
```

## create Namespace

```python
kubectl create ns harbor
```

---

# Edit helm chart

## Create TLS secret

Harbor를 도메인으로 접속, 사용하기 위해서는 HTTPS 접속이 필요하다.

테스트를 위한 인증서는 OpenSSL로 생성후 테스트를 진행한다.

```bash
## yaml 파일 생성
kubectl create secret tls tls-harbor --key cert.key --cert cert.crt --dry-run=client -o yaml > tls-yourdomain.yaml

## namespace 추가 해야 반영된다.
apiVersion: v1
data:
  tls.crt: ##
  tls.key: ##
kind: Secret
metadata:
  name: tls-domain.net
  namespace: harbor
type: kubernetes.io/tls

## seceret 확인
NAME                           TYPE                                  DATA   AGE
tls-domain.net                  kubernetes.io/tls                     2      4m35s
```

## PV,PVC 생성

### Create pv

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-registry
  namespace: harbor
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/registry
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-chartmuseum
  namespace: harbor
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/chartmuseum
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-jobservice
  namespace: harbor
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/jobservice
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-database
  namespace: harbor
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/database
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-redis
  namespace: harbor
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/redis
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-trivy
  namespace: harbor
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /k8s-nas/harbor/trivy
```

## Create PVC

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: chartmuseum
  name: harbor-chartmuseum
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Filesystem
  volumeName: harbor-chartmuseum
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: database
  name: harbor-database
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeMode: Filesystem
  volumeName: harbor-database
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: jobservice
  name: harbor-jobservice
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeMode: Filesystem
  volumeName: harbor-jobservice
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: registry
  name: harbor-registry
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Filesystem
  volumeName: harbor-registry
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: redis
  name: harbor-redis
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeMode: Filesystem
  volumeName: harbor-redis
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  labels:
    app: harbor
    component: trivy
  name: harbor-trivy
  namespace: harbor
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeMode: Filesystem
  volumeName: harbor-trivy
```

## Edit value.yaml

```bash
expose:
  type: ingress
  tls:
    enabled: true
    certSource: secret  #auto
    auto:
      commonName: ""
    secret:
      secretName: "tls-domain.net"
      notarySecretName: ""
---
persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      existingClaim: "harbor-registry"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 10Gi
    chartmuseum:
      existingClaim: "harbor-chartmuseum"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 10Gi
    jobservice:
      existingClaim: "harbor-jobservice"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
    database:
      existingClaim: "harbor-database"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
redis:
      existingClaim: "harbor-redis"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
    trivy:
      existingClaim: "harbor-trivy"
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi

---
harborAdminPassword: "admin"
---
ingress:
    hosts:
      core: core.domain.net
      notary: notary.doamin.net

---
externalURL: https://core.domain.net:30443

```

## +) When you need HA configuration, add replicas, pod affinity settings

```bash
replicas: 2
---
affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - chartmuseum
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - core
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - jobservice
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - notary-server
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - notary-signer
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - portal
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - registry
            topologyKey: kubernetes.io/hostname
---
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - trivy
            topologyKey: kubernetes.io/hostname
```

## Edit /etc/docker/daemon.json

```bash
# add
"storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "insecure-registries":["core.domain.net:30443"],
  "log-opts": { "max-size": "10m" }

```

# Push command example

### Docker image push

```bash
docker tag SOURCE_IMAGE[:TAG] core.harbor.domain:30774/infra/REPOSITORY[:TAG]
docker push core.harbor.domain:30774/infra/REPOSITORY[:TAG]

docker tag nginx:latest core.harbor.domain:30774/infra/nginx:v1
docker push core.harbor.domain:30774/infra/nginx:v1
docker pull core.harbor.domain:30774/infra/nginx:v1
```

### Helm chart push

```bash
helm chart save CHART_PATH core.harbor.domain:30774/infra/REPOSITORY[:TAG]
helm chart push core.harbor.domain:30774/infra/REPOSITORY[:TAG]
```