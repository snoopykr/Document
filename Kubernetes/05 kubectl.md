# Kubernetes Job

## 잡의 실행수

job-normal-end.yml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: normal-end
spec:
  template:
    spec:
      containers:
      - name: busybox
        image: busybox:latest
        command: ["sh",  "-c", "sleep 5; exit 0"]
      restartPolicy: Never
  completions: 6
  parallelism: 2        # 동시 실행수
```

```bash
$ kubectl apply -f job-normal-end.yml
job.batch/normal-end created

$ kubectl get job
NAME         COMPLETIONS   DURATION   AGE
normal-end   0/6           7s         7s

// # parallelism: 2 (동시 실행 X)
$ kubectl describe job normal-end
Name:             normal-end
Namespace:        default
Selector:         controller-uid=32104740-2124-4545-84f4-12f267a2fea3
Labels:           controller-uid=32104740-2124-4545-84f4-12f267a2fea3
                  job-name=normal-end
Annotations:      <none>
Parallelism:      2
Completions:      6
Completion Mode:  NonIndexed
Start Time:       Fri, 10 Dec 2021 14:04:12 +0900
Completed At:     Fri, 10 Dec 2021 14:05:06 +0900
Duration:         54s
Pods Statuses:    0 Running / 6 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=32104740-2124-4545-84f4-12f267a2fea3
           job-name=normal-end
  Containers:
   busybox:
    Image:      busybox:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 5; exit 0
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  55s   job-controller  Created pod: normal-end--1-k85s6
  Normal  SuccessfulCreate  46s   job-controller  Created pod: normal-end--1-cqx4c
  Normal  SuccessfulCreate  37s   job-controller  Created pod: normal-end--1-skx6g
  Normal  SuccessfulCreate  28s   job-controller  Created pod: normal-end--1-cxjpj
  Normal  SuccessfulCreate  19s   job-controller  Created pod: normal-end--1-j6gcf
  Normal  SuccessfulCreate  10s   job-controller  Created pod: normal-end--1-k8mjf
  Normal  Completed         1s    job-controller  Job completed

// parallelism: 2  (동시 실행 2)
$ kubectl describe job normal-end
Name:             normal-end
Namespace:        default
Selector:         controller-uid=fa334e73-2d17-4b10-b158-b4f2d561790c
Labels:           controller-uid=fa334e73-2d17-4b10-b158-b4f2d561790c
                  job-name=normal-end
Annotations:      <none>
Parallelism:      2
Completions:      6
Completion Mode:  NonIndexed
Start Time:       Fri, 10 Dec 2021 14:24:50 +0900
Completed At:     Fri, 10 Dec 2021 14:25:17 +0900
Duration:         27s
Pods Statuses:    0 Running / 6 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=fa334e73-2d17-4b10-b158-b4f2d561790c
           job-name=normal-end
  Containers:
   busybox:
    Image:      busybox:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 5; exit 0
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  2m16s  job-controller  Created pod: normal-end--1-t9722
  Normal  SuccessfulCreate  2m16s  job-controller  Created pod: normal-end--1-xgx49
  Normal  SuccessfulCreate  2m8s   job-controller  Created pod: normal-end--1-tzkrg
  Normal  SuccessfulCreate  2m7s   job-controller  Created pod: normal-end--1-6927m
  Normal  SuccessfulCreate  119s   job-controller  Created pod: normal-end--1-2ssfq
  Normal  SuccessfulCreate  118s   job-controller  Created pod: normal-end--1-vvrwm
  Normal  Completed         109s   job-controller  Job completed
```

## 비 정상종료 (Single Container)

job-abnormal-end.yml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: abnormal-end
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: busybox
        image: busybox:latest
        command: ["sh",  "-c", "sleep 5; exit 1"]
      restartPolicy: Never
```

