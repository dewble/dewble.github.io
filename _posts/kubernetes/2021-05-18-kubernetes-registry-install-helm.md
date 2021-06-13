---
title: \[Kubernetes\]\(Registry\)Install helm
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Registry
- Helm
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Install helm that set resources about kubernetes

---
# Install helm
```bash
root@ajtv005 [~/workspace/helm]curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
root@ajtv005 [~/workspace/helm]chmod 700 get_helm.sh
root@ajtv005 [~/workspace/helm]./get_helm.sh
Downloading https://get.helm.sh/helm-v3.4.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

root@ajtv005 [~/workspace/helm]helm version
version.BuildInfo{Version:"v3.4.2", GitCommit:"23dd3af5e19a02d4f4baa5b2f242645a1a3af629", GitTreeState:"clean", GoVersion:"go1.14.13"}
```

---
# Add helm stable repo

```bash
root@ajtv005 [~/workspace/helm]helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories

## add other repo
helm repo add argo https://argoproj.github.io/argo-helm
```

---
# Search helm list

## With repo

```bash
root@ajtv005 [~/workspace/helm]helm search repo stable
```

## With hub

```bash
root@ajtv005 [~/workspace/helm]helm search hub stable
```

---
# Helm repo update

```bash
helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harbor" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "argo" chart repository
...Successfully got an update from the "nicholaswilde" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```