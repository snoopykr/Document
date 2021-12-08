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
$ kubectl describe deployment web-deploy
Name:                   web-deploy
Namespace:              default
CreationTimestamp:      Wed, 08 Dec 2021 18:11:35 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
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
NewReplicaSet:   web-deploy-6bc4dfc596 (5/5 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  19m    deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 3
  Normal  ScalingReplicaSet  12m    deployment-controller  Scaled up replica set web-deploy-6bc4dfc596 to 10
  Normal  ScalingReplicaSet  6m57s  deployment-controller  Scaled down replica set web-deploy-6bc4dfc596 to 5
```