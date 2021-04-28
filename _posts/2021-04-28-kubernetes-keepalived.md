---
title: \[kubernetes\]\(kubeadm\) HA Configuration1-keepalived
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- kubernetes
- kubeadm
- keepalived
categories:
- kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---

Text

# var, let, const
변수를 설정하는 방법에는 크게 3가지 선언문이 존재합니다. ES6 이전에 존재하였던 var과 ES6에 등장한 let, cont 변수에  관해 한번 차이점을 
비교해보겠습니다.

## var
var의 경우는 ES6 이전의 문법으로 매우 유연한 방식의 변수 선언 방법입니다. var의 경우는 let, cont와 다르게 블럭 단위 scope가 
아닌 함수 단위 scope입니다. 때문에 if {} 해당 블럭내부에 선언되나, 밖에 선언되나 같은 scope입니다. 이 외 javascript 변수에 
관한 특징들은 나열하지 않겠습니다.