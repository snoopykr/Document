# Kubenetes Deployment

## 디플로이먼트 생성

deployment1.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:                     # deployment와 pod의 매칭
    matchLabels:
      app: web
  template:                     # pod 템플릿
    metadata:
      labels:
        app: web                # pod 라벨
    spec:
      containers:               # 컨테이너 사양
      - name: nginx
        image: nginx:1.16
```

```bash
$ kubectl apply -f deployment1.yml
deployment.apps/web-deploy created

$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   2/3     3            2           17s

$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
web-deploy-6bc4dfc596   3         3         3       27s

$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-6bc4dfc596-2gwt2   1/1     Running   0          36s
web-deploy-6bc4dfc596-m8vqh   1/1     Running   0          36s
web-deploy-6bc4dfc596-n26qc   1/1     Running   0          36s


$ kubectl get pod -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
web-deploy-6bc4dfc596-2gwt2   1/1     Running   0          2m52s   172.17.0.3   minikube   <none>           <none>
web-deploy-6bc4dfc596-m8vqh   1/1     Running   0          2m52s   172.17.0.5   minikube   <none>           <none>
web-deploy-6bc4dfc596-n26qc   1/1     Running   0          2m52s   172.17.0.4   minikube   <none>           <none>
```

## 스케일

deployment2.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 10                  # 3 -> 10 변경
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
```

```bash
$ kubectl apply -f deployment2.yml
deployment.apps/web-deploy configured

$ kubectl get pod -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS        AGE
default       web-deploy-6bc4dfc596-2gwt2        1/1     Running   0               8m29s
default       web-deploy-6bc4dfc596-56v8b        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-6hhcs        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-6ttvx        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-72v7g        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-7jq2s        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-cbtts        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-fsc2h        1/1     Running   0               2m1s
default       web-deploy-6bc4dfc596-m8vqh        1/1     Running   0               8m29s
default       web-deploy-6bc4dfc596-n26qc        1/1     Running   0               8m29s

$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   10/10   10           10          10m


$ kubectl get po,deploy
NAME                              READY   STATUS    RESTARTS   AGE
pod/web-deploy-6bc4dfc596-2gwt2   1/1     Running   0          11m
pod/web-deploy-6bc4dfc596-56v8b   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-6hhcs   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-6ttvx   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-72v7g   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-7jq2s   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-cbtts   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-fsc2h   1/1     Running   0          5m8s
pod/web-deploy-6bc4dfc596-m8vqh   1/1     Running   0          11m
pod/web-deploy-6bc4dfc596-n26qc   1/1     Running   0          11m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-deploy   10/10   10           10          11m

$ kubectl scale --replicas=5 deployment.apps/web-deploy
deployment.apps/web-deploy scaled

$ kubectl get po,deploy
NAME                              READY   STATUS    RESTARTS   AGE
pod/web-deploy-6bc4dfc596-2gwt2   1/1     Running   0          12m
pod/web-deploy-6bc4dfc596-cbtts   1/1     Running   0          5m44s
pod/web-deploy-6bc4dfc596-fsc2h   1/1     Running   0          5m44s
pod/web-deploy-6bc4dfc596-m8vqh   1/1     Running   0          12m
pod/web-deploy-6bc4dfc596-n26qc   1/1     Running   0          12m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-deploy   5/5     5            5           12m
```

## 롤아웃

