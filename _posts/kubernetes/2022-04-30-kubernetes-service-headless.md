---
title: \[Kubernetes\]\(Service\)How to use Kubernetes Headless service
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Service
- Headless
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
.spec.clusterIP: None 으로 설정하면 클러스터 IP가 없는 서비스를 만들 수 있다.
이런 서비스를 'headless' 서비스라고 한다.

로드 밸런싱이 필요 없거나 단일 서비스 IP가 필요 없을 때 사용한다.

- .spec.selector 를 설정하면 쿠버네티스 API로 확인할 수 있는 엔드포인트가 만들어진다.
    - selector가 없으면 Endpoints가 만들어지지 않는다.
- 서비스와 연결된 파드를 직접 가리키는 DNS A 레코드도 만들어 진다.
- 단, selector가 없더라도 DNS 시스템은 ExternalName 타입의 서비스에 사용할 CNAME레코드가 만들어진다.

---

# Example - headless

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: nginx-for-svc
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

`.spec.clusterIP`: None

### 서비스 확인 

```bash
$ kubectl get svc
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
headless-service            ClusterIP      None            <none>        80/TCP                   17s
```

- CLUSTER-IP: None
- EXTERNAL-IP: <none>

### Endpoints 항목 확인

```yaml
$ kubectl describe svc headless-service
Name:              headless-service
Namespace:         kube-system
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-for-svc
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.233.118.224:80,10.233.125.121:80
Session Affinity:  None
Events:            <none>
```

- `Endpoints`: 10.233.118.224:80,10.233.125.121:80

CLUSTER-IP는 None 이지만 , Endpoints 항목에는 .spec.selector 필드에서 선택한 조건에 맞는 파드들의 IP와 포트 정보를 확인 할 수 있다.

### headless 서비스의 도메인

headless-service.default.svc.cluster.local

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/#헤드리스-headless-서비스](https://kubernetes.io/ko/docs/concepts/services-networking/service/#%ED%97%A4%EB%93%9C%EB%A6%AC%EC%8A%A4-headless-%EC%84%9C%EB%B9%84%EC%8A%A4)
> 
