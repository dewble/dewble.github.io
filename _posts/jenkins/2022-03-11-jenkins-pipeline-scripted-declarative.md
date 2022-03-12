---
title: \[Jenkins\]\(Pipeline\)Scripted VS Declarative
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Jenkins
- Pipeline
- Scripted
- Declarative
- CI/CD
categories:
- Jenkins
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
# Scripted VS Declarative
- Declarative pipeline은 Jenkins pipeline의 최신 기능
- Declarative pipeline은 Scripted pipeline 보다 더욱 풍부한 기능 제공
- Declarative pipeline은 pipeline 코드를 더 쉽게 작성하고 읽을 수 있도록 설계
- 모두 방법 groovy DSL을 기반으로 동작한다.
    - Scripted: groovy 기반에 구축된 첫 번째 pipeline이기 때문에 더 엄격한 groovy 기반 구문을 사용
    - Declarative: 더 간단하고 더 많은 옵션이 있는 Groovy 구문을 제공하기 위해 도입

> Scripted VS Declarative  
[https://www.jenkins.io/doc/book/pipeline/#declarative-versus-scripted-pipeline-syntax](https://www.jenkins.io/doc/book/pipeline/#declarative-versus-scripted-pipeline-syntax)

> groovy DSL(Domain Specific Language) : 어떤 목적이 있고 그 목적만 달성할 수 있는 언어  
[https://lannstark.tistory.com/13](https://lannstark.tistory.com/13)
 

## Declarative

- pipeline {} 으로 시작
- steps 단계 필요
- Jenkins에서 제공하는 Syntax가 Delcarative로 작성
    - Snippet Generator의 예제는 양쪽에서 사용가능

<center><img src="/assets/images/posts/jenkins/pipeline-scripted-declarative.png" width="150%" height="150%" ></center>  


- pipeline syntax doc → Delcarative으로 작성되어 있다
[https://www.jenkins.io/doc/book/pipeline/syntax/](https://www.jenkins.io/doc/book/pipeline/syntax/)
- e.g. 작성 예시

```jsx
pipeline {
   agent any

	environment {
     mvnHome = "/home/jenkins/apache-maven-3.8.1-jnms"
   }

   stages {
      stage('Hello') {
         steps {
            echo 'Hello World'
         }
      }
   }
}
```

## Scripted

- e.g. 작성 예시

```jsx
node {
   def mvnHome = "/home/jenkins/apache-maven-3.8.1-jnms"

   stage('Hello') {
       echo 'Hello World'
   }
}
```