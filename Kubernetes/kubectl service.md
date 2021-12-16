# Kubenetes Service

| 서비스 타입 | 설명 |
|---|---|
| `ClusterIP`	| 타입을 지정하지 않으면 기본으로 설정되며, 클러스터 내부의 파드에서 서비스의 이름으로 접근할 수 있다 |
| `NodePort` | ClusterIP의 접근 범위뿐만 아니라 k8s 클러스터 외부에서도 노드의 IP 주소와 포트번호를 접근할 수 있다. |
| `LoadBalancer` | NodePort의 접근 번위뿐만 아니라 k8s 클러스터 외부에서 대표 IP 주소로 접근할 수 있다. |
| `ExternalName` | k8s 클러스터 내의 파드에서 외부 IP 조소에 서비스의 이름으로 접근할 수 있다. |

## ClusterIP

deploy.yml

```yaml
## 디플로이먼트
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:           # deployment - pod 대응용
    matchLabels:
      app: web
  template:           # 여기서부터 파드 템플릿
    metadata:
      labels:
        app: web      # 파드의 라벨
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

svc.yml

```yaml
## 서비스
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:                 # type을 생략하여 ClusterIP가 적용된다. 
  selector:           # service - 백엔드 pod와 연결
    app: web
  ports:
  - protocol: TCP
    port: 80
```

```bash
$ kubectl apply -f svc.yml
service/web-service created

$ kubectl apply -f deploy.yml
deployment.apps/web-deploy created

$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/web-deploy-86cd4d65b9-65dqb   1/1     Running   0          72s
pod/web-deploy-86cd4d65b9-j82db   1/1     Running   0          72s
pod/web-deploy-86cd4d65b9-nshfc   1/1     Running   0          72s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   25h
service/web-service   ClusterIP   10.111.231.63   <none>        80/TCP    81s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-deploy   3/3     3            3           72s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/web-deploy-86cd4d65b9   3         3         3       72s

$ kubectl run -it bustbox --restart=Never --rm --image=busybox sh
If you don't see a command prompt, try pressing enter.

/ # wget -q -O - http://web-service
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

/ # env | grep WEB_SERVICE
WEB_SERVICE_SERVICE_PORT=80
WEB_SERVICE_PORT=tcp://10.111.231.63:80
WEB_SERVICE_PORT_80_TCP_ADDR=10.111.231.63
WEB_SERVICE_PORT_80_TCP_PORT=80
WEB_SERVICE_PORT_80_TCP_PROTO=tcp
WEB_SERVICE_PORT_80_TCP=tcp://10.111.231.63:80
WEB_SERVICE_SERVICE_HOST=10.111.231.63
/ # exit

$ for pod in $(kubectl get pods | awk 'NR>1 {print $1}' | grep web-deploy); do kubectl exec $pod -- /bin/sh -c "hostname>/usr/share/nginx/html/index.html"; done

$ kubectl run -it bustbox --restart=Never --rm --image=busybox sh
If you don't see a command prompt, try pressing enter.

/ # while true; do wget -q -O - http://web-service; sleep 1; done
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-j82db
web-deploy-86cd4d65b9-nshfc
web-deploy-86cd4d65b9-j82db
web-deploy-86cd4d65b9-nshfc
```

## 세션 어피니티

svc-sa.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
  sessionAffinity: ClientIP
```

```bash
$ kubectl apply -f svc-sa.yml
service/web-service configured

$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/web-deploy-86cd4d65b9-65dqb   1/1     Running   0          17m
pod/web-deploy-86cd4d65b9-j82db   1/1     Running   0          17m
pod/web-deploy-86cd4d65b9-nshfc   1/1     Running   0          17m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   25h
service/web-service   ClusterIP   10.111.231.63   <none>        80/TCP    17m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-deploy   3/3     3            3           17m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/web-deploy-86cd4d65b9   3         3         3       17m

$ kubectl run -it bustbox --restart=Never --rm --image=busybox sh
If you don't see a command prompt, try pressing enter.

/ # while true; do wget -q -O - http://web-service; sleep 1; done
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
```

