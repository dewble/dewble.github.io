---
title: \[Kubernetes\]\(CronJob\)How to use CronJob
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
- How to
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
- 실행 시간 제약
- 동시성 관리
- History Limit

---

# 실행 시간 제약, 동시성 관리

cronjob-concurrency.yaml

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-concurrency
spec:
  schedule: "*/1 * * * *"
  startingDeadlineSeconds: 600
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster; sleep 6000 # 6000초 동안 안끝나고 대기
          restartPolicy: OnFailure
```

**`.spec.startingDeadlineSeconds**` 

- 지정된 시간에 크론잡이 실행되지 못했을 때 필드 값으로 설정한 시간까지 지나면 크론잡이 실행되지 않게 한다. 이 필드 값을 설정하지 않으면 실행 시간이 지나더라도 제약 없이 잡이 실행된다.

**`.spec.concurrencyPolicy`** 

- 크론잡이 실행하는 잡의 ***동시성을 관리***한다.
    - 기본 값은 **Allow**고 크론 잡이 여러개 잡을 동시에 실행할 수 있도록 한다.
    - **Forbid** 로 설정하면 잡을 동시에 실행하지 않도록 한다. 새로운 잡을 실행할 시간이고 이전 잡 실행이 아직 완료되지 않은 경우, 크론 잡은 새로운 잡 실행을 건너뛴다.
    - **Replace** 로 설정하면 새로운 잡을 실행할 시간이고 이전 잡 실행이 아직 완료되지 않은 경우, 크론 잡은 현재 실행 중인 잡 실행을 새로운 잡 실행으로 대체한다.

### 적용

```bash
$ kubectl apply -f cronjob-concurrency.yaml
cronjob.batch/hello-concurrency created

$ kubectl get po | grep hello
hello-concurrency-1600782180-qt5vn            1/1     Running            0          20s
```

- 6000초를 대기하도록 설정하였으므로, 1분이 지나고 다음 잡을 실행하려고 할때 기존 잡이 남아 있는 상태가 된다.
- concurrencyPolicy: Forbid 로 설정하였으므로 잡이 동시에 실행되지 않고 기다린다. 즉, 처음 실행했던 작업이 끝나야 다음 작업이 실행된다.

### **`.spec.concurrencyPolicy`**: Allow 로 변경

```bash
$ kubectl edit cronjob hello-concurrency
cronjob.batch/hello-concurrency edited

concurrencyPolicy: Allow

$ kubectl get po | grep hello
hello-concurrency-1600782180-qt5vn            1/1     Running   0          4m14s
hello-concurrency-1600782360-gjhtj            1/1     Running   0          24s
hello-concurrency-1600782420-fkhxg            1/1     Running   0          14s
```

- 크론잡이 여러개 잡을 동시에 실행할 수 있도록 한다.

새로운 파드가 실행 됨을 확인 할 수 있다.

### **`.spec.concurrencyPolicy`: Replace 로 변경**

```bash
$ kubectl edit cronjob hello-concurrency
cronjob.batch/hello-concurrency edited
concurrencyPolicy: Replace

$ kubectl get po | grep hello
hello-concurrency-1600782180-qt5vn            1/1     Terminating        0          6m50s
hello-concurrency-1600782360-gjhtj            1/1     Terminating        0          3m
hello-concurrency-1600782420-fkhxg            1/1     Terminating        0          2m50s
hello-concurrency-1600782480-swgnw            1/1     Terminating        0          109s
hello-concurrency-1600782540-2gmrg            1/1     Running            0          10s
```

- 새로운 잡을 실행할 시간이고 이전 잡 실행이 아직 완료되지 않은 경우, 크론 잡은 현재 실행 중인 잡 실행을 새로운 잡 실행으로 대체한다.

남아 있던 잡들을 모두 종료시키고 새로 잡이 실행됨을 볼 수 있다.

### `suspend`: false → true

```bash
$ kubectl get cronjob,po | grep hello
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-concurrency   */1 * * * *   True      1        2m50s           11m
pod/hello-concurrency-1600782660-fgk27            1/1     Running            0          2m28s
```

더이상 크론잡이 실행되지 않고 멈춘다. 단, 기존 실행중이던 잡이 멈추지는 않는다.

시간이 지나 확인해보면 .spec.concurrencyPolicy: Replace 라서 기존 파드를 종료하고 새로 파드를 설정해야하지만 대기 중임을 알 수 있다.

---

# History Limit

### successfulJobsHistoryLimit, failedJobsHistroyLimit

```bash
cronjob.batch/hello-concurrency   */1 * * * *   True      1        13m             22m
pod/hello-1600783320-lx4xg                        0/1     Completed          0          2m40s
pod/hello-1600783380-cmk6w                        0/1     Completed          0          100s
pod/hello-1600783440-79s6n                        0/1     Completed          0          39s
pod/hello-concurrency-1600782660-fgk27            1/1     Running            0          13m
```

`.spec.successfulJobsHistoryLimit`

- default: 3
    - 첫번째 예제는 default로 설정되어 잡이 3개 남아 있음을 확인할 수 있다.

`.spec.failedJobsHistroyLimit` 

- default 1
- 0으로 설정하면 잡이 종료된 다음 내역을 저장하지 않는다.

---

> [https://kubernetes.io/ko/docs/tasks/job/automated-tasks-with-cron-jobs/](https://kubernetes.io/ko/docs/tasks/job/automated-tasks-with-cron-jobs/)
>