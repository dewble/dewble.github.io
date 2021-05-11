---
title: \[Kubernetes\]\(Monitoring\)Install EFK for kubernetes logging
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Logging
- Elasticsearch
- Fluentd
- Kibana
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
kubernetes logging

# 1. Install elasticsearch

```bash
helm repo add elastic https://helm.elastic.co
helm fetch elastic/elasticsearch
```

## Create pv first for elasticsearch

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efk-elasticsearch
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /k8s-nas
    server: 10.50.20.40
    #hostPath:
    #  path: /k8s-nas/efk
---
```

## Edit value.yaml

mapping volumeName to pv name

```bash
volumeClaimTemplate:
  #storageClassName: joins-nfs-storageclass
  accessModes: [ "ReadWriteOnce" ]
  resources:
    requests:
      storage: 20Gi
  volumeName: efk-elasticsearch
```

```bash
helm install elasticsearch . -n efk
```

### Check elasticsearch

```bash
root@jv0535 [~/workspace/yaml/efk/elasticsearch]curl 10.111.12.152:9200
{
  "name" : "elasticsearch-master-0",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "WnHSjspTRz61gJ7dVwWsLw",
  "version" : {
    "number" : "7.12.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "78722783c38caa25a70982b5b042074cde5d3b3a",
    "build_date" : "2021-03-18T06:17:15.410153305Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

# 2. Install Fluentd



```bash
helm fetch stable/fluentd-elasticsearch
helm install fluentd -n efk . --dry-run
```

## Edit value.yaml

```bash
elasticsearch:
  host: 'elasticsearch-master'
```

## Specify mount path ( HostPath )

Edit templates/daemonset.yaml

```bash
volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: libsystemddir
          mountPath: /host/lib
          readOnly: true
        - name: config-volume
          mountPath: /etc/fluent/config.d
```

### +) input tail usage

[tail usage link: docs.fluentd.org](https://docs.fluentd.org/input/tail)


# 3. Install Kibana

```bash
helm fetch elastic/kibana
```

## Edit value.yaml

```bash
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: kibana.ta.com
      paths:
        - path: /
```

```bash
helm install kibana . -n efk
```

# 4. Using EFK

## Define index pattern

Stack Management → Index Patterns 

<center><img src="/assets/images/posts/kubernetes/efk-define-index.png" width="150%" height="150%"></center>
```bash
logstash-*
```

## Search log

Analytics → Discover

<center><img src="/assets/images/posts/kubernetes/efk-discover.png" width="40%" height="40%"></center>  

<center><img src="/assets/images/posts/kubernetes/efk-discover2.png" width="150%" height="150%"></center>