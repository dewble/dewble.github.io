---
title: \[Kubernetes\]\(Security\)Use TLS certificates as secret
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Security
- Secret
- TLS
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Purpose
Create TLS secret for HTTPS use

# Generate openssl Cert for testing

  HTTPS 인증서를 저장하는 용도로 시크릿을 사용할 수 있다.

```bash
[dewble@instance-1 secret]$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=example.com"
Generating a 2048 bit RSA private key
..........................................+++
.......................+++
writing new private key to 'tls.key'
-----
```

# Create secret using openssl cert

```bash
[dewble@instance-1 secret]$ kubectl create secret tls tlssecret --key tls.key --cert tls.crt
secret/tlssecret created
```

kubectl create secret tls [secret-name] --key tls.key --cert tls.crt

## Check Secret

```bash
[dewble@instance-1 secret]$ kubectl get secret tlssecret -o yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvekNDQWVlZ0F3SUJBZ0lKQUlhU0FIRWgvRkRSTUEwR0NTcUdTSWIzRFFFQkN3VUFNQll4RkRBU0JnTlYKQkFNTUMyVjRZVzF3YkdVdVkyOXRNQjRYRFRJd01UQXdOREUxTWpNeU5Wb1hEVEl4TVRBd05ERTFNak15TlZvdwpGakVVTUJJR0ExVUVBd3dMWlhoaGJYQnNaUzVqYjIwd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3CmdnRUtBb0lCQVFDODJoOEpvak8rQ0FOR1NkYnM3ZDhSblZsVE12ZzRuQXRTYWhBaWx4SiswdmJoOW5rUHUySXQKeGV6TjF4ZVZEQUt3Q3BoRy8zV2RqSCsxTndnVHJDVWI4cEdCT1lzbkttMXloNDB0T1duZ3hFWGVFeDA2eVFpRAo4bGtEU25CUC9VSS9iYmh2ak04TVJHTmlXVmlZZjN0eTRNaFY1bkEyNFBJbUhBdml1VmxENUN6WUp4emtBRUN5CmZYSkErZHU1SnJyeWd0T2pKbUF1Q1NBbkxyWXVEc1ExZDY5NG5iUmFHSUdqOFhFcCtzSnB3NjJqTGozVXp2SzYKbDNTRWordTlTcndDZExQZnJ5OXRvQmdSdGVqRzNqNjgrSGFEQ0RIUHFrMVBjeWhzdHVGKzZQei9PTmgySVV2egpZR3FLcFMvSjg2U2sxZHhzenY0a1pyZFBTVG51NmdFVEFnTUJBQUdqVURCT01CMEdBMVVkRGdRV0JCUjUyNTlpCjVpYW9YZ2RybXV0Z01HNW40dEh2WmpBZkJnTlZIU01FR0RBV2dCUjUyNTlpNWlhb1hnZHJtdXRnTUc1bjR0SHYKWmpBTUJnTlZIUk1FQlRBREFRSC9NQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUNsazU0cmhwQyttVDFWQXc4SApyRTlJLzBoUnd4bU9nUlhPWE55cUdicHFrajFLclZxR0xHOVk2YjlNSFUveEtxcklTQ2hWSFVqWlNTUWc1dFFsClUrd09jYVB2VEFDQWhBZzFUYTJIUXJ3ZWQ0RGQ2Z0poYVdkWVJLZndEelU0S0F6c0pkTEE4TDhHdzd3VzR1aEgKUEcyNUFWVy9OMTgxRFJrK1FxOEw4ais1SnVyWnc2dTBoN2VWdEVNMmN6SU42T1gzcFNBSFVqbFhBNk9yclZHcgpNUWpwdGJyV1ZLZW9iVFk4S3ptL0dRblpPNlNvcmQ2V3lwb0xtMDFFS3YzdGI3YmhVYVE4VTYrcWFPRkxCYkk1CndHcmY5alQvVTJpRjIxS0tLSUdmR3B2QzIvZVI5Skxac2JReVRWQVd4SXRyOXEzTkhNOWNpZytsd1BsQlU4Y1YKRHR5RgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQzgyaDhKb2pPK0NBTkcKU2RiczdkOFJuVmxUTXZnNG5BdFNhaEFpbHhKKzB2Ymg5bmtQdTJJdHhlek4xeGVWREFLd0NwaEcvM1dkakgrMQpOd2dUckNVYjhwR0JPWXNuS20xeWg0MHRPV25neEVYZUV4MDZ5UWlEOGxrRFNuQlAvVUkvYmJodmpNOE1SR05pCldWaVlmM3R5NE1oVjVuQTI0UEltSEF2aXVWbEQ1Q3pZSnh6a0FFQ3lmWEpBK2R1NUpycnlndE9qSm1BdUNTQW4KTHJZdURzUTFkNjk0bmJSYUdJR2o4WEVwK3NKcHc2MmpMajNVenZLNmwzU0VqK3U5U3J3Q2RMUGZyeTl0b0JnUgp0ZWpHM2o2OCtIYURDREhQcWsxUGN5aHN0dUYrNlB6L09OaDJJVXZ6WUdxS3BTL0o4NlNrMWR4c3p2NGtacmRQClNUbnU2Z0VUQWdNQkFBRUNnZ0VBV1B2dDQvd3BwVURoU2gxQXlDTE5HTityVnlpTkRSOTV0anVEbnNqUVRqSFoKWWw5Z2E2ay9lWkhwSXBSVzZFUGdnRko5cmZadzFPdCt3VVJNNmZnVEJEZ25sMXdsMVM2SW80NTdWdlBXajdIcgp1ZGdIemNzcjJBQTVNUFBDTis1OWFLV3FZZVZYS2RDUGc2ZlZ0d1ZhaGFha3Z0VDF5dVh6TTBIRDEvQzl2dDdUCjhVNjhXcjM4T1NpeUdZWjhFTHovUlE5M08wYlByTkcvM3dVRDJLbjdvd3B1SW5XZ2R1YkppdVJsa3dIbXNqNW8KeDkvcHN5clhWSzlQVFFlem1hUUVvdFhpaUs3Q1NZMGhnVkY1d3oyZUhNc1k2WlZXeHJlRDVXRXByTjBTZG5NTAo3OG1PS0MwUzZNZHdzVnIyeTRjdytxQTNTMzFRVnNrYWl3NHk5aUs0QVFLQmdRRDVDTEdoSm82b3N4Rk1DODM1Cm8rQ2M4NFkyWGxxQ0hjSkxxMitPdjV4NDhCb2Z3MDl1Q0Vab3ZyUTJnUElJTXFEYW1GQ3pmWEo2QWhJQVJnR2sKVjdHdGttS3F2VXgxU090b2hmbGhFTkFERlBFR3NMbjNMU1JPR29qNmVNYnI1VnZDWUszMXkzZVZYRUMvWGlRWQpiQVlpUVhoc1hubFJ0akZVQmhQc0tJNFo4d0tCZ1FEQ0luaU5zMmpxQzFISzdtYmMyYVJ0Q0s0ZTUyNnBYOW14ClU2WHRubGt3WG9ScWRpTUNUWVFaaTNVUldOc21qTWZmZFhBWTZBbFpyN1k3eE4xVUhwSXNXN0JkeGFobitBS1cKZlVEeXR1eEQ3VEVzQUp6QUd4d1Bhakh5YWFyOXFXeFFXV1BuMDBxRVJUS08yUnV2VWtXYzVzM0ZIWXpqTFlpTApMVWZqNCtRa1lRS0JnUURYcXNQS1A4NEVJeSt6bi9WOVlJTEE2ZFV0ZUlFQmRpd3h4QUlVcWJRa3VDcW5uMGxHCmpUd01zanIzaUt3U2xXWGdhVkJhWVNXbXEreFMrRTJydVpaU0x4ZnJyWXh0ZGYwSXhCMjRCZ3RlMzkvc1gxaHQKeTFaSm5ZbExBUldrYlRrT0dSUU9iV3JlbXNvbjhLdHB5d04wM3lZZkU2SVZOYWQ2a05qb0NDY29LUUtCZ0dNTwpjN1RaOW81MWVDYXp2b2l5Qk5RZHVickxIQXdRZkdPZTZ1dDBBTTVOYkFObWhEYUlsdjd4eWFvd1RLSSs4ejF5CkR1Q21oUjdlS1g0VjFWazJ3QjhpS2J1dlAxN05qWVI4Sk1lenpwcGFUTnpHOHpTU29KNjg4UDlnSzMrREUyRnMKT3kzdkFmYTcyREJMVjNUOTVjZEpmWFUydnN5c1R4KzAyeG5ORG53QkFvR0FHaVpJbUxtRXNQblJEY1ZJWjNNRApjemFydk04cEJCUkJTTW1oNXNpWU5oWGJLemltZllZQm5xMFBrV21kTzZNN3ZuUGszQkllR0ZycW0rVGZOMFhICkZMRFgrcDFMSiszT051MU5PRktHQWN4UGhPM2xWT2RYQWlDNVJEVXVEQUNxUnB4UXI2YkVaYXpubFhraTZyNnEKWVdpOHdYN29WNUgvZzJadENrQ2RvWDg9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: "2020-10-04T15:24:10Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:tls.crt: {}
        f:tls.key: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2020-10-04T15:24:10Z"
  name: tlssecret
  namespace: default
  resourceVersion: "6325412"
  selfLink: /api/v1/namespaces/default/secrets/tlssecret
  uid: d38e204f-37f4-4946-8a1b-b7cc78a32c8e
**type: kubernetes.io/tls**
```

type: [kubernetes.io/tls](http://kubernetes.io/tls) → TLS 인증서를 사용한다는 뜻 

# Connect tls secret with ingress

Edit secretName to tls secret name

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
spec:
  rules:
  - host: example.com
    http:
      paths:
      - backend:
          service:
            name: demo
            port:
              number: 8080
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - example.com
    secretName: tlssecret 
```