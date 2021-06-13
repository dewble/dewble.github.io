---
title: \[Kubernetes\]\(Registry\)Understand how to use helm chart
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
- Howto
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
To understand How to use helm chart

---
# Helm chart directory structure
```bash
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

---
# Find helm chart - helm search

```bash
helm search repo/hub [chartname]
```

---
# Download and install helm chart - helm fetch and install

**Before install helm chart, you can change values.yaml file what you want**

```bash
helm fetch [helmchart]
helm install [helmchart] .[chart-path] -n [namespace]
helm uninstall [helmchart] -n [namespace]
```
### Option

- template debug
    1. lint : verify template and dependencies
    2. --dry-run : you can see template yaml files and error

```bash
1. helm lint prometheus-operator
2. helm install prometheus-operator
```

---
# Helm upgrade - After edit value.yaml

```bash
helm upgrade -n prometheus prometheus -f ./values.yaml ./ --dry-run

helm history -n prometheus prometheus
REVISION        UPDATED                         STATUS          CHART                           APP VERSION     DESCRIPTION
1               Fri Apr  2 13:17:14 2021        superseded      prometheus-operator-9.3.2       0.38.1          Install complete
2               Fri Apr  2 13:24:14 2021        deployed        prometheus-operator-9.3.2       0.38.1          Upgrade complete
```

---
# Do not remove resource such as pvc

```go
metadata:
  annotations:
    "helm.sh/resource-policy": keep
[...]
```