deployment3.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 10
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.17   # 1.16 -> 1.17 변경
```

```bash
$ kubectl describe deploy web-deploy
Name:                   web-deploy
Namespace:              default
CreationTimestamp:      Wed, 08 Dec 2021 18:11:35 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx:
    Image:        nginx:1.16
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-deploy-6bc4dfc596 (10/10 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 5
  Normal  ScalingReplicaSet  84s   deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  68s   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10

$ kubectl apply -f deployment3.yml
deployment.apps/web-deploy configured

$ kubectl get po
NAME                          READY   STATUS              RESTARTS       AGE
web-deploy-5899d78c9-6mv85    1/1     Running             0              8s
web-deploy-5899d78c9-847d8    1/1     Running             0              23s
web-deploy-5899d78c9-8tpp5    1/1     Running             0              5s
web-deploy-5899d78c9-bv7bp    1/1     Running             0              6s
web-deploy-5899d78c9-d9frg    0/1     ContainerCreating   0              4s
web-deploy-5899d78c9-f27zj    1/1     Running             0              23s
web-deploy-5899d78c9-gpp22    0/1     ContainerCreating   0              23s
web-deploy-5899d78c9-l2lg5    0/1     ContainerCreating   0              4s
web-deploy-5899d78c9-trflk    1/1     Running             0              23s
web-deploy-5899d78c9-v4hz4    0/1     ContainerCreating   0              23s
web-deploy-6bc4dfc596-2gwt2   1/1     Running             1 (3m6s ago)   15h
web-deploy-6bc4dfc596-42qrw   1/1     Terminating         0              116s
web-deploy-6bc4dfc596-7g6w6   1/1     Terminating         0              116s
web-deploy-6bc4dfc596-m8vqh   1/1     Running             1 (3m6s ago)   15h

$ kubectl describe deploy web-deploy
Name:                   web-deploy
Namespace:              default
CreationTimestamp:      Wed, 08 Dec 2021 18:11:35 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=web
Replicas:               10 desired | 5 updated | 13 total | 8 available | 5 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx:
    Image:        nginx:1.17                # 1.16 -> 1.17 변경
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  web-deploy-6bc4dfc596 (8/8 replicas created)
NewReplicaSet:   web-deploy-5899d78c9 (5/5 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  15h   deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 5
  Normal  ScalingReplicaSet  113s  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  97s   deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  4s    deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 3
  Normal  ScalingReplicaSet  4s    deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 8
  Normal  ScalingReplicaSet  4s    deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 5
```

## 롤백

```bash
$ kubectl rollout  undo deployment web-deploy
deployment.apps/web-deploy rolled back

$ kubectl describe deploy web-deploy
Name:                   web-deploy
Namespace:              default
CreationTimestamp:      Wed, 08 Dec 2021 18:11:35 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=web
Replicas:               10 desired | 8 updated | 13 total | 8 available | 5 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   nginx:
    Image:        nginx:1.16                # 1.17 -> 1.16 원복...
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  web-deploy-5899d78c9 (5/5 replicas created)
NewReplicaSet:   web-deploy-6bc4dfc596 (8/8 replicas created)
Events:
  Type    Reason             Age                    From                   Message
  ----    ------             ----                   ----                   -------
  Normal  ScalingReplicaSet  15h                    deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  15h                    deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  15h                    deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 5
  Normal  ScalingReplicaSet  7m13s                  deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  5m40s                  deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 5
  Normal  ScalingReplicaSet  5m40s                  deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 3
  Normal  ScalingReplicaSet  5m40s                  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 8
  Normal  ScalingReplicaSet  5m26s                  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 7
  Normal  ScalingReplicaSet  5m25s                  deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 6
  Normal  ScalingReplicaSet  5m23s                  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 6
  Normal  ScalingReplicaSet  5m23s                  deployment-controller  Scaled up replica set web-deploy-5899d78c9 to 7
  Normal  ScalingReplicaSet  5m21s (x2 over 7m29s)  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  0s (x16 over 5m22s)    deployment-controller  (combined from similar events): Scaled down replica set web-deploy-5899d78c9 to 5

$ kubectl get po
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-6bc4dfc596-9v758   1/1     Running   0          31s
web-deploy-6bc4dfc596-dc95w   1/1     Running   0          36s
web-deploy-6bc4dfc596-hltp9   1/1     Running   0          33s
web-deploy-6bc4dfc596-jl57v   1/1     Running   0          36s
web-deploy-6bc4dfc596-q4mzd   1/1     Running   0          36s
web-deploy-6bc4dfc596-qrkgq   1/1     Running   0          36s
web-deploy-6bc4dfc596-s86cb   1/1     Running   0          36s
web-deploy-6bc4dfc596-vrcj6   1/1     Running   0          30s
web-deploy-6bc4dfc596-z87bk   1/1     Running   0          32s
web-deploy-6bc4dfc596-zx98f   1/1     Running   0          30s
```

## 파드 IP의 변경

```bash
$ kubectl get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
web-deploy-6bc4dfc596-9v758   1/1     Running   0          4m19s   172.17.0.10   minikube   <none>           <none>
web-deploy-6bc4dfc596-dc95w   1/1     Running   0          4m24s   172.17.0.12   minikube   <none>           <none>
web-deploy-6bc4dfc596-hltp9   1/1     Running   0          4m21s   172.17.0.14   minikube   <none>           <none>
web-deploy-6bc4dfc596-jl57v   1/1     Running   0          4m24s   172.17.0.9    minikube   <none>           <none>
web-deploy-6bc4dfc596-q4mzd   1/1     Running   0          4m24s   172.17.0.6    minikube   <none>           <none>
web-deploy-6bc4dfc596-qrkgq   1/1     Running   0          4m24s   172.17.0.5    minikube   <none>           <none>
web-deploy-6bc4dfc596-s86cb   1/1     Running   0          4m24s   172.17.0.7    minikube   <none>           <none>
web-deploy-6bc4dfc596-vrcj6   1/1     Running   0          4m18s   172.17.0.8    minikube   <none>           <none>
web-deploy-6bc4dfc596-z87bk   1/1     Running   0          4m20s   172.17.0.15   minikube   <none>           <none>
web-deploy-6bc4dfc596-zx98f   1/1     Running   0          4m18s   172.17.0.3    minikube   <none>           <none>


$ kubectl delete po web-deploy-6bc4dfc596-9v758
pod "web-deploy-6bc4dfc596-9v758" deleted

$ kubectl get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
web-deploy-6bc4dfc596-dc95w   1/1     Running   0          6m4s    172.17.0.12   minikube   <none>           <none>
web-deploy-6bc4dfc596-hltp9   1/1     Running   0          6m1s    172.17.0.14   minikube   <none>           <none>
web-deploy-6bc4dfc596-jl57v   1/1     Running   0          6m4s    172.17.0.9    minikube   <none>           <none>
web-deploy-6bc4dfc596-kmfxz   1/1     Running   0          4s      172.17.0.4    minikube   <none>           <none>             # 새로시작 ip변경
web-deploy-6bc4dfc596-q4mzd   1/1     Running   0          6m4s    172.17.0.6    minikube   <none>           <none>
web-deploy-6bc4dfc596-qrkgq   1/1     Running   0          6m4s    172.17.0.5    minikube   <none>           <none>
web-deploy-6bc4dfc596-s86cb   1/1     Running   0          6m4s    172.17.0.7    minikube   <none>           <none>
web-deploy-6bc4dfc596-vrcj6   1/1     Running   0          5m58s   172.17.0.8    minikube   <none>           <none>
web-deploy-6bc4dfc596-z87bk   1/1     Running   0          6m      172.17.0.15   minikube   <none>           <none>
web-deploy-6bc4dfc596-zx98f   1/1     Running   0          5m58s   172.17.0.3    minikube   <none>           <none>
```

## 자동복구

pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test1
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh",  "-c", "sleep 3600; exit 0"]
  restartPolicy: Always
```  

deployment4.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test2
spec:
  replicas: 4
  selector:
    matchLabels:
      app: test2
  template:
    metadata:
      labels:
        app: test2
    spec:
      containers:
      - name: busybox
        image: busybox:1
        command: ["sh",  "-c", "sleep 3600; exit 0"]	
```

```bash
$ kubectl get node
NAME           STATUS   ROLES                  AGE    VERSION
minikube       Ready    control-plane,master   2m7s   v1.22.3
minikube-m02   Ready    <none>                 92s    v1.22.3
minikube-m03   Ready    <none>                 56s    v1.22.3
minikube-m04   Ready    <none>                 22s    v1.22.3

$ kubectl apply -f pod.yml
pod/test1 created

$ kubectl apply -f deployment4.yml
deployment.apps/test2 created

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
test1                    1/1     Running   0          77s   10.244.3.2   minikube-m04   <none>           <none>
test2-5db466bbc6-4h7bb   1/1     Running   0          64s   10.244.2.2   minikube-m03   <none>           <none>
test2-5db466bbc6-hnh4n   1/1     Running   0          64s   10.244.1.3   minikube-m02   <none>           <none>
test2-5db466bbc6-pd68b   1/1     Running   0          64s   10.244.3.3   minikube-m04   <none>           <none>
test2-5db466bbc6-x5tdb   1/1     Running   0          64s   10.244.2.3   minikube-m03   <none>           <none>

$ minikube node delete minikube-m04

$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
test2-5db466bbc6-4h7bb   1/1     Running   0          3m54s   10.244.2.2   minikube-m03   <none>           <none>
test2-5db466bbc6-hnh4n   1/1     Running   0          3m54s   10.244.1.3   minikube-m02   <none>           <none>
test2-5db466bbc6-kkcvg   1/1     Running   0          23s     10.244.2.4   minikube-m03   <none>           <none>               # 4 -> 3 삭제후 재실행
test2-5db466bbc6-x5tdb   1/1     Running   0          3m54s   10.244.2.3   minikube-m03   <none>           <none>
```

## 고가용성

mysql-act-stby.yml

```yaml
# 퍼시스턴트 볼륨
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gvol-1
spec:
  storageClassName: gluster-heketi
  #storageClassName: ibmc-file-bronze   # 용량 20Gi IKS 
  accessModes:
   - ReadWriteMany
  resources:
    requests:
      storage: 12Gi
      #storage: 20Gi   # for IKS
---
# MySQL서버 디플로이먼트
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: qwerty
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: pvc
        livenessProbe:
          exec:
            command: ["mysqladmin","-p$MYSQL_ROOT_PASSWORD","ping"]
          initialDelaySeconds: 60
          timeoutSeconds: 10
      volumes:
      - name: pvc
        persistentVolumeClaim:
          claimName: gvol-1
---
# MySQL서버 서비스
apiVersion: v1
kind: Service
metadata:
  name: mysql-dpl
  labels:
    app: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    nodePort: 30306
  selector:
    app: mysql
```

```bash
$ minikube node list
minikube        192.168.49.2
minikube-m02    192.168.49.3
minikube-m03    192.168.49.4
