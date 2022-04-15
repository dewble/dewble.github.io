---
title: \[Kubernetes\]\(Concept\)레플리카세트(ReplicaSet)
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
- ReplicaSet
- DaemonSet
- StatefulSet
- Job
- CronJob
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
레플리카세트의 목적은 레플리카 파드 집합의 실행을 항상 안정적으로 유지하는 것이다. 이처럼 레플리카셋은 보통 ***명시된 동일 파드 개수에 대한 가용성을 보증***하는데 사용한다.

집합 기반의 셀럭터를 지원한다. 

집합 기반의 셀럭터는 in, notin, exists 같은 연산자를 지원한다.

### 권장

디플로이먼트는 레플리카세트를 관리하고 다른 유용한 기능과 함께 파드에 대한 선언적 업데이트를 제공하는 상위 개념이므로 업데이트가 전혀 필요하지 않거나 custom 오케스트레이션이 필요 없는 경우가 아니면 레플리카세트를 직접 사용하기 보다는 디플로이먼트를 사용하는 것을 권장한다.

---

# ****Example****

replicaset-nginx.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
# 어떤 파드를 실행할지에 관한 정보를 설정. 
# .sepc.template 하위에 다시 .metadata와 .spec이라는 필드가 있다.
  template:
    metadata:
      name: nginx-replicaset
      labels:
        app: nginx-replicaset
# spec.containers[] 하위에 
# .name, .image, .ports[], .containerPort 필드를 이용해 컨테이너의 구체적인 명세를 설정한다.
    spec:
      containers:
      - name: nginx-replicaset
        image: nginx:latest
        ports:
        - containerPort: 80
  replicas: 3 # modify replicas according to your case
# 어떤 레이블(.lables)의 파드를 선택해서 관리할지를 설정.
  selector: 
# 레이블을 기준으로 파드를 관리하므로 실행 중인 파드를 중단하거나 재시작하지 않고 컨트롤러가 관리하는 파드를 변경할 수 있다.
# .spec.template.metadata.labes의 하위 필드 설정과 
# .spec.selector.matchlabels의 하위 필드 설정이 같아야 한다.
# 별도의 selector 설정이 없으면 .spec.template.metadata.labes.app 에 있는 내용을 기본값으로 설정한다.
    matchLabels:
      app: nginx-replicaset
```

클러스터에 적용 & pods 확인

```bash
$ kubectl apply -f replicaset-nginx.yaml
replicaset.apps/nginx-replicaset created

$ kubectl get pods | grep nginx-replicaset
nginx-replicaset-48qnh                        1/1     Running   0          44s
nginx-replicaset-6tdpv                        1/1     Running   0          44s
nginx-replicaset-mzvl7                        1/1     Running   0          44s
```

임의로 pod 1개 삭제 후 확인

```bash
$ kubectl delete pod nginx-replicaset-48qnh
pod "nginx-replicaset-48qnh" deleted
$ kubectl get replicaset,pods
NAME                                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-replicaset                        3         3         3       4m59s

NAME                                              READY   STATUS    RESTARTS   AGE
pod/nginx-replicaset-6tdpv                        1/1     Running   0          4m59s
pod/nginx-replicaset-mzvl7                        1/1     Running   0          4m59s
pod/nginx-replicaset-w24k5                        1/1     Running   0          2m48s

```

유지할 파드 개수를 조절하려면 yaml 파일 안 .spec.replicas 필드 값을 변경하고

kubectl apply -f replicaset-nginx.yaml 명령을 실행한다.

---

## 레플리카세트와 파드의 연관 관계

레플리카세트와 파드는 느슨하게 결합되어 있다. 

레플리카세트와 파드를 한꺼번에 삭제할 때는 kubectl delete replicaset [컨테이너이름] 명령을 실행하지만,

***--cascade=false*** 옵션을 사용하면 **레플리카세트가 관리하는 파드에 영향을 끼치지 않고 레플리카세트만 삭제할 수 있다.** 그럼 현재 실행 중인 파드를 관리하는 레플리카세트를 추가로 만들 수도 있다.

—cascade=fase

```bash
$ kubectl get replicaset
NAME                                    DESIRED   CURRENT   READY   AGE
nginx-replicaset                        3         3         3       35h