```bash
$ kubectl apply -f job-abnormal-end.yml
job.batch/abnormal-end created

$ kubectl get jobs
NAME           COMPLETIONS   DURATION   AGE
abnormal-end   0/1           80s        80s

$ kubectl get all
NAME                        READY   STATUS              RESTARTS   AGE
pod/abnormal-end--1-d6zch   0/1     ContainerCreating   0          1s
pod/abnormal-end--1-vcvd8   0/1     Error               0          10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   28h

NAME                     COMPLETIONS   DURATION   AGE
job.batch/abnormal-end   0/1           10s        10s

$ kubectl describe job abnormal-end
Name:             abnormal-end
Namespace:        default
Selector:         controller-uid=eb398966-1a89-4c26-9d1c-758a698daf0b
Labels:           controller-uid=eb398966-1a89-4c26-9d1c-758a698daf0b
                  job-name=abnormal-end
Annotations:      <none>
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Fri, 10 Dec 2021 14:34:28 +0900
Pods Statuses:    0 Running / 0 Succeeded / 4 Failed
Pod Template:
  Labels:  controller-uid=eb398966-1a89-4c26-9d1c-758a698daf0b
           job-name=abnormal-end
  Containers:
   busybox:
    Image:      busybox:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 5; exit 1
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type     Reason                Age    From            Message
  ----     ------                ----   ----            -------
  Normal   SuccessfulCreate      3m16s  job-controller  Created pod: abnormal-end--1-vcvd8
  Normal   SuccessfulCreate      3m7s   job-controller  Created pod: abnormal-end--1-d6zch
  Normal   SuccessfulCreate      2m57s  job-controller  Created pod: abnormal-end--1-hxlcm
  Normal   SuccessfulCreate      2m37s  job-controller  Created pod: abnormal-end--1-l98lm
  Warning  BackoffLimitExceeded  117s   job-controller  Job has reached the specified backoff limit
```

## 비 정상종료 (Multi Container)

job-container-failed.yml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: two-containers
spec:
  template:
    spec:
      containers:
      - name: busybox1
        image: busybox:1
        command: ["sh", "-c", "sleep 5; exit 0"]
      - name: busybox2
        image: busybox:1
        command: ["sh", "-c", "sleep 5; exit 1"]
      restartPolicy: Never
  backoffLimit: 2
```

```bash
$ kubectl apply -f job-container-failed.yml
job.batch/two-containers created

$ kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
two-containers   0/1           56s        56s

$ kubectl describe jobs two-containers
Name:             two-containers
Namespace:        default
Selector:         controller-uid=372c8a2d-6da9-425a-a407-f78e809c8e37
Labels:           controller-uid=372c8a2d-6da9-425a-a407-f78e809c8e37
                  job-name=two-containers
Annotations:      <none>
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Fri, 10 Dec 2021 14:42:02 +0900
Pods Statuses:    0 Running / 0 Succeeded / 3 Failed
Pod Template:
  Labels:  controller-uid=372c8a2d-6da9-425a-a407-f78e809c8e37
           job-name=two-containers
  Containers:
   busybox1:
    Image:      busybox:1
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 5; exit 0
    Environment:  <none>
    Mounts:       <none>
   busybox2:
    Image:      busybox:1
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      sleep 5; exit 1
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type     Reason                Age   From            Message
  ----     ------                ----  ----            -------
  Normal   SuccessfulCreate      118s  job-controller  Created pod: two-containers--1-4q66q
  Normal   SuccessfulCreate      108s  job-controller  Created pod: two-containers--1-wrlfz
  Normal   SuccessfulCreate      98s   job-controller  Created pod: two-containers--1-wpdmq
  Warning  BackoffLimitExceeded  78s   job-controller  Job has reached the specified backoff limit
```

## 크론잡

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
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
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```bash
$ kubectl apply -f cron-job.yml
cronjob.batch/hello created


$ kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          20s

$ kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        37s             3m7s

$ kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-27318590   1/1           3s         2m48s
hello-27318591   1/1           4s         108s
hello-27318592   1/1           4s         48s
```