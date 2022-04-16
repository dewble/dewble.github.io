---
title: \[Kubernetes\]\(Concept\)디플로이먼트(Deployment)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Workloads
- Deployment
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
쿠버네티스에서 상태가 없는 앱을 배포할 때 사용하는 가장 기본적인 컨트롤러이다.  
Deployment는 Pod와 ReplicaSet에 대한 선언적 업데이트를 제공한다.  
> 참고: Deployment가 소유하는 ReplicaSet은 관리하지 말아야 한다.
>

---

# Example - Deployment

deployment-nginx.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3 # 생성할 pod 개수
  selector:
    matchLabels: # metadata.labels 하위 필드와 같은 설정
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers: # 실제 사용하려는 컨테이너 이름과 이미지 정보 작성
      - name: nginx-deployment
        image: nginx:latest
        ports:
        - containerPort: 80
```

deployment 생성 & 확인

```bash
$ kubectl apply -f deployment-nginx.yaml
deployment.apps/nginx-deployment created

$ k get deploy,rs,rc,pods | grep nginx-deployment
NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment             3/3     3            3           66s

NAME                                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-69cfdf5bc7             3         3         3       66s

NAME                                              READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-69cfdf5bc7-89djk             1/1     Running   0          66s
pod/nginx-deployment-69cfdf5bc7-r7dln             1/1     Running   0          66s
pod/nginx-deployment-69cfdf5bc7-shzw9             1/1     Running   0          66s
```

- Deployment → nginx-deployment
- 이 Deployment가 관리하는 ReplicaSet → nginx-deployment-69cfdf5bc7
- ReplicaSet이 관리하는 pod → nginx-deployment-69cfdf5bc7-89djk
- 69cfdf5bc7 → 레플리카세트를 구분하는 UUID 해쉬 문자

---

# 디플로이먼트 상태 (status)

### Progressing → complete/ failed

Progressing

- 새로운 ReplicaSet을 만들때
- 새로운 ReplicaSet의 Pod 개수를 늘릴 때
- 예전 ReplicaSet의 Pod 개수를 줄일 때
- 새로운 Pod가 준비 상태가 되거나 이용 가능한 상태가 되었을 때

Complete

- Deployment가 관리하는 모든 ReplicaSet가 업데이트 완료되었을 때
- 모든 ReplicaSet가 사용 가능해졌을 때
- 예전 ReplicaSet가 모두 종료 되었을 때

Failed

- 쿼터 부족
- readinessProbe 진단 실패
- 컨테이너 이미지 가져오기 에러
- 권한 부족
- 제한 범위 초과
- 앱 실행 조건을 잘못 지정

### .spec.progressDeadlineSeconds

시간이 지났을 때 상태를 False로 바꾼다.

```bash
$ kubectl get deploy nginx-deployment -o=jsonpath="{.spec.progressDeadlineSeconds}{'\n'}"
600

$ kubectl patch deployment/nginx-deployment -p "{\"spec\":{\"progressDeadlineSeconds\":2}}"
deployment.apps/nginx-deployment patched

$ kubectl get deploy nginx-deployment -o=jsonpath="{.spec.progressDeadlineSeconds}{'\n'}"
2
```

2초를 넘어갈 경우 Status= False가 된다.

```bash
$ kubectl describe deploy nginx-deployment
Name:                   nginx-deployment
Namespace:              kube-system

Pod Template:
  Labels:       app=nginx-deployment
  Annotations:  kubectl.kubernetes.io/restartedAt: 2020-09-19T09:11:51Z
  Containers:
   nginx-deployment:
    Image:        nginx:1.14
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable # 2초를 넘어갈 경우 Status= False가 된다.
```

--- 

> [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
>