$ kubectl delete replicaset nginx-replicaset --cascade=false
replicaset.apps "nginx-replicaset" deleted
```

레플리카세트를 삭제할 때 ***파드는 남겨***두고 컨트롤러만 삭제하는것

삭제 후 레플리카세트와 파드정보 확인

```bash
$ kubectl get replicaset,pods | grep nginx
pod/nginx-replicaset-6tdpv                        1/1     Running   0          35h
pod/nginx-replicaset-mzvl7                        1/1     Running   0          35h
pod/nginx-replicaset-w24k5                        1/1     Running   0          35h
```

다시 kubectl apply -f replicaset-nginx.yaml 명령을 실행하여 파드를 관리하는 레플리카세트를 만든다.

```bash
$ kubectl apply -f replicaset-nginx.yaml
replicaset.apps/nginx-replicaset created
$ kubectl get replicaset,pods | grep nginx
NAME                                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-replicaset                        3         3         3       22s
NAME                                              READY   STATUS    RESTARTS   AGE
pod/nginx-replicaset-6tdpv                        1/1     Running   0          35h
pod/nginx-replicaset-mzvl7                        1/1     Running   0          35h
pod/nginx-replicaset-w24k5                        1/1     Running   0          35h
```

- DESIRED → 레플리카세트 설정에 지정한 파드 개수
- CURRENT → 레플리카세트를 이용해 현재 클러스터에 동작하는 실제 파드 개수.

동작 확인

```bash
$ kubectl delete pods nginx-replicaset-w24k5
pod "nginx-replicaset-w24k5" deleted

$ kubectl get replicaset,pods | grep nginx-replicaset
replicaset.apps/nginx-replicaset                        3         3         3       4m47s
pod/nginx-replicaset-5dnb5                        1/1     Running   0          25s
pod/nginx-replicaset-6tdpv                        1/1     Running   0          35h
pod/nginx-replicaset-mzvl7                        1/1     Running   0          35h
```

기존 pod을 삭제했을때 새로운 pod 이 생성된걸 확인할 수 있다.

실행 중인 pod의 .metadata.labels.app 필드를 수정

```bash
$ kubectl edit pod nginx-replicaset-5dnb5
pod/nginx-replicaset-5dnb5 edited
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-19T03:03:48Z"
  generateName: nginx-replicaset-
  labels:
    app: nginx-replicaset -> nginx-other
```

다시 pods 을 확인해 보면 새로운 pod이 추가 되어 pod 4개가 실행되는 것을 확인할 수 있다.

```bash
$ kubectl get replicaset,pods | grep nginx
replicaset.apps/nginx-replicaset                        3         3         3       12m
pod/nginx-proxy-instance-4                        1/1     Running     0          4d21h
pod/nginx-proxy-instance-5                        1/1     Running     0          4d21h
pod/nginx-replicaset-5dnb5                        1/1     Running     0          8m
pod/nginx-replicaset-6tdpv                        1/1     Running     0          35h
pod/nginx-replicaset-mzvl7                        1/1     Running     0          35h
pod/nginx-replicaset-pxg88                        1/1     Running     0          3m41s
```

각 파드의 .metadata.labels.app 필드 확인

```bash
$ kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.metadata.labels}{'\n'}{end}" | grep nginx
nginx-replicaset-5dnb5	{"app":"nginx-other"}
nginx-replicaset-6tdpv	{"app":"nginx-replicaset"}
nginx-replicaset-mzvl7	{"app":"nginx-replicaset"}
nginx-replicaset-pxg88	{"app":"nginx-replicaset"}
```

5dnb5 파드의 [metadata.labels.app](http://metadata.labels.app) 필드가 다른 것을 알 수 있다.

즉 해당 파드는 nginx-replicaset 이라는 레플리카세트에서 분리되었음을 뜻한다.

레플리카세트는 3개의 파드를 실행해야하므로 새로운 파드를 하나 생성한것이다.

---

> [https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
>