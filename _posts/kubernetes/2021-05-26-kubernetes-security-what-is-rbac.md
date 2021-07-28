---
title: \[Kubernetes\]\(Security\)What is RBAC ? ( Role based access control )
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Security
- Helm
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
When lots of users use a cluster, we can seperate access rights with namespace or API.

클러스터 하나를 여러 명이 사용할 때는 API나 네임스페이스별로 권한을 구분해서 권한이 있는곳에만 접근하도록 만들 수 있다.

---

# Kubernetes API Access Control type

### ABAC(Attribute-based access control)

access control paradigm whereby access rights are granted to users through the use of policies which combine attributes together.

### RBAC(Role-based access control)

method of regulating access to computer or network resources based on the roles of individual users within your organization.

---

# RBAC architecture in kubernetes


<center><img src="/assets/images/posts/kubernetes/security/security-what-is-rbac.png" width="150%" height="150%" ></center>  

---

# Role - Role / ClusterRole

특정 API나 자원 사용 권한들을 명시해둔 규칙의 집합.

일반 롤, 클러스터 롤 두가지가 존재.

## Role - 일반 롤

해당 롤이 속한 **네임스페이스에만** 적용.  추가로 네임스페이스에 한정되지 않은 자원과 API들의 사용 권한을 설정할 수 있다.

```bash
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default #1 이 롤이 속한 기본 네임스페이스
  name: read-role # 롤의 이름
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

metadata.namespace 항목을 이용하여 특정 네임스페이스에 권한 적용

## ClusterRole - 클러스터롤

특정 네임스페이스 사용 권한이 아닌 **클러스터 전체 사용 권한**을 관리한다.

```bash
kind: ClusterRole #1
apiVersion: rbac.authorization.k8s.io/v1 #2
metadata:
  name: read-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

일반 롤과 달리 namespace 항목이 없다. 클러스터 전체에 적용

---

# RoleBinding - RoleBinding / ClusterRoleBinding

### **롤과 사용자를 묶는 역할.**

권한 → 롤과 클러스터롤로 구분

바인딩 → 롤바인딩과 클러스터롤바인딩으로 구분

## RoleBinding - 롤바인딩

특정 네임스페이스 하나에 적용

```bash
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-rolebinding
  namespace: default
subjects: #1
- kind: ServiceAccount
  name: myuser
  apiGroup: ""
roleRef: #2
  kind: Role
  name: read-role
  apiGroup: rbac.authorization.k8s.io
```

### #1 metadata.subjects

어떤 유형의 사용자 계정과 연결하는지 설정.

apiGroup: "" → 핵심 API 그룹으로 설정했다는 뜻.

### #2 metadata.roleRef

사용자에게 어떤 롤을 할당할지를 설정.

roleRef.kind → Role or ClusterRole

## ClusterRoleBinding - 클러스터 롤바인딩

한번 설정 했을 때 클러스터 전체에 적용

```bash
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: myuser
  namespace: default #1
  apiGroup: "" #2
roleRef: #3
  kind: ClusterRole
  name: read-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

### #1 subjects.namespace

클러스터 관련 설정이므로 .metadata 하위가 아닌 .subjects[] 하위에 .namespace 필드가 있다.

### #2 subjects.apiGroup

어떤 유형의 사용자 계정과 연결하는지 설정.

apiGroup: "" → 핵심 API 그룹으로 설정했다는 뜻.

### #2 roleRef

사용자에게 어떤 롤을 할당할지를 설정.

roleRef.kind → Role or ClusterRole