---
title: \[Kubernetes\]\(Node\)Drain and Uncordon for node upgrade
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Node
- Drain
- Cordon
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# **Purpose**
Kubelet, kubectl, kubeadm version upgrade

---

# Work process

1. 노드 업그레이드 순서 kubeadm upgrade → kubelet, kubectl upgrade
    1. 기본 컨트롤 플레인 노드를 업그레이드한다.
    2. 추가 컨트롤 플레인 노드를 업그레이드한다.
    3. 워커(worker) 노드를 업그레이드한다.

---

# Master node

# 1. kubeadm upgrade

### Check version

```bash
kubeadm version
```

## Decision version

```bash
## yum list 에서 확인
yum list --showduplicates kubeadm --disableexcludes=kubernetes

>> 1.21.4-0
```

## Master node drain

```bash
master $ kubectl drain controlplane --ignore-daemonsets

master $ kubectl get nodes
NAME           STATUS                     ROLES    AGE   VERSION
controlplane   Ready,SchedulingDisabled   master   17m   v1.18.0
node01         Ready                      <none>   17m   v1.18.0
```

## Master node upgrade && uncordon

```bash

## 위에서 확인한 버전 install
yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
yum install -y kubeadm-1.21.4-0 --disableexcludes=kubernetes
[Y]

## upgrade plan 확인
kubeadm upgrade plan

## 업그레이드 적용 (첫 번째 컨트롤플레인 노드에서 아래 명령 실행)
kubeadm upgrade apply v1.19.0 
kubeadm upgrade apply v1.21.4
## 첫번째 이후 다른 컨트롤 플레인 노드의 경우 아래명령 실행
kubeadm upgrade node 

## upgrade version 확인
kubeadm version
```

---

# 2. kubelet, kubectl upgrade

## Kubelet & Kubectl upgrade

```bash
## yum list 에서 확인
yum list --showduplicates kubelet kubectl --disableexcludes=kubernetes

>> 1.21.4-0

apt install kubelet=1.19.0-00
yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
yum install -y kubelet-1.21.4-0 kubectl-1.21.4-0 --disableexcludes=kubernetes
```

## Restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubelet --version
kubectl version --short
```

## Uncordon

```bash
kubectl uncordon controlplane
```

---

# Worker node

# 1. Kubeadm upgrade

## Work node drain

```bash
kubectl drain node01 --ignore-daemonsets
```

## Worker node kubeadm upgrade

```bash
ssh node01
```

```bash
## controlPlane node 와 버전을 맞춘다
## install kubeadm
yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
yum install -y kubeadm-1.21.4-0 --disableexcludes=kubernetes

## upgrade kubeadm
kubeadm upgrade node

## check version
kubeadm version
```

---

# 2. kubelet, kubectl upgrade

## Worker node kubelet, kubeadm upgrade

```bash
yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
yum install -y kubelet-1.21.4-0 kubectl-1.21.4-0 --disableexcludes=kubernetes
```

## Restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubelet --version

```

```bash
master $ kubectl get nodes
NAME           STATUS                     ROLES    AGE   VERSION
controlplane   Ready                      master   32m   v1.19.0
node01         Ready,SchedulingDisabled   <none>   31m   v1.19.0
```

## Uncordon

```bash
kubectl uncordon <node-to-drain>
```