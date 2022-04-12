---
title: \[Kubernetes\]\(Pod\)Static pods
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Pod
- Init Containers
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# 스태틱 파드 (Static pods)
kube-apiserver를 통하지 않고 특정 노드에 있는 kubelet이 직접 실행하는 파드

k8s에서 파드를 실행하려면 kube-apiserver 가 필요한데 kube-apiserver 자체를 처음 실행하는 별도의 수단으로 static pod를 이용한다.

kubelet은 각각의 static pod에 대하여 k8s API 서버에서 static pod를 추적하는 미러 파드를 생성하려고 시도하고 이를 통해 구동되는 static pod는 API 서버에서 볼 수 있지만, API 서버에서 제어될 수는 없다.

kubelet 설정의

```yaml
--pod-manifest-path 
```

옵션에 지정한 디렉터리에 static pod로 실행하려는 파드를 넣어두면 kubelet이 그걸 감지해서 파드로 실행한다.

### static pod 경로 확인

```bash
## find static pod definition files (--config) - config.yaml
$ ps -aux | grep kubelet |grep -i "config"
root      2771  3.3  4.8 1856928 98224 ?       Ssl  14:59   0:22 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2

## find static pod definition files (--config) - staticPodPath
 $ cat /var/lib/kubelet/config.yaml | grep -i "static"
staticPodPath: /etc/kubernetes/manifests

$ ls -alF /etc/kubernetes/manifests/
합계 16
drwxr-xr-x. 2 kube root   96  9월 14 05:57 ./
drwxr-xr-x. 4 kube root 4096  9월 14 06:03 ../
-rw-------. 1 root root 4011  9월 14 05:57 kube-apiserver.yaml
-rw-------. 1 root root 2990  9월 14 05:57 kube-controller-manager.yaml
-rw-------. 1 root root 1334  9월 14 05:57 kube-scheduler.yaml
```

---

> [https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/](https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/)
>