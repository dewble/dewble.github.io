---
title: \[Kubernetes\]\(Monitoring\)Install Kubernetes WEB UI Dashboard
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
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Monitoring kubernetes and controll kubernetes using WEB UI

---
# Install WEB UI Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

## pod 확인

```bash
root@AJTV005 [~]kubectl get pod -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
## 중략

kubernetes-dashboard   dashboard-metrics-scraper-7b59f7d4df-q9pl8   1/1     Running   0          3m27s
kubernetes-dashboard   kubernetes-dashboard-74d688b6bc-zprkk        1/1     Running   0          3m28s
```

## Check cluster info

```bash
root@AJTV005 [~]kubectl cluster-info
Kubernetes master is running at https://10.50.107.23:8443
KubeDNS is running at https://10.50.107.23:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

---
# Edit service ( you can change yaml as well )

## change svc type to nodePort

```bash
root@AJTV005 [~/workspace]kubectl get svc -A
NAMESPACE              NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
# 중략
kube-system            metrics-server              ClusterIP   10.104.203.75   <none>        443/TCP                  2d23h
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.104.59.152   <none>        8000/TCP                 6m43s
kubernetes-dashboard   kubernetes-dashboard        ClusterIP   10.109.20.224   <none>        443/TCP                  6m44s
```

```bash
root@AJTV005 [~/workspace]kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

type: ClusterIP --> type: NodePort
```

## Check for svc type change

```bash
root@AJTV005 [~/workspace]kubectl get svc -A -o wide
NAMESPACE              NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
default                kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                  6d20h   <none>
kube-system            kube-dns                    ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   6d20h   k8s-app=kube-dns
kube-system            metrics-server              ClusterIP   10.104.203.75   <none>        443/TCP                  3d      k8s-app=metrics-server
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.104.59.152   <none>        8000/TCP                 13m     k8s-app=dashboard-metrics-scraper
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.109.20.224   <none>        443:30225/TCP            13m     k8s-app=kubernetes-dashboard
```

## Use nodeport to https access on WEB UI dashboard

```bash
https://10.50.107.23:30225/#/login
```

---
# 3 Login methods

1. Use kubeconfig 

2. Use Token

3. Login without any auth

<center><img src="/assets/images/posts/kubernetes/monitoring/monitoring-web-ui-dashboard-login.png" width="150%" height="150%"></center>


## 1. Use kubeconfig

upload your kubeconfig file

It is located in $HOME/.kube

## 2. Use token

### 2.1 Create svc account

```bash
cat << EOF > sa-admin-user.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
EOF
```

### 2.2 Create ClusterRoleBinding

```bash
cat << EOF > rbac-admin-user.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```

```bash
root@AJTV005 [~/workspace/account]kubectl apply -f .
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
serviceaccount/admin-user created
```

### 2.3 Get Token

```bash
cat << EOF > token-info
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user| awk '{print $1}')
EOF
```

```bash
root@AJTV005 [~/workspace/account]kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user| awk '{print $1}')
Name:         admin-user-token-gjp22
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 2d318872-fcfa-4243-8945-7bd3297e2b96

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkhIMk1peHVUSDFZX0JydXZPaTFqdXBxcFJkLW03V09WZHpBUjVQTXZGVjgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWdqcDIyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyZDMxODg3Mi1mY2ZhLTQyNDMtODk0NS03YmQzMjk3ZTJiOTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.p5MOK8ErB9SRw9rL8DYNbGp7Pt3AZes41Onuk8ZJZVkoHEihfNu3R6eh5feKCtX6aMDrDTSl8rb5nZ-3PpNt56DHkBjrjzn3sAJjrRMPCa6KVemscUWgGB3PbwODZAliI8IlpBcZFwOjyBzsuj-W2_7FN2g4LtW3PTNJE8rcRSpA5xAoLsZx9DJZlYc9S-b08yP5UWPpwvsZuZEfOxJGlvCBXdoCmx72NbAYoIIuB_-lD1mElSZpSMq4zgaiODqyiPoa-GOZjBznwH8PV8I9ZgMmsKyhtXhuvNYhDli8hbcNYxZ0N0K0Te_zFr0DvC1E90gaS1qjq1xz7OslAhUOEw
ca.crt:     1066 bytes
namespace:  11 bytes
```

## 3. Login without any auth

### Chnage deployment spec.args

```go
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
        ports:
        - containerPort: 9090 #2
          protocol: TCP
        args:
        # - --auto-generate-certificates
        - --enable-insecure-login #1 인증토큰 없이 로그인
        - --insecure-bind-address=0.0.0.0 #2 http 접속 가능
        - --insecure-port=9090 #2 http 접속 가능
        - --enable-skip-login #1 인증토큰 없이 로그인
        # Uncomment the following line to manually specify Kubernetes API server Host
        # If not specified, Dashboard will attempt to auto discover the API server and connect
        # to it. Uncomment only if the default does not work.
        # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTP
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
```

---
# Use WEB UI dashboard
<center><img src="/assets/images/posts/kubernetes/monitoring/monitoring-web-ui-dashboard-example.png" width="150%" height="150%"></center>
