---
title: \[Kubernetes\]\(Backup\)Kubernetes ETCD backup and restore
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Backup
- ETCD
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Check ETCD **information**
### check ETCD version
```bash
kubectl describe pod etcd-controlplane -n kube-system
```

### At what address do you reach the ETCD cluster from you master node?

check listen urls(endpoint)

```bash
kubectl describe pod etcd-controlplane -n kube-system

## 중략
--listen-client-urls=https://127.0.0.1:2379,https://172.17.0.62:2379
```

### ETCD server certificate file located

find server cert

```bash
kubectl describe pod etcd-controlplane -n kube-system
## 중략

--cert-file=/etc/kubernetes/pki/etcd/server.crt
```

### ETCD CA Certificate file located

find ca cert

```bash
kubectl describe pod etcd-controlplane -n kube-system
## 중략

--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

### ETCD Key file located

find key file

```bash
kubectl describe pod etcd-controlplane -n kube-system
##  중략
--key-file=/etc/kubernetes/pki/etcd/server.key
```

---

# ETCD backup and restore

```bash
etcdctl snapshot save [filename]
```

## check etcd member first

- --endpoints
- --cacert
- --cert
- --key

```bash
ETCDCTL_API=3 etcdctl member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
```

## Backup ETCD

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
     
```

## Restore ETCD

```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd-from-backup
```