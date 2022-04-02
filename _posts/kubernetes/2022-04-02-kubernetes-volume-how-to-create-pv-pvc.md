---
title: \[Kubernetes\]\(Volume\)How to create pv and pvc in kubernetes
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- volume
- persistentvolume
- persistentvolumeClaim
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
아래 템플릿을 사용하여 hostpath pv,pvc 를 생성할 수 있다

---

# PersistentVolume Template

pv-hostpath.yaml

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  capacity:
    storage: 2Gi #1 스토리지 용량
  volumeMode: Filesystem #2 
  accessModes: #3
  - ReadWriteOnce
  storageClassName: manual #4
  persistentVolumeReclaimPolicy: Delete #5
  hostPath: #6
    path: /data/storage/test-pv
```

```bash
[dewble@instance-1 volume]$ kubectl -f apply pv-hostpath.yaml
[dewble@instance-1 volume]$ kubectl get pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-hostpath   2Gi        RWO            Delete           Available           manual                  7s
```

### STATUS

Available → PVC 에서 사용할 수 있도록 준비됨

Bound → 특정 PVC에 연결됨

Released → PVC는 삭제되었고 PV는 아직 초기화 되지 않은 상태

Failed → 자동 초기화 실패

#3 

볼륨의 읽기/쓰기 옵션을 설정.

볼륨은 한번에 .accessModes 필드를 하나만 설정할 수 있으며 필드 값은 세가지가 있다.

- ReadWriteOnce: 노드 하나에만 볼륨을 읽기/쓰기하도록 마운트할 수 있음
- ReadOnlyMany: 여러 개 노드에서 읽기 전용으로 마운트할 수 있음
- ReadWriteMany: 여러 개 노드에서 읽기/쓰기 가능하도록 마운트할 수 있음.

#4

스토리지 클래스를 설정하는 필드.

특정  스토리지 클래스가 있는 PV는 해당 스토리지 클래스에 맞는 PVC와만 연결된다.

.storageClassName 필드 설정이 없으면 .storageClassName 필드 설정이 없는 PVC와만 연결된다.

#5

PV가 해제되었을 때의 초기화 옵션을 설정한다.

앞에서 살펴봤던 것처럼 Retain, Recycle, Delete 정책 중 하나를 설정한다.

#6

해당 PV의 볼륨 플러그인을 명시한다.

하위의 .path 필드에는 마운트시킬 로컬 서버의 경로를 설정한다.

---

# PersistentVolumeClaim Template

pvc-hostpath.yaml

```bash
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-hostpath
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

주요 필드는 pv-hostpath.yaml 과 같다.

.requests.storage → 자원을 얼마나 사용할 것인지 요청. 필드 값은 앞에서 만든 PV의 용량을 초과하면 안된다.

만약 초과하면 만들 수 있는 PV 가 없으므로 Pending 상태가 된다.

### pvc 확인

```bash
[dewble@instance-1 volume]$ kubectl -f apply pvc-hostpath.yaml
[dewble@instance-1 volume]$ kubectl get pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-hostpath   Bound    pv-hostpath   2Gi        RWO            manual         2m11s
```

STATUS → Bound

VOLUME → pv-hostpath

### pv 확인

```bash
[dewble@instance-1 volume]$ kubectl get pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pv-hostpath   2Gi        RWO            Delete           Bound    default/pvc-hostpath   manual                  5h7m
```

PV가 PVC에 연결되어서 STATUS 항목이 Bound 로 나타난다

# PVC 크기 늘리기

.spec.storageClassName.allowVolumeExpansion → True

.spec.[resources.requests.storage](http://resources.requests.storage) → 필드 값에 더 높은 용량을 설정한 후 클러스터에 적용.

### 기존 파드가 사용중인 볼륨 크기를 늘리려면 파드를 재시작해야 한다.