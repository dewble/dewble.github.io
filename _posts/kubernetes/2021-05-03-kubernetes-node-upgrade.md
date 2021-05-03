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
# Purpose
Kubelet, Kubeadm version upgrade

---

# Work process

1. 기본 컨트롤 플레인 노드를 업그레이드한다.
2. 추가 컨트롤 플레인 노드를 업그레이드한다.
3. 워커(worker) 노드를 업그레이드한다.

### Check version

```bash
kubectl version --short
```

---

# Master node

## Decision version

```bash
kubeadm upgrade plan

## yum list 에서 확인
yum list --showduplicates kubeadm --disableexcludes=kubernetes
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

- Master Upgraded to v1.19.0
- Master Kubelet Upgraded to v1.19.0

```bash
apt update
apt install kubeadm=1.19.0-00
yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
[Y]

## 업그레이드 적용
kubeadm upgrade apply v1.19.0 
[Y]

master $ kubectl version --short
Client Version: v1.18.0
Server Version: v1.19.0

```

## Kubelet & Kubectl upgrade

```bash
apt install kubelet=1.19.0-00
yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
```

## Restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

## Uncordon

```bash
kubectl uncordon controlplane
```

---

# Worker node

## Work node drain

```bash
kubectl drain node01 --ignore-daemonsets
```

## Worker node upgrade

```bash
ssh node01
```

```bash
apt update
apt install kubeadm=1.19.0-00 
[y]
kubeadm upgrade node
apt install kubelet=1.19.0-00

or

yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
kubeadm upgrade node
yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
```

## Restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet

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