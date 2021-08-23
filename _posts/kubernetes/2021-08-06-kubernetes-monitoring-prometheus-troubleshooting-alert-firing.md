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
"KubeSchedulerDown"  
"KubeControllerManagerDown"  
"etcdInsufficientMembers"  
"etcdMembersDown  

---

# Cause

cannot connect node-exporter to these pods(kube-proxy,kubelet,etcd,kube-scheduler,kube-controller-manager) that running with node IP

---

# Solution

edit yaml file for opening pod IP and change health check url suitably

---

# etcd

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
    port: 2381 ## 변경
    targetPort: 2381 ## 변경
```

---

# kube-proxy

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

---

# kubelet

values.yaml

```bash
resource: true
    # From kubernetes 1.18, /metrics/resource/v1alpha1 renamed to /metrics/resource
    resourcePath: "/metrics/resource"  ## Edit path
```

---

# kube-controller-manager

/etc/kubernetes/manifests/kube-controller-manager.yaml

```bash
- --bind-address=0.0.0.0  # or [node ip] # orgin 127.0.0.1
```

values.yaml

```bash
kubeControllerManager:
  enabled: true

  ## If your kube controller manager is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints: []
  # - 10.141.4.22
  # - 10.141.4.23
  # - 10.141.4.24

  ## If using kubeControllerManager.endpoints only the port and targetPort are used
  ##
  service:
    port: 10257 #10252 # 변경
    targetPort: 10257 #10252 # 변경
    # selector:
    #   component: kube-controller-manager

  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""

    ## Enable scraping kube-controller-manager over https.
    ## Requires proper certs (not self-signed) and delegated authentication/authorization checks
    ##
    https: true #false # 변경

    # Skip TLS certificate validation when scraping
    insecureSkipVerify: true #null # 변경
```

- 위 values.yaml 변경해도 안될 경우 아래 방법으로 진행

k edit -n prometheus [servicemonitors.monitoring.coreos.com](http://servicemonitors.monitoring.coreos.com/) prometheus-prometheus-oper-kube-controller-manager

```bash
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    port: http-metrics
    scheme: https
    tlsConfig:
      caFile: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      insecureSkipVerify: true
  jobLabel: jobLabel
```

---

# kube-scheduler

kube-controller-manager 와 같은 방법으로 수정

/etc/kubernetes/manifests/kube-scheduler.yaml

```bash
- --bind-address=10.50.103.136 #127.0.0.1
```

values.yaml

```bash
kubeScheduler:
  enabled: true

  ## If your kube scheduler is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints: []
  # - 10.141.4.22
  # - 10.141.4.23
  # - 10.141.4.24

  ## If using kubeScheduler.endpoints only the port and targetPort are used
  ##
  service:
    port: 10259 #10251 #변경
    targetPort: 10259 #10251 #변경
    # selector:
    #   component: kube-scheduler

  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""
    ## Enable scraping kube-scheduler over https.
    ## Requires proper certs (not self-signed) and delegated authentication/authorization checks
    ##
    https: true  #false # 변경

    ## Skip TLS certificate validation when scraping
    insecureSkipVerify: true  #null #변경
```