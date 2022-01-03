# Kubernetes Job
Job Controller는 Pod에 있는 모든 컨테이너가 정상적으로 종료할 때까지 재실행한다.

## 잡의 실행수

[ job-normal-end.yml ]
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
# kubectl apply -f job-normal-end.yml
job.batch/normal-end created

# kubectl get job
NAME         COMPLETIONS   DURATION   AGE
normal-end   0/6           7s         7s

// parallelism을 설정 안한 경우
# kubectl describe job normal-end
Name:             normal-end
Namespace:        default
Selector:         controller-uid=32104740-2124-4545-84f4-12f267a2fea3
Labels:           controller-uid=32104740-2124-4545-84f4-12f267a2fea3
                  job-name=normal-end
Annotations:      <none>
Parallelism:      1
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

// parallelism을 2로 설정한 경우
# kubectl describe job normal-end
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
Age를 보면 Parallelism에 의해 잡이 어떻게 실행이 되는지를 확인 할수 있다.

## Single Container로 구성된 Pod가 이상 종료되는 경우

[ job-abnormal-end.yml ]
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
# kubectl apply -f job-abnormal-end.yml
job.batch/abnormal-end created

# kubectl get jobs
NAME           COMPLETIONS   DURATION   AGE
abnormal-end   0/1           3m42s      3m42s

# kubectl get all
NAME                     READY   STATUS   RESTARTS   AGE
pod/abnormal-end-2cbl8   0/1     Error    0          3m18s
pod/abnormal-end-qjsqv   0/1     Error    0          3m48s
pod/abnormal-end-tmcsf   0/1     Error    0          3m58s

NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)   AGE
service/kubernetes         ClusterIP      10.96.0.1    <none>           443/TCP   6d20h

NAME                     COMPLETIONS   DURATION   AGE
job.batch/abnormal-end   0/1           3m58s      3m58s

# kubectl describe job abnormal-end
Name:           abnormal-end
Namespace:      default
Selector:       controller-uid=7f865d95-5168-46c8-9f28-bfd4c150cb0c
Labels:         controller-uid=7f865d95-5168-46c8-9f28-bfd4c150cb0c
                job-name=abnormal-end
Annotations:    Parallelism:  1
Completions:    1
Start Time:     Tue, 21 Dec 2021 11:17:06 +0900
Pods Statuses:  0 Running / 0 Succeeded / 3 Failed
Pod Template:
  Labels:  controller-uid=7f865d95-5168-46c8-9f28-bfd4c150cb0c
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
  Normal   SuccessfulCreate      4m27s  job-controller  Created pod: abnormal-end-tmcsf
  Normal   SuccessfulCreate      4m17s  job-controller  Created pod: abnormal-end-qjsqv
  Normal   SuccessfulCreate      3m47s  job-controller  Created pod: abnormal-end-2cbl8
  Warning  BackoffLimitExceeded  3m6s   job-controller  Job has reached the specified backoff limit
