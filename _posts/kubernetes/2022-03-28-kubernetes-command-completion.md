---
title: \[Kubernetes\]\(Command\)kubectl command 자동 완성
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Command
- Completion
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# kubectl cheat sheet setting
bashrc 혹은 profile 설정이 가능 곳에 아래 내용을 입력하고 적용한다.

```bash
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```

## Example

tap을 눌러 자동 완성 할 수 있다

```bash
k get pod
k get node
```
