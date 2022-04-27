---
title: \[Kubernetes\]\(Service\)How to use Kubernetes NodePort
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Service
- NodePort
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
<center><img src="/assets/images/posts/kubernetes/service/service-nodeport-1.png" width="150%" ></center>  

사용자는 서비스(NodePort)를 통해 모든 노드의 지정된 포트로 파드에 접근할 수 있다.

클러스터 외부에서 클러스터 안 파드로 접근할 때 사용할 수 있는 가장 간단한 방법이다.

node1:30080, node2:30080 처럼 노드에 상관 없이 서비스에 지정된 포트를 사용하여 파드에 접근할 수 있다.

특이한점은 node1에만 실행되어 있는 파드가 있을때도 node2:30080 으로 접근했을 때 node1에 실행된 파드로 연결한다.

# Example - Nodeport

<center><img src="/assets/images/posts/kubernetes/service/service-nodeport-2.png" width="150%" ></center>  

사용자는 nodeIP:30080 을 통해 파드 A에 접근할 수 있다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
    selector:
    app: nginx-for-svc
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
```

```bash
$ kubectl apply -f service-nodeport.yaml
service/nodeport-service created
$ kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
clusterip-service           ClusterIP   10.233.54.164   <none>        80/TCP                   46h
nodeport-service            NodePort    10.233.39.47    <none>        80:30080/TCP             24h
```

---

# Option

### 유저 스페이스(User space) 프록시 모드

`.spec.sessionAffinity` : 한 클라이언트의 요청을 같은 인스턴스로 접속할 수 있도록 세션을 유지해준다(sticky session)

- `None`(default): 옵션을 사용하지 않는다.
- `ClusterIP`: 특정 클라이언트에서 생성된 모든 요청을 같은 pod로 연결

### 외부 트래픽 정책

`.spec.externalTrafficPolicy`: 외부 소스에서 오는 트래픽이 어떻게 라우트될지 제어

- `Cluster`(default) → 외부 트래픽을 Status Ready상태의 모든 엔드포인트로 라우트
- `Local` → 연결한 노드 - 로컬 엔드포인트로만 라우트한다.
    - NodePort로 연결될 때 트래픽을 연결한 노드에서 실행중인 Pod로 리다이렉트 access 할 수 있다.
    - 만약 트래픽 정책이 `Local`로 설정되어 있는데 노드-로컬 엔드포인트가 하나도 없는 경우, kube-proxy는 연관된 서비스로의 트래픽을 포워드하지 않는다.

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
>