```
`exit 1`으로 이상 종료가 되도록 처리가 되었고 관련 Pod는 Error가 발생한 것을 확인할 수 있다.

## Multi Container중 일부가 이상 종료되는 경우

[ job-container-failed.yml ]
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
# kubectl apply -f job-container-failed.yml
job.batch/two-containers created

# kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
two-containers   0/1           4m45s      4m45s

job.batch/two-containers   0/1           82s        82s

# kubectl get all
NAME                       READY   STATUS      RESTARTS   AGE
pod/two-containers-4nfw6   0/2     Completed   0          4m24s
pod/two-containers-b5hsh   0/2     Completed   0          3m50s
pod/two-containers-mrsc9   0/2     Completed   0          2m55s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d20h

NAME                       COMPLETIONS   DURATION   AGE
job.batch/two-containers   0/1           4m24s      4m24s

# kubectl describe jobs two-containers
NAME             COMPLETIONS   DURATION   AGE
two-containers   0/1           4m45s      4m45s

# kubectl describe jobs two-containers
Name:           two-containers
Namespace:      default
Selector:       controller-uid=db5d4553-c19d-4fb0-b77b-e0c16fd64220
Labels:         controller-uid=db5d4553-c19d-4fb0-b77b-e0c16fd64220
                job-name=two-containers
Annotations:    Parallelism:  1
Completions:    1
Start Time:     Tue, 21 Dec 2021 11:24:14 +0900
Pods Statuses:  0 Running / 0 Succeeded / 3 Failed
Pod Template:
  Labels:  controller-uid=db5d4553-c19d-4fb0-b77b-e0c16fd64220
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
  Type     Reason                Age    From            Message
  ----     ------                ----   ----            -------
  Normal   SuccessfulCreate      5m38s  job-controller  Created pod: two-containers-4nfw6
  Normal   SuccessfulCreate      5m4s   job-controller  Created pod: two-containers-b5hsh
  Normal   SuccessfulCreate      4m9s   job-controller  Created pod: two-containers-mrsc9
  Warning  BackoffLimitExceeded  3m49s  job-controller  Job has reached the specified backoff limit
```
Pod안에 2개의 Container가 존재하는데 하나는 정상 종료가 되고 다른 하나는 비정상 종료된다. Kubernetes는 두개의 Container 중에 무엇이 나중에 종료되었는지를 확인해서 Pod의 상태를 Complete로 할지, Error로 할지를 결정한다. 

## 소수 계산 Container

[ prime_numpy.py ]
```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import numpy as np
import math
np.set_printoptions(threshold='nan')


# 소수 판정 함수 
def is_prime(n):
    if n % 2 == 0 and n > 2:
        return False
    return all(n % i for i in range(3, int(math.sqrt(n)) + 1, 2))


# 배열에 1부터 차례대로 숫자를 배치
nstart = eval(os.environ.get("A_START_NUM"))
nsize  = eval(os.environ.get("A_SIZE_NUM"))
nend   = nstart + nsize
ay     = np.arange(nstart, nend)

# 소수 판정 함수를 벡터화
pvec = np.vectorize(is_prime)

# 배열 요소에 적용
primes_tf = pvec(ay)

# 소수만 추출하여 표시
primes = np.extract(primes_tf, ay)
print primes
```

[ requirements.txt ]
```
numpy==1.14.1
```

[ Dockerfile ]
```dockerfile
FROM python:2
COPY ./requirements.txt /requirements.txt
COPY ./prime_numpy.py /prime_numpy.py
RUN pip install --no-cache-dir -r /requirements.txt
CMD [ "python", "/prime_numpy.py" ]
```

[ pn_job.yml ]
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: prime-number
spec:
  template:
    spec:
      containers:
      - name: pn-generator
        image: maho/pn_generator:0.1
        env:
        - name: A_START_NUM
          value: "2"
        - name: A_SIZE_NUM
          value: "10**5"
      restartPolicy: Never
  backoffLimit: 4
```

```bash
# docker build -t pn_generator .

# docker login

# docker tag pn_generator:latest welovefish/pn_generator:0.1

# docker push welovefish/pn_generator:0.1

# kubectl apply -f pn_job.yml 

# kubectl get job
NAME           COMPLETIONS   DURATION   AGE
prime-number   1/1           84s        2m35s

# kubectl get pod
NAME                 READY   STATUS      RESTARTS   AGE
prime-number-wmm2q   0/1     Completed   0          2m38s

# kubectl logs prime-number-wmm2q
[    2     3     5     7    11    13    17    19    23    29    31    37
// 중간 생략
 99707 99709 99713 99719 99721 99733 99761 99767 99787 99793 99809 99817
 99823 99829 99833 99839 99859 99871 99877 99881 99901 99907 99923 99929
 99961 99971 99989 99991]
```
소수를 생성하는 Container를 만들어서 

## Message Broker와 조합

[ prime_numpy.py ]
```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
import os
import numpy as np
import math
np.set_printoptions(threshold='nan')

# 소수 판정 함수
def is_prime(n):
    if n % 2 == 0 and n > 2:
        return False
    return all(n % i for i in range(3, int(math.sqrt(n)) + 1, 2))

