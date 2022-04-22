---
title: \[Kubernetes\]\(Concept\)What is CronJob(크론잡)
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
tag:
- Kubernetes
- Concept
- Controller
- CronJob
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
크론잡은 백업, 리포트 생성 등의 정기적 작업을 수행하기 위해 사용된다. 각 작업은 무기한 반복되도록 구성해야 한다(예: 1일/1주/1달마다 1회). 작업을 시작해야 하는 해당 간격 내 특정 시점을 정의할 수 있다.

- 크론잡은 반복 일정에 따라 잡을 만든다.
- 하나의 크론잡 오브젝트는 *크론탭* (크론 테이블) 파일의 한 줄과 같다. 크론잡은 잡을 [크론](https://ko.wikipedia.org/wiki/Cron) 형식으로 쓰여진 주어진 일정에 따라 주기적으로 동작시킨다.

> 모든 크론잡의 시간은 kube-controller-manager(컨트롤러 프로세스를 실행하는 컨트롤 플레인 컴포넌트)
의 시간대를 기준으로 한다.  
> 

---

# Example - CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *" # 스케줄 지정
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args: # 쉘 스크립트로 간단한 환영 메시지를 출력
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

`.spec.jobTemplate` 하위 필드에 어떤 작업을 실행할지 설정한다.

```bash
$ kubectl apply -f cronjob.yaml
cronjob.batch/hello created
$ kubectl get cronjobs
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          8s
```

- SUSPEND → 현재 이 크론잡이 정지되어 있는지 나타낸다. False 이므로 정지되지 않았다.
- ACTIVE → 현재 실행 중인 잡이 있는지
- LAST SCHEDULE → 마지막으로 잡을 실행한 후 어느 정도 시간이 지났는지 나타낸다.

### 크론잡이 실행한 잡 확인

```bash
$ kubectl get jobs
NAME               COMPLETIONS   DURATION   AGE
hello-1600694340   1/1           15s        68s
hello-1600694400   1/1           5s         16s
pi                 1/1           63s        49m
```

- COMPLETIONS → 작업을 성공적으로 완료한 횟수/총 작업 횟수
- DURATION → 성공적으로 완료하는데 걸린 시간

### Job, CronJob 삭제

kubectl delete [Name of CronJob]

```bash
$ kubectl delete cronjobs --all
cronjob.batch "hello" deleted

$ kubectl get jobs
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           63s        52m

$ kubectl delete jobs --all
job.batch "pi" deleted

$ kubectl get jobs
No resources found in kube-system namespace.
```

---

> [https://kubernetes.io/ko/docs/concepts/workloads/controllers/cron-jobs/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/cron-jobs/)
>