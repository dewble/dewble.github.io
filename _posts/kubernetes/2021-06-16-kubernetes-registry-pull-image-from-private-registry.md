---
title: \[Kubernetes\]\(Registry\)Pull an Image from a Private Registry
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
use private registry with docker-registry secret

---

# Create registry secret with command

```bash
kubectl -n [namespace] create secret docker-registry [secret-name] --docker-server=<your-registry-server> --docker-username='[your-name]' --docker-password='[your-pword]' --docker-email='[your-email]'
```

- `[your-registry-server]` 은 프라이빗 도커 저장소의 FQDN 주소이다. 도커허브(DockerHub)는 `https://index.docker.io/v2/` 를 사용한다.
- `[your-name]` 은 도커 사용자의 계정이다.
- `[your-pword]` 은 도커 사용자의 비밀번호이다.
- `[your-email]` 은 도커 사용자의 이메일 주소이다.

---

# Verify secret

namespace: springboot  
secret-name: join-harbor


```bash
k -n springboot get secrets join-harbor -o yaml
```

```bash
piVersion: v1
data:
  .dockerconfigjson: eyJhdXRo## 중략 ##
kind: Secret
metadata:
  creationTimestamp: "2021-05-31T04:58:24Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:.dockerconfigjson: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2021-05-31T04:58:24Z"
  name: join-harbor
  namespace: springboot
  resourceVersion: "13800590"
  selfLink: /api/v1/namespaces/springboot/secrets/join-harbor
  uid: 1c2aa797-814d-4bc9-99c3-c9f60b18a605
type: kubernetes.io/dockerconfigjson
```

dockerconfigjson → 도커 인증 정보 값

### check data

```bash
k -n springboot get secrets join-harbor -o jsonpath="{.data.\.dockerconfigjson}" | base64 -d
```

```bash
{"auths":{"harbor-ta.join.net":{"username":"admin","password":"password","auth":"YWRtaW"}}}
```

비밀번호가 auth 필드에 base64로 인코딩되어 저장된다.

---


# Create pod using secret

```bash
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: <docker-registry-secret>
```