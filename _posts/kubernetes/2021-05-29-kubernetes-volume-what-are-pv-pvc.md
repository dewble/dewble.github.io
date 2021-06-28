---
title: \[Kubernetes\]\(Volume\)What are PV, PVC?
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
- What-is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
### PersistentVolume
A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.  

*퍼시스턴트볼륨* (PV)은 관리자가 프로비저닝하거나 [스토리지 클래스](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/)를 사용하여 동적으로 프로비저닝한 클러스터의 스토리지이다. 노드가 클러스터 리소스인 것처럼 PV는 클러스터 리소스이다. PV는 Volumes와 같은 볼륨 플러그인이지만, PV를 사용하는 개별 파드와는 별개의 라이프사이클을 가진다. 이 API 오브젝트는 NFS, iSCSI 또는 클라우드 공급자별 스토리지 시스템 등 스토리지 구현에 대한 세부 정보를 담아낸다.

### PersistentVolumeClaim
A request for storage by a user. It is similar to a Pod.  

*퍼시스턴트볼륨클레임* (PVC)은 사용자의 스토리지에 대한 요청이다. 파드와 비슷하다. 파드는 노드 리소스를 사용하고 PVC는 PV 리소스를 사용한다. 파드는 특정 수준의 리소스(CPU 및 메모리)를 요청할 수 있다. 클레임은 특정 크기 및 접근 모드를 요청할 수 있다(예: ReadWriteOnce, ReadOnlyMany 또는 ReadWriteMany로 마운트 할 수 있음. [AccessModes](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#%EC%A0%91%EA%B7%BC-%EB%AA%A8%EB%93%9C) 참고).

> [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction)

## PV, PVC, Pod template
<center><img src="/assets/images/posts/kubernetes/volume/volume-pv-pvc1.png" width="150%" height="150%" ></center>  

---
# PV  (PersistentVolume)

볼륨 자체. 클러스터 안에서 자원으로 다룬다. 파드하고는 별개로 관리되고 별도의 생명 주기가 있다.

---
# PVC (PersistentVolumeClaim)

사용자가 PV에 하는 요청. 사용하고 싶은 용량은 얼마인지, 읽기/쓰기는 어떤 모드로 설정하고 싶은지 등을 정해서 요청한다.

쿠버네티스는 볼륨을 파드에 직접 할당하는 방식이 아니라 중간에 PVC를 두어 파드와 파드가 사용할 스토리지를 분리했다. 이런 구조로 파드는 각각의 상황에 맞게 다양한 스토리지를 사용할 수 있게 한다.

<center><img src="/assets/images/posts/kubernetes/volume/volume-pv-pvc2.png" width="150%" height="150%" ></center>  

---

# PV, PVC life cyccle

<center><img src="/assets/images/posts/kubernetes/volume/volume-pv-pvc3.png" width="150%" height="150%" ></center>  

## 1. Provisioning - 프로비저닝

PV를 만드는 단계.

### 2가지 방법

1.PV 를 먼저 만들어 두고 사용하는 정적 방법 

사용할 수 있는 스토리지 용량에 제한이 있을때 유용

2.요청이 있을때마다 PV를 만드는 동적 방법 

사용자가 PVC를 거쳐서 PV를 요청했을 때 생성해 제공한다.

## 2. Binding - 바인딩

프로비저닝으로 만든 PV를 PVC와 연결하는 단계.

PV와 PVC의 매핑은 1대1 관계

## 3. Using - 사용

PVC는 파드에 설정되고 파드는 PVC를 볼륨으로 인식해서 사용한다.

할당된 PVC는 파드를 유지하는 동안 계속 사용하며 시스템에서 임의로 삭제할 수 없다.
이 기능을 Storage Object in Use Protection 이라고 한다.

## 4. Reclaiming - 반환

사용이 끝난 PVC는 삭제되고 PVC를 사용하던 PV를 초기화(reclaim)하는 과정을 거친다. 이를 반환(Reclaiming) 이라고 한다.

초기화 정책은 Retain, Delete, Recycle가 있다.

### Retain

PV를 그대로 보존. PV를 재사용하려면 다음 순서대로 직접 초기화 해야한다.

1. PV 삭제. 만약 PV가 외부 스토리지와 연결되었다면 PV는 삭제되더라도 외부 스토리지의 볼륨은 그대로 남아 있다.
2. 스토리지에 남은 데이터를 직접 정리
3. 남은 스토리지의 볼륨을 삭제하거나 재사용하려면 해당 볼륨을 이용하는 PV를 다시 만든다.

### Delete

PV를 삭제하고 연결된 외부 스토리지 쪽의 볼륨도 삭제한다.

동적 볼륨으로 생성된 PV들은 기본 반환 정책이 Delete 이다

### Recycle

PV의 데이터들을 삭제하고 다시 새로운 PVC에서 PV를 사용할 수 있도록 한다.

현재는 동적 볼륨 할당을 기본 사용할 것은 추천