# 소수 생성 함수 
def prime_number_generater(nstart, nsize):
    nend = nstart + nsize
    ay   = np.arange(nstart, nend)
    # 소수 판정 함수 벡터화
    pvec = np.vectorize(is_prime)
    # 배열 요소에 적용
    primes_t = pvec(ay)
    # 소수만 추출하여 표시
    primes = np.extract(primes_t, ay)
    return primes

if __name__ == '__main__':
    p = sys.stdin.read().split(",")
    print p
    print prime_number_generater(int(p[0]),int(p[1]))
```

[ requirements.txt ]
```
numpy==1.14.1
```

[ Dockerfile ]
```dockerfile
FROM ubuntu:16.04
RUN apt-get update && \
    apt-get install -y curl ca-certificates amqp-tools python python-pip

COPY ./requirements.txt requirements.txt
COPY ./prime_numpy.py /prime_numpy.py
RUN pip install --no-cache-dir -r /requirements.txt

CMD  /usr/bin/amqp-consume --url=$BROKER_URL -q $QUEUE -c 1 /prime_numpy.py
```

[ taskQueue-deploy.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: taskqueue
spec:
  selector:
    matchLabels:
      app: taskQueue
  replicas: 1
  template:
    metadata:
      labels:
        app: taskQueue
    spec:
      containers:
      - image: rabbitmq
        name: rabbitmq
        ports:
        - containerPort: 5672
        resources:
          limits:
            cpu: 100m
---
apiVersion: v1
kind: Service
metadata:
  name: taskqueue
spec:
  type: NodePort
  ports:
  - port: 5672
    nodePort: 31672
  selector:
    app: taskQueue
```

[ job-initiator/Dockerfile ]
```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y iputils-ping dnsutils curl apt-transport-https gnupg2

# install kubectl
RUN curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN touch /etc/apt/sources.list.d/kubernetes.list 
RUN echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
RUN apt-get update && apt-get install -y kubectl

# pyhon
RUN apt-get install -y python python-pip
RUN pip install pika 
RUN pip install kubernetes
```

[ job-initiator/py/job-initiator.py ]
```py
#!/usr/bin/env python
# -*- coding:utf-8 -*-

#from os import path
import yaml
import pika
from kubernetes import client, config

OBJECT_NAME = "pngen"
qname = 'taskqueue'

# 메시지 브로커 접속
def create_queue():
    qmgr_cred= pika.PlainCredentials('guest', 'guest')
    qmgr_host='192.168.1.103'  # Pod가 실행되고 있는 노드의 IP를 지정해 준다.
    qmgr_port='31672'
    qmgr_pram = pika.ConnectionParameters(
              host=qmgr_host,
              port=qmgr_port,
              credentials=qmgr_cred)
    conn = pika.BlockingConnection(qmgr_pram)
    chnl = conn.channel()
    chnl.queue_declare(queue=qname)
    return chnl

# 잡 매니페스트 작성
def create_job_manifest(n_job, n_node):
    container = client.V1Container(
        name="pn-generator",
        image="maho/pn_generator:0.7",
        env=[
            client.V1EnvVar(name="BROKER_URL",value="amqp://guest:guest@taskqueue:5672"),
            client.V1EnvVar(name="QUEUE",value="taskqueue")
        ]
    )
    template = client.V1PodTemplateSpec(
        spec=client.V1PodSpec(containers=[container],
                              restart_policy="Never"                              
        ))
    spec = client.V1JobSpec(
        backoff_limit=4,
        template=template,
        completions=n_job,
        parallelism=n_node)
    job = client.V1Job(
        api_version="batch/v1",
        kind="Job",
        metadata=client.V1ObjectMeta(name=OBJECT_NAME),
        spec=spec)
    return job


if __name__ == '__main__':

    # 소수 계산 분할 파라미터
    job_parms = [[1,1000],[1001,2000],[2001,2000],[3001,4000]]
    jobs  = len(job_parms)
    nodes = 2

    # 큐에 삽입
    queue = create_queue()
    for param_n in job_parms:
        param = str(param_n).replace('[','').replace(']','')
        queue.basic_publish(exchange='',routing_key=qname,body=param)

    # kubectl의.kube config를 읽어 k8s마스터에 잡 요청 전송
    config.load_kube_config()
    client.BatchV1Api().create_namespaced_job(
        body=create_job_manifest(jobs,nodes),namespace="default")
```

