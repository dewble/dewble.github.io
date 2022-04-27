---
title: \[Kubernetes\]\(Service\)How to use Kubernetes LoadBalancer
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Service
- LoadBalancer
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
<center><img src="/assets/images/posts/kubernetes/service/service-loadbalancer.png" width="150%" ></center>  
클라우드 공급자의 로드 밸런서를 사용하여 서비스를 외부에 노출시킨다. 외부 로드 밸런서가 라우팅는 NodePort와 ClusterIP 서비스가 자동으로 생성된다.

클라우드에서 제공하는 로드밸런서와 파드를 연결한 후 로드밸런서의 IP를 이용해 클러스터 외부에서 파드에 접근할 수 있도록 한다. 

EXTERNAL-IP 항목에 로드밸런서 IP를 표시한다. 이 IP를 사용해 클러스터 외부에서 파드에 접근한다.

---

# 실제 요청

service → kube-proxy → iptables 생성(실체는 iptables)

---

# Example - LoadBalancer

```bash
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: nginx-for-svc
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
$ kubectl get svc | grep service
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
clusterip-service           ClusterIP      10.233.54.164   <none>        80/TCP                   46h
loadbalancer-service        LoadBalancer   10.233.12.138   <pending>     80:31124/TCP             10m
nodeport-service            NodePort       10.233.39.47    <none>        80:30080/TCP             24h
```

이전과 달리 EXTERNAL-IP  항목이 설정되었다.

값이 pending으로 나타나는데 만약 외부 로드밸런서와 연계되어 쿠버네티스 클러스터가 설정된 상태라면 실제 외부에서 접근이 가능한 IP가 나타난다.

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
>