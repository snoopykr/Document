# Kubernetes run

kubenetes가 버전업하면서 run명령에도 변화가 생겼다.

기존에는 pod를 생성하면서 deployment와 replicaset를 자동으로 생성해 주었는데 현재(v1.22)는 pod만 생성되고 있다. 

그리고 run명령어 사용을 자제하는 듯하다...

## kubectl run

```bash
# kubectl run hello-world --image=hello-world -it --restart=Never --rm

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
 # docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
단순하게 1개의 pod를 만드는 단순한 명령이다.

| 옵션 | 설명 |
|---|---|
| `kubectl`	| k8s 클러스터를 조작하기 위해 사용되는 커맨드 |
| `run` | 컨테이너 실행을 명령하는 서브 커맨드 |
| `hello-world` | 쿠버네티스 오브젝트의 이름(파드나 컨트롤러 등) |
| `--image=hello-world` | 컨테이너의 이미지, 파드 단위로 컨테이너 기동, 리포지터리명이 없는 경우 도커허브 사용 |
| `-it` | 도커 -it와 동일, `--restart=Never`인 경우에만 유효 그외 백그라운드 실행 |
| `--restart=Never` | Never는 직접 파드가 기동, Always나 OnFailure는 컨트롤러 통해 기동  |
| `--rm` | 파드 종료 후 자동 삭제 |

## run vs create
run은 단 1개의 pod만을 생성하지만 create는 deployment를 만들고 그 안에 pod를 생성한다.

```bash
[root@m-k8s 3.1.6]# kubectl run nginx-pod --image=nginx
pod/nginx-pod created

[root@m-k8s 3.1.6]# kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          10s

[root@m-k8s 3.1.6]# kubectl create nginx --image=nginx
Error: unknown flag: --image
See 'kubectl create --help' for usage.

[root@m-k8s 3.1.6]# kubectl create deployment dpy-nginx --image=nginx
deployment.apps/dpy-nginx created

[root@m-k8s 3.1.6]# kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
dpy-nginx-c8d778df-mxj56   1/1     Running   0          7s
nginx-pod                  1/1     Running   0          79s

[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
dpy-nginx-c8d778df-mxj56   1/1     Running   0          63s     172.16.132.6     w3-k8s   <none>           <none>
nginx-pod                  1/1     Running   0          2m15s   172.16.221.131   w1-k8s   <none>           <none>

[root@m-k8s 3.1.6]# kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/dpy-nginx-c8d778df-mxj56   1/1     Running   0          6m56s
pod/nginx-pod                  1/1     Running   0          5m44s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23h

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dpy-nginx   1/1     1            1           6m56s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/dpy-nginx-c8d778df   1         1         1       6m56s
```
deployment-replicaset-pod 순으로 이름이 만들어 진다. `dpy-nginx-c8d778df`, `dpy-nginx-c8d778df-mxj56`

## kubectl run for controller

[ 에러 ]
```bash
# kubectl run hello-world --image=hello-world
pod/hello-world created
```
deployment, replicaset이 생성되어야 하는데 버전업되면서 pod만 생성...

```bash
# kubectl run webserver --image=nginx --replicas=5
Error: unknown flag: --replicas
See 'kubectl run --help' for usage.
```
replicaset도 바로 적용이 되지 않는다.

[ 수정 ]
```bash
# kubectl create deployment --image=nginx webserver
deployment.apps/webserver created

# kubectl scale --replicas=5 deployment/webserver
deployment.apps/webserver scaled

# kubectl get all
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

# kubectl delete po webserver-559b886555-5hdkv webserver-559b886555-6952d
pod "webserver-559b886555-5hdkv" deleted
pod "webserver-559b886555-6952d" deleted

# kubectl get all
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

# kubectl delete deployment webserver
deployment.apps "webserver" deleted

# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   56m
```
pod를 삭제해도 replicaset의 영향으로 새로 생성이 된다. pod를 완전히 제거하기 위해서는 deployment를 삭제해야 한다.

## job 생성
job controller는 pod가 비정상 종료하면 재시작하며 정상 종료할 때까지 지정한 횟수만큼 재실행한다.

[ 에러 ]
```bash
# kubectl run hello-world --image=hello-world --restart=OnFailure
pod/hello-world created

# kubectl get all
NAME              READY   STATUS      RESTARTS   AGE
pod/hello-world   0/1     Completed   0          100s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23h

# kubectl delete pod/hello-world
pod "hello-world" deleted
```
run 명령만으로는 job 생성 안된다.

[ 수정 ]
```bash
# kubectl create job hello-world --image=hello-world --restart=OnFailure
job.batch/hello-world created

# kubectl get all
NAME                       READY   STATUS              RESTARTS   AGE
pod/hello-world--1-l4t5s   0/1     ContainerCreating   0          2s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   60m

NAME                    COMPLETIONS   DURATION   AGE
job.batch/hello-world   0/1           2s         2s
```

[ kubernetes 1.18의 경우 에러가 발생 ]
```
[root@m-k8s 3.1.6]# kubectl create job hello-world --image=hello-world --restart=OnFailure
Error: unknown flag: --restart
See 'kubectl create job --help' for usage.

[root@m-k8s 3.1.6]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.4", GitCommit:"c96aede7b5205121079932896c4ad89bb93260af", GitTreeState:"clean", BuildDate:"2020-06-17T11:41:22Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.20", GitCommit:"1f3e19b7beb1cc0110255668c4238ed63dadb7ad", GitTreeState:"clean", BuildDate:"2021-06-16T12:51:17Z", GoVersion:"go1.13.15", Compiler:"gc", Platform:"linux/amd64"}

[root@m-k8s 3.1.6]# kubectl create job hello-world --image=hello-world -- --restart=OnFailure
job.batch/hello-world created

[root@m-k8s 3.1.6]# kubectl get all
NAME                    READY   STATUS               RESTARTS   AGE
pod/hello-world-bxjlg   0/1     ContainerCannotRun   0          52s
pod/hello-world-kxk95   0/1     ContainerCannotRun   0          62s
pod/hello-world-r2w4w   0/1     ContainerCannotRun   0          32s
pod/hello-world-ztgwk   0/1     ContainerCannotRun   0          67s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   24h

NAME                    COMPLETIONS   DURATION   AGE
job.batch/hello-world   0/1           67s        67s
```
job은 만들었지만 pod가 원활하게 생성되지 않는 것을 알수 있다.

## job의 기능

```bash
# kubectl create job job1 --image=ubuntu -- /bin/bash -c "exit 0" --restart=OnFailure
job.batch/job1 created

# kubectl create job job2 --image=ubuntu -- /bin/bash -c "exit 1" --restart=OnFailure
job.batch/job2 created

# kubectl get all
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

# kubectl get jobs
NAME   COMPLETIONS   DURATION   AGE
job1   1/1           10s        18h
job2   0/1           18h        18h
```
job1과 달리 job2의 경우 `exit 1`로 비정상 종료가 되기 때문에 지속해서 pod를 생성하지만 정상적인 종료가 되지 않는다.

파드 실패 : restartPolicy=Never<br>
컨테이너 오류 : restartPolicy=OnFailure