```bash
# docker build --tag pn_generator:0.2 .

# docker tag pn_generator:0.2 welovefish/pn_generator:0.2

# docker push welovefish/pn_generator:0.2

# kubectl apply -f taskQueue-deploy.yml 
deployment.apps/taskqueue created
service/taskqueue created

# kubectl get all -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
pod/taskqueue-5f6557cd4f-mv67s   1/1     Running   0          13m   172.16.132.25   w3-k8s   <none>           <none>

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          6d22h   <none>
service/taskqueue    NodePort    10.108.93.42   <none>        5672:31672/TCP   13m     app=taskQueue

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES     SELECTOR
deployment.apps/taskqueue   1/1     1            1           13m   rabbitmq     rabbitmq   app=taskQueue

NAME                                   DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES     SELECTOR
replicaset.apps/taskqueue-5f6557cd4f   1         1         1       13m   rabbitmq     rabbitmq   app=taskQueue,pod-template-hash=5f6557cd4f

# kubectl get node -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
m-k8s    Ready    master   6d22h   v1.18.4   192.168.1.10    <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w1-k8s   Ready    <none>   6d22h   v1.18.4   192.168.1.101   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w2-k8s   Ready    <none>   6d22h   v1.18.4   192.168.1.102   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w3-k8s   Ready    <none>   6d22h   v1.18.4   192.168.1.103   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1

// job-initiator.py의 qmgr_host='192.168.1.103' 부분을 상황에 맞게 변경해 준다. (w3-k8s의 IP를 지정)

# docker build --tag job-init:0.1 .

# docker run -it --rm --name kube -v `pwd`/py:/py -v ~/.kube:/root/.kube -v ~/.minikube:/root/.minikube job-init:0.1 bash 

root@b5198581f38a:/# kubectl get node
NAME     STATUS   ROLES    AGE     VERSION
m-k8s    Ready    master   6d22h   v1.18.4
w1-k8s   Ready    <none>   6d22h   v1.18.4
w2-k8s   Ready    <none>   6d22h   v1.18.4
w3-k8s   Ready    <none>   6d22h   v1.18.4

root@b5198581f38a:/# exit

# kubectl get job,pod 
NAME              COMPLETIONS   DURATION   AGE
job.batch/pngen   4/4           59s        9m41s

NAME                             READY   STATUS      RESTARTS   AGE
pod/pngen-8s5fw                  0/1     Completed   0          8m44s
pod/pngen-d69h8                  0/1     Completed   0          9m41s
pod/pngen-t858n                  0/1     Completed   0          8m45s
pod/pngen-wzdw8                  0/1     Completed   0          9m41s
pod/taskqueue-5f6557cd4f-mv67s   1/1     Running     0          36m

# kubectl logs pod/pngen-8s5fw 
['3001', ' 4000']
[3001 3011 3019 3023 3037 3041 3049 3061 3067 3079 3083 3089 3109 3119
 3121 3137 3163 3167 3169 3181 3187 3191 3203 3209 3217 3221 3229 3251
 3253 3257 3259 3271 3299 3301 3307 3313 3319 3323 3329 3331 3343 3347
 3359 3361 3371 3373 3389 3391 3407 3413 3433 3449 3457 3461 3463 3467
 3469 3491 3499 3511 3517 3527 3529 3533 3539 3541 3547 3557 3559 3571
 3581 3583 3593 3607 3613 3617 3623 3631 3637 3643 3659 3671 3673 3677
 3691 3697 3701 3709 3719 3727 3733 3739 3761 3767 3769 3779 3793 3797
 3803 3821 3823 3833 3847 3851 3853 3863 3877 3881 3889 3907 3911 3917
 3919 3923 3929 3931 3943 3947 3967 3989 4001 4003 4007 4013 4019 4021
 4027 4049 4051 4057 4073 4079 4091 4093 4099 4111 4127 4129 4133 4139
 4153 4157 4159 4177 4201 4211 4217 4219 4229 4231 4241 4243 4253 4259
 4261 4271 4273 4283 4289 4297 4327 4337 4339 4349 4357 4363 4373 4391
 4397 4409 4421 4423 4441 4447 4451 4457 4463 4481 4483 4493 4507 4513
 4517 4519 4523 4547 4549 4561 4567 4583 4591 4597 4603 4621 4637 4639
 4643 4649 4651 4657 4663 4673 4679 4691 4703 4721 4723 4729 4733 4751
 4759 4783 4787 4789 4793 4799 4801 4813 4817 4831 4861 4871 4877 4889
 4903 4909 4919 4931 4933 4937 4943 4951 4957 4967 4969 4973 4987 4993
 4999 5003 5009 5011 5021 5023 5039 5051 5059 5077 5081 5087 5099 5101
 5107 5113 5119 5147 5153 5167 5171 5179 5189 5197 5209 5227 5231 5233
 5237 5261 5273 5279 5281 5297 5303 5309 5323 5333 5347 5351 5381 5387
 5393 5399 5407 5413 5417 5419 5431 5437 5441 5443 5449 5471 5477 5479
 5483 5501 5503 5507 5519 5521 5527 5531 5557 5563 5569 5573 5581 5591
 5623 5639 5641 5647 5651 5653 5657 5659 5669 5683 5689 5693 5701 5711
 5717 5737 5741 5743 5749 5779 5783 5791 5801 5807 5813 5821 5827 5839
 5843 5849 5851 5857 5861 5867 5869 5879 5881 5897 5903 5923 5927 5939
 5953 5981 5987 6007 6011 6029 6037 6043 6047 6053 6067 6073 6079 6089
 6091 6101 6113 6121 6131 6133 6143 6151 6163 6173 6197 6199 6203 6211
 6217 6221 6229 6247 6257 6263 6269 6271 6277 6287 6299 6301 6311 6317
 6323 6329 6337 6343 6353 6359 6361 6367 6373 6379 6389 6397 6421 6427
 6449 6451 6469 6473 6481 6491 6521 6529 6547 6551 6553 6563 6569 6571
 6577 6581 6599 6607 6619 6637 6653 6659 6661 6673 6679 6689 6691 6701
 6703 6709 6719 6733 6737 6761 6763 6779 6781 6791 6793 6803 6823 6827
 6829 6833 6841 6857 6863 6869 6871 6883 6899 6907 6911 6917 6947 6949
 6959 6961 6967 6971 6977 6983 6991 6997]

# kubectl logs pod/pngen-d69h8 
['1', ' 1000']
[  1   2   3   5   7  11  13  17  19  23  29  31  37  41  43  47  53  59
  61  67  71  73  79  83  89  97 101 103 107 109 113 127 131 137 139 149
 151 157 163 167 173 179 181 191 193 197 199 211 223 227 229 233 239 241
 251 257 263 269 271 277 281 283 293 307 311 313 317 331 337 347 349 353
 359 367 373 379 383 389 397 401 409 419 421 431 433 439 443 449 457 461
 463 467 479 487 491 499 503 509 521 523 541 547 557 563 569 571 577 587
 593 599 601 607 613 617 619 631 641 643 647 653 659 661 673 677 683 691
 701 709 719 727 733 739 743 751 757 761 769 773 787 797 809 811 821 823
 827 829 839 853 857 859 863 877 881 883 887 907 911 919 929 937 941 947
 953 967 971 977 983 991 997]

# kubectl delete job pngen
job.batch "pngen" deleted
```
사용하는 파일도 많고 내용도 좀 복잡하다. 먼저 메세지큐를 이해하고 접근하는 것이 좀 더 수월하지 않을까 싶다.

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
# kubectl apply -f cron-job.yml
cronjob.batch/hello created


# kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          20s

# kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        37s             3m7s

# kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-27318590   1/1           3s         2m48s
hello-27318591   1/1           4s         108s
hello-27318592   1/1           4s         48s
```