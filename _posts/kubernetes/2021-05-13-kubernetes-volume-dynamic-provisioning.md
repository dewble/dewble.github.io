---
title: \[Kubernetes\]\(Volume\)Dynamic Provisioning with NFS in Kubernetes
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Volume
- PersistentVolumeClaim
- PersistentVolume
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
PVC를 생성할 때 PV 가 자동으로 생성되는 Dynamic Provisioning 을 NFS 기반으로 운영
<center><img src="/assets/images/posts/kubernetes/volume/volume-dynamic-pvc.png" width="150%" height="150%" ></center>  

---

# 1. Create Service Account - API 인증 구성

provisioner가 사용할 Service Account(nfs-pod-provisioner-sa)를 만들어 ClusterRole을 통해 pv와 pvc에 [get, list, watch, create, delete] 권한을 가질 수 있도록 API 인증을 구성

```bash
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-pod-provisioner-sa

---
kind: ClusterRole 
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-clusterRole
rules:
  - apiGroups: [""] 
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["watch","create", "update", "patch"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-rolebinding
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa 
    namespace: default
roleRef: 
  kind: ClusterRole
  name: nfs-provisioner-clusterRole
  apiGroup: rbac.authorization.k8s.io

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa
    namespace: default
roleRef:
  kind: Role
  name: nfs-pod-provisioner-otherRoles
  apiGroup: rbac.authorization.k8s.io
```

```bash
[root@master nfs-pv-provisioner]# kubectl apply -f nfs-prov-sa.yaml 
serviceaccount/nfs-pod-provisioner-sa created
clusterrole.rbac.authorization.k8s.io/nfs-provisioner-clusterRole created
clusterrolebinding.rbac.authorization.k8s.io/nfs-provisioner-rolebinding created
role.rbac.authorization.k8s.io/nfs-pod-provisioner-otherRoles created
rolebinding.rbac.authorization.k8s.io/nfs-pod-provisioner-otherRoles created
```

# 2. Create storageclass - storageclass 를 구성하여 PV 나 PVC에서 사용할 수 있도록 구성

pvc에서 pv 의 name을 지정하지 않고, 이제 storageclass name으로 볼륨 요청을 하게 된다.

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: join-nfs-storageclass # IMPORTANT pvc needs to mention this name
provisioner: join-nfs-provisioner # name can be anything 
parameters:
  archiveOnDelete: "false"
```

- 애플리케이션이 제거된 후에도 데이터를 유지하려면 archiveOnDelete를 true로 설정한다.

```bash
[root@master nfs-pv-provisioner]# kubectl apply -f storageclass-nfs.yaml 
storageclass.storage.k8s.io/nfs-storageclass created

[root@master nfs-pv-provisioner]# kubectl get storageclasses.storage.k8s.io 
NAME               PROVISIONER   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-storageclass   nfs-test      Delete          Immediate           false                  12m
```

# 3. Create a Provisioner to automatically generate PV

```bash
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-pod-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-pod-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-pod-provisioner
    spec:
      serviceAccountName: nfs-pod-provisioner-sa # name of service account
      containers:
        - name: nfs-pod-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-provisioner-v
              mountPath: /persistentvolumes
          ****env:
            - name: PROVISIONER_NAME # do not change
              value: join-nfs-provisioner # SAME AS PROVISIONER NAME VALUE IN STORAGECLASS
            - name: NFS_SERVER # do not change
              value: 10.50.20.40 # Ip of the NFS SERVER
            - name: NFS_PATH # do not change
              value: /k8s-dev # path to nfs directory setup
      volumes:
       - name: nfs-provisioner-v # same as volumemounts name
         nfs:
           server: 10.50.107.23
           path: /k8s-dev
```

```bash
[root@master nfs-pv-provisioner]# kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
nfs-pod-provisioner-ddbfdfb95-sw8st   1/1     Running   0          13m
```

# 4. Verify that pv is automatically generated when pvc is created

### Create pvc

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc-test
spec:
  storageClassName: join-nfs-storageclass # SAME NAME AS THE STORAGECLASS
 **** accessModes:
    - ReadWriteOnce #  must be the same as PersistentVolume
  resources:
    requests:
      storage: 1Gi
```

### verify that pv is automatically geberated

```bash
[root@master nfs-pv-provisioner]# kubectl apply -f nfs-dynamic-pvc.yaml 
persistentvolumeclaim/nfs-pvc-test created

[root@master nfs-pv-provisioner]# kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS       REASON   AGE
persistentvolume/pvc-a6793c2a-3a1a-4f3e-a8f1-3e0781462233   1Gi        RWO            Delete           Bound    default/nfs-pvc-test   join-nfs-storageclass            15m

NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
persistentvolumeclaim/nfs-pvc-test   Bound    pvc-a6793c2a-3a1a-4f3e-a8f1-3e0781462233   1Gi        RWO            join-nfs-storageclass   15m

```

만들지 않았던 PV가 자동으로 생성되면서 PVC가 Bound되는 것을 확인할 수 있다.

# 5. Create if the storage space is created on NFS server

NFS 서버(station011)에서도 storage space가 생성되었는지 확인

```bash
[root@station011 /]# ll /dynamicdir/
total 0
drwxrwxrwx. 2 root root 6 Nov 20 11:14 default-nfs-pvc-test-pvc-a6793c2a-3a1a-4f3e-a8f1-3e0781462233
```

# 6. PV is also deleted when PVC is deleted

```bash
kubectl delete pv nfs-pvc-test
kubectl get pvc,pv
```

# 7. NFS 서버에서도 확인

```bash
[root@station011 /]# ll /dynamicdir/
```