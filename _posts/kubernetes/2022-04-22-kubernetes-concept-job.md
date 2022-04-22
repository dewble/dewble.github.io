---
title: \[Kubernetes\]\(Concept\)잡(Job)
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
- Job
- What is
categories:
- Kubernetes
toc: true
toc_sticky: true
toc_label: Contents
popular: true
---
하나 이상의 파드를 생성하고 지정된 수의 파드가 성공적으로 종료될 때까지 계속해서 파드의 실행을 재시도한다.

→ 특정 개수만큼의 파드를 정상적으로 실행 종료함을 보장한다.

→ Pod 실행 실패, 하드웨어 장애 발생, 노드 재시작 등 문제가 발생하면 다시 Pod를 실행한다.

실행된 후 종료해야 하는 성격의 작업을 실행시킬때 사용하는 컨트롤러이다.

Job 은 apply로 update 할 수 없고 삭제 후 다시 apply해야 한다.

Job 하나가 Pod를 여러개 실행할 수도 있다.

---

# Example - Job

job.yaml

```yaml
apiVersion: batch/v1 
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"] #원주율을 게산하는 펄 명령을 설정
      restartPolicy: Never 
  backoffLimit: 4 
```

- `.spec.template.spec.restartPolicy:`
    - `Never`로 설정하여 파드가 재시작하지 않는다.
    - `OnFailure` 로 설정하면 파드 안 컨테이너가 비정상 종료됐거나 다양한 이유로 실행이 정상 종료되지 않았을 때 파드는 그 노드에 유지되고 컨테이너를 다시 시작하도록 한다.
- `.spec.backoffLimit:`
    - 잡이 실행 실패 했을 때 최대 몇번까지 재시작할것 인지 설정
    - 파드 비정상 실행 종료의 백오프 정책이라고 한다. default: 6
    - 보통 잡 컨트롤러가 재시작을 관리할 때마다 시간 간격을 늘린다.

### 클러스터에 적용한 후 describe

```bash
$ kubectl apply -f job.yaml
job.batch/pi created

$ kubectl describe job pi
Name:           pi
Namespace:      kube-system
Selector:       controller-uid=9db0740d-97f2-43f7-adbb-6809e39704df
Labels:         controller-uid=9db0740d-97f2-43f7-adbb-6809e39704df
                job-name=pi
Annotations:    <none>
Parallelism:    1
Completions:    1
Start Time:     Mon, 21 Sep 2020 12:30:25 +0000
Completed At:   Mon, 21 Sep 2020 12:31:28 +0000
Duration:       63s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=9db0740d-97f2-43f7-adbb-6809e39704df
           job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  82s   job-controller  Created pod: pi-ppbxd
  Normal  Completed         20s   job-controller  Job completed
```

63초 동안 파드를 하나 실행하고 정상적으로 종료했다.

### 원주율 계산 결과 확인

```bash
$ kubectl get pods | grep pi
pi-ppbxd                                      0/1     Completed          0          3m7s
$ kubectl logs pi-ppbxd
3.14159265358979323846264338327950288419716939937
```

---

# 잡 병렬성 관리

잡 하나가 몇 개의 ***파드를 동시에 실행***할지를 '잡 병렬성' 이라고 한다.

.sepc.parallelism 필드에 설정할 수 있다. 

기본 값은 1 이고 0으로 설정하면 잡을 정지할 수 있다.

---

# 잡의 종류

비-병렬(Non-parallel) 잡

고정적(fixed)인 완료 *횟수*를 가진 병렬 잡

작업 큐(queue)가 있는 병렬 잡

## 비-병렬(Non-parallel) 잡

파드 하나만 실행된다. 파드가 정상적으로 실행종료(Succeeded)되면 잡 실행을 완료한다.

- `.spec.completions` 과 `.spec.parallelism` 필드를 설정하지 않는다. default: 1
    - `.spec.completion` → 정상적으로 실행 종료되어야 하는 파드 개수가 몇 개 인지를 설정
    - `.spec.parallelism` → 병렬성을 지정하는 필드로 동시에 몇 개의 파드가 실행되어도 괜찮은지를 설정
    - 1로 설정하면 한 번에 파드 하나만 실행되어야 하고 정상적으로 실행 종료되어야 하는 파드 개수도 1개이다

## *고정적(fixed)인 완료 횟수*를 가진 병렬 잡

`.spec.completion` → 필드 값으로 양수를 설정. 필드 값이 1이면 정상적으로 실행 종료된 파드가 1개만 생겨도 잡이 완료.

`.spec.parallelism` → 필드를 설정하지 않거나 기본값인 1로 설정

## *작업 큐(queue)*가 있는 병렬 잡

`.spec.completion` → 필드는 설정하지 않고, `.spec.parallelism` 필드는 양수로 설정한다.

completion 값을 설정하지 않으면 기본값이 parallelism 필드와 동일하게 설정된다.

---

# 비정상적으로 실행 종료된 파드 관리하기

`.spec.template.spec.restartPolicy` 필드를 지정해 컨테이너 재시작 정책을 설정할 수 있다.

OnFailure → 원래 실행 중이던 노드에서 컨테이너를 재시작

Never → 재시작을 막을 수도 있다.

> 참고: 만약 잡에 `restartPolicy = "OnFailure"` 가 있는 경우 잡 백오프 한계에 도달하면 잡을 실행 중인 파드가 종료된다. 이로 인해 잡 실행 파일의 디버깅이 더 어려워질 수 있다. 디버깅하거나 로깅 시스템을 사용해서 실패한 작업의 결과를 실수로 손실되지 않도록 하려면 `restartPolicy = "Never"` 로 설정하는 것을 권장한다.
> 

---

# Job의 종료와 정리

잡이 정상적으로 실행 종료되면 파드가 새로 생성되지도 삭제되지도 않는다. 또한 잡도 남아 있다.

파드나 잡이 삭제되지 않고 남아 있으면 로그에서 에러나 경고를 확인할 수 있고, 잡의 상태도 계속해서 확인할 수 있다. 이를 이용해 필요한 분석을 할 수 있다.

잡 삭제는 kubectl delete job [jobname] 명령으로 사용자가 직접해야 한다. 잡을 삭제하면 관련된 파드들도 같이 삭제된다.

- `.spec.activeDeadlineSeconds` 필드에 시간을 설정한다. 지정된 시간에 해당 잡 실행을 강제로 끝내면서 모든 파드 실행도 죵료한다.
    - 시간을 지정해 잡 실행을 끝냈다면 잡의 상태를 확인했을 때 reason:DeadlineExceeded 로 표시된다.

---

# Job의 패턴

- 작업마다 잡을 하나씩 생성해 사용하는 것보다는 모든 작업을 관리하는 잡 하나를 사용하는 것이 좋다.
    - 잡을 생성하는 오버헤드가 크기 때문이다. 작업이 많아질수록 잡하나가 여러개 작업을 처리하는 것이 좋다.
- 작업 개수만큼 파드를 생성하는 것보다 파드 하나가 여러 개의 작업을 처리하는 것이 좋다.
    - 파드를 생성하는 오버헤드도 크므로 작업이 많아질수록 파드 하나가 여러 개 작업을 처리하는 것이 유리하다.
- 작업 큐(queue)를 사용한다면 카프카나 RabbitMQ 같은 서비스로 작업 큐를 구현하도록 기존프로그램이나 컨테이너를 수정해야 한다.
    - 작업 큐를 사용하지 않으며 그냥 기본 설정 그대로 컨테이너를 사용하므로 비효율 적이다

---

> [https://kubernetes.io/ko/docs/concepts/workloads/controllers/job/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/job/)
>