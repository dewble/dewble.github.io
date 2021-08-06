---
title: \[Kubernetes\]\(Monitoring\)Prometheus - troubleshooting alert firing(etcd,kubelet,kube-proxy)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Monitoring
- Dashboard
- Prometheus
- Troubleshooting
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# problem
"TargetDown for kube-proxy"  
"TargetDown for kubelet"  
"TargetDown for etcd"  

---

# Cause

cannot connect node-exporter to these pods(kube-proxy,kubelet,etcd) that running with node IP

---

# Solution

edit yaml file for opening pod IP and change health check url suitably

## etcd

### etcd.yaml

-listen-metrics-urls 

```bash
- --listen-metrics-urls=http://0.0.0.0:2381
```

### values.yaml

```bash
kubeEtcd:
  enabled: true

  ## If your etcd is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints: []
  # - 10.141.4.22
  # - 10.141.4.23
  # - 10.141.4.24

  ## Etcd service. If using kubeEtcd.endpoints only the port and targetPort are used
  ##
  service: ##  Match port modified in --listen-metrics-urls
    port: 2381
    targetPort: 2381
```

## kube-proxy

### Edit kube-proxy config map

k edit cm kube-proxy -n kube-system

```bash
kind: KubeProxyConfiguration
    metricsBindAddress: "0.0.0.0:10249" ## 추가
    mode: ""
    nodePortAddresses: null
    oomScoreAdj: null
    portRange: ""
    showHiddenMetricsForVersion: ""
    udpIdleTimeout: 0s
    winkernel:
      enableDSR: false
      networkName: ""
      sourceVip: ""
```

Edit configmap and restart kube-proxy pod

## kubelet

values.yaml

```bash
resource: true
    # From kubernetes 1.18, /metrics/resource/v1alpha1 renamed to /metrics/resource
    resourcePath: "/metrics/resource"  ## Edit path
```