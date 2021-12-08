# Kubernetes run
kubenetes가 버전업하면서 run명령에도 변화가 생겼다.

기존에는 pod를 생성하면서 deployment와 replicaset를 자동으로 생성해 주었는데 현재(v1.22)는 pod만 생성되고 있다. 

그리고 run 명령 사용을 자제하는 듯하다...

## kubectl run
```bash
$ kubectl run hello-world --image=hello-world -it --restart=Never

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

| 옵션 | 설명 |
|---|:---:|
| `kubectl`	| k8s 클러스터를 조작하기 위해 사용되는 커맨드 |
| `run` | 컨테이너 실행을 명령하는 서브 커맨드 |
| `hello-world` | 쿠버네티스 오브젝트의 이름(파드나 컨트롤러 등) |
| `--image=hello-world` | 컨테이너의 이미지, 파드 단위로 컨테이너 기동, 리포지터리명이 없는 경우 도커허브 사용 |
| `-it` | 도커 -it와 동일, `--restart=Never`인 경우에만 유효 그외 백그라운드 실행 |
| `--restart=Never` | Never는 직접 파드가 기동, Always나 OnFailure는 컨트롤러 통해 기동  |
| `--rm` | 파드 종료 후 자동 삭제 |

## kubectl run for controller
```bash
$ kubectl run hello-world --image=hello-world
pod/hello-world created
```
deployment, replicaset이 생성되어야 하는데 버전업되면서 pod만 생성...

```bash
$ kubectl run webserver --image=nginx --replicas=5
Error: unknown flag: --replicas
See 'kubectl run --help' for usage.
```
replicaset도 바로 적용이 되지 않는다.

버전업 문제 해결...

```bash
$ kubectl create deployment --image=nginx webserver
deployment.apps/webserver created

$ kubectl scale --replicas=5 deployment/webserver
deployment.apps/webserver scaled

$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/webserver-559b886555-5hdkv   1/1     Running   0          3m57s
pod/webserver-559b886555-6952d   1/1     Running   0          3m57s
pod/webserver-559b886555-8vg4t   1/1     Running   0          3m57s
pod/webserver-559b886555-ct2kk   1/1     Running   0          4m35s
pod/webserver-559b886555-tjl4l   1/1     Running   0          3m57s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   53m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webserver   5/5     5            5           4m35s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/webserver-559b886555   5         5         5       4m35s

$ kubectl delete po webserver-559b886555-5hdkv webserver-559b886555-6952d
pod "webserver-559b886555-5hdkv" deleted
pod "webserver-559b886555-6952d" deleted

$ kubectl get all
NAME                             READY   STATUS              RESTARTS   AGE
pod/webserver-559b886555-7d6r4   0/1     ContainerCreating   0          5s
pod/webserver-559b886555-8vg4t   1/1     Running             0          5m23s
pod/webserver-559b886555-bqhc9   0/1     ContainerCreating   0          5s
pod/webserver-559b886555-ct2kk   1/1     Running             0          6m1s
pod/webserver-559b886555-tjl4l   1/1     Running             0          5m23s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   55m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webserver   3/5     5            3           6m1s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/webserver-559b886555   5         5         3       6m1s

$ kubectl delete deployment webserver
deployment.apps "webserver" deleted

$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   56m
```

## job

```bash
$ kubectl run hello-world --image=hello-world --restart=OnFailure
pod/hello-world created

$ kubectl get all
NAME              READY   STATUS              RESTARTS   AGE
pod/hello-world   0/1     ContainerCreating   0          10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   58m

$ kubectl delete pod/hello-world
pod "hello-world" deleted
```
job 생성 안됨

수정...

```bash
$ kubectl create job hello-world --image=hello-world
job.batch/hello-world created

$ kubectl get all
NAME                       READY   STATUS              RESTARTS   AGE
pod/hello-world--1-l4t5s   0/1     ContainerCreating   0          2s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   60m

NAME                    COMPLETIONS   DURATION   AGE
job.batch/hello-world   0/1           2s         2s
```

## job의 기능

```bash
$ kubectl create job job1 --image=ubuntu -- /bin/bash -c "exit 0" --restart=OnFailure
job.batch/job1 created

$ kubectl create job job2 --image=ubuntu -- /bin/bash -c "exit 1" --restart=OnFailure
job.batch/job2 created

$ kubectl get all
NAME                READY   STATUS              RESTARTS   AGE
pod/job1--1-j576z   0/1     Completed           0          96s
pod/job2--1-fr6n7   0/1     ContainerCreating   0          4s
pod/job2--1-gdgwq   0/1     Error               0          74s
pod/job2--1-lmrhb   0/1     Error               0          64s
pod/job2--1-mfg85   0/1     Error               0          77s
pod/job2--1-n4smm   0/1     Error               0          44s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   78m

NAME             COMPLETIONS   DURATION   AGE
job.batch/job1   1/1           4s         96s
job.batch/job2   0/1           77s        77s
```
파드의 실패(restartPolicy=Never), 컨테이너가 오류(restartPolicy=OnFailure)

job2의 경우 `exit 1`로 비정상 종료가 됨으로 인해 지속적인 pod를 생성하는 것을 확인할 수 있다.