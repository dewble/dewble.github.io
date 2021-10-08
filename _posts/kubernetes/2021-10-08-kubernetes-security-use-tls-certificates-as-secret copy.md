---
title: \[Kubernetes\]\(Command\)Check cluster IPs range
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Command
- IP
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Check cluster IPs range

# Services IP range

```bash
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
"--service-cluster-ip-range=10.96.0.0/12",
```

# Pods IP range

```bash
kubectl cluster-info dump | grep -m 1 cluster-cidr
"--cluster-cidr=10.254.0.0/16",
```