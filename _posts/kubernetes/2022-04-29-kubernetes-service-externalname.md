---
title: \[Kubernetes\]\(Service\)How to use Kubernetes ExternalName
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Service
- ExternalName
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
<center><img src="/assets/images/posts/kubernetes/service/service-externalname.png" width="100%" ></center>
값과 함께 CNAME 레코드를 리턴하여, 서비스를 externalName 필드의 콘텐츠 (예:foo.bar.example.com
)에 매핑한다. 어떤 종류의 프록시도 설정되어 있지 않다.

→ 외부 서비스를 쿠버네티스 내부에서 호출하고자 할때 사용할 수 있다.

> 참고: `ExternalName` 유형을 사용하려면 kube-dns 버전 1.7 또는 CoreDNS 버전 1.7 이상이 필요하다.
> 

---

# Example - ExternalName

```yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname-service
spec:
  type: ExternalName # 
  externalName: google.com #연결하려는 외부 도메인 값을 설정
```

`.spec.externalName`: google.com #연결하려는 외부 도메인 값을 설정

```bash
$ kubectl get svc
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
externalname-service        ExternalName   <none>          google.com    <none>                   8s
```

- default 네임스페이스의 externalname-service 라는 서비스를 google.com에 매핑한다.
    - externalname-service 서비스로 들어오는 모든 요청을 google.com으로 포워딩 해준다.
- EXTERNAL-IP 항목이 앞서 설정한 값으로 지정된다.

---

> [https://kubernetes.io/ko/docs/concepts/services-networking/service/#externalname](https://kubernetes.io/ko/docs/concepts/services-networking/service/#externalname)
>