## NodePort

svc-np.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-np
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
  type: NodePort
```

```bash
$ kubectl apply -f deploy.yml
deployment.apps/web-deploy created

$ kubectl apply -f svc-np.yml
service/web-service-np created

$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        25h
web-service-np   NodePort    10.108.182.122   <none>        80:31115/TCP   46s

$ curl http://192.168.49.2:31115
curl: (7) Failed to connect to 192.168.49.2 port 31115: Timed out

$ >minikube service web-service-np
|-----------|----------------|-------------|---------------------------|
| NAMESPACE |      NAME      | TARGET PORT |            URL            |
|-----------|----------------|-------------|---------------------------|
| default   | web-service-np |          80 | http://192.168.49.2:31115 |
|-----------|----------------|-------------|---------------------------|
* web-service-np 서비스의 터널을 시작하는 중
|-----------|----------------|-------------|------------------------|
| NAMESPACE |      NAME      | TARGET PORT |          URL           |
|-----------|----------------|-------------|------------------------|
| default   | web-service-np |             | http://127.0.0.1:53371 |
|-----------|----------------|-------------|------------------------|
* Opening service default/web-service-np in default browser...
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.

$ curl http://127.0.0.1:53371
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 로드밸런서

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-lb
spec:
  selector:
    app: web
  ports:
  - name: webserver
    protocol: TCP
    port: 80
  type: LoadBalancer
```

```bash
$ kubectl apply -f deploy.yml
deployment.apps/web-deploy created

$ kubectl apply -f svc-lb.yml
service/web-service-np created

$ kubectl get svc
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP      10.96.0.1       <none>        443/TCP        25h
web-service-lb   LoadBalancer   10.106.64.214   <pending>     80:31648/TCP   7s

$ kubectl describe svc web-service-lb
Name:                     web-service-lb
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=web
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.106.64.214
IPs:                      10.106.64.214
Port:                     webserver  80/TCP
TargetPort:               80/TCP
NodePort:                 webserver  31648/TCP
Endpoints:                10.244.1.6:80,10.244.1.7:80,10.244.2.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ kubectl get no
NAME           STATUS   ROLES                  AGE   VERSION
minikube       Ready    control-plane,master   25h   v1.22.3
minikube-m02   Ready    <none>                 39m   v1.22.3
minikube-m03   Ready    <none>                 38m   v1.22.3

$ kubectl get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
web-deploy-86cd4d65b9-rwv7k   1/1     Running   0          10m   10.244.1.7   minikube-m02   <none>           <none>
web-deploy-86cd4d65b9-ttngq   1/1     Running   0          10m   10.244.1.6   minikube-m02   <none>           <none>
web-deploy-86cd4d65b9-wq2cl   1/1     Running   0          10m   10.244.2.4   minikube-m03   <none>           <none>

$ while true; do curl http://169.**.*.**; sleep 1; done
web-deploy-86cd4d65b9-rwv7k
web-deploy-86cd4d65b9-rwv7k
web-deploy-86cd4d65b9-ttngq
web-deploy-86cd4d65b9-ttngq
web-deploy-86cd4d65b9-wq2cl
web-deploy-86cd4d65b9-rwv7k
web-deploy-86cd4d65b9-ttngq
```

## ExternalName

svc-ext.yml
```yaml
kind: Service
apiVersion: v1
metadata:
  name: apl-on-baremetal
spec:
  type: ExternalName
  externalName: 10.132.253.7
```

```bash
$ kubectl apply -f svc-ext.yml
service/apl-on-baremetal created

$ kubectl get svc
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)   AGE
apl-on-baremetal   ExternalName   <none>       10.132.253.7   <none>    8s
kubernetes         ClusterIP      10.96.0.1    <none>         443/TCP   25h

$ kubectl run -it bustbox --restart=Never --rm --image=busybox sh
If you don't see a command prompt, try pressing enter.

/ # ping apl-on-baremetal
ping: bad address 'apl-on-baremetal'
```