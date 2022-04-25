---
title: \[Kubernetes\]\(Troubleshooting\)Kubernetes Dynamic provisioning failure
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Troubleshooting
- Dynamic provisioning
- Volume
- PersistentVolumeClaim
- PersistentVolume
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Problem
Dynamic provisioning시 provisioner pod에서 아래와 같은 에러 메시지와 함께 PVC가 pending 상태가 된다.

“unexpected error getting claim reference: selfLink was empty, can't make reference”

k logs nfs-pod-provisioner-5948549f8c-46h4h

```bash
I0330 01:31:27.061484       1 controller.go:987] provision "default/test-dynamic-nfs-pvc" class "joins-nfs-storageclass": started
E0330 01:31:27.068286       1 controller.go:1004] provision "default/test-dynamic-nfs-pvc" class "joins-nfs-storageclass": unexpected error getting claim reference: selfLink was empty, can't make reference
```

# Solution

kube-apiserver에 아래 옵션을 추가 하고 kube-apiserver pod를 재시작 한다.

    - --feature-gates=RemoveSelfLink=false
추가

/etc/kubernetes/manifests/kube-apiserver.yaml

```bash
spec:
  containers:
  - command:
    - kube-apiserver
    - --feature-gates=RemoveSelfLink=false
    - --advertise-address=10.50.107.21
```

---

> Using Kubernetes v1.20.0, getting "unexpected error getting claim reference: selfLink was empty, can't make reference
> 
> 
> [Using Kubernetes v1.20.0, getting "unexpected error getting claim reference: selfLink was empty, can't make reference" · Issue #25 · kubernetes-sigs/nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25)
>