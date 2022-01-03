# Kubenetes Service

| 서비스 타입 | 설명 |
|---|---|
| `ClusterIP`	| 타입을 지정하지 않으면 기본으로 설정되며, 클러스터 내부의 파드에서 서비스의 이름으로 접근할 수 있다 |
| `NodePort` | ClusterIP의 접근 범위뿐만 아니라 k8s 클러스터 외부에서도 노드의 IP 주소와 포트번호를 접근할 수 있다. |
| `LoadBalancer` | NodePort의 접근 번위뿐만 아니라 k8s 클러스터 외부에서 대표 IP 주소로 접근할 수 있다. |
| `ExternalName` | k8s 클러스터 내의 파드에서 외부 IP 조소에 서비스의 이름으로 접근할 수 있다. |

## Service
ClusterIP는 Pod 집합을 대표하는 IP 주소로 ClusterIP를 통해 Pod에 접근할 수 있다.<br>
NodePort는 외부에서 접근 가능한 Port를 열어주는 기능을 한다. 쉽고 편리하다는 장점이 있지만 정식 서비스에 사용하는 것은 추천하지 않는다.<br>
LoadBalancer는 로드밸런서와 연동하는 서비스를 해주며 NodePort와 ClusterIP가 자동으로 만들어진다.<br>
ExternalName은 Pod에서 외부의 k8s 클러스터의 외부 엔드포인트에 접속하기 위해 사용된다.

[ deploy.yml ]
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

[ svc.yml ]
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
# kubectl apply -f svc.yml
service/web-service created

# kubectl apply -f deploy.yml
deployment.apps/web-deploy created

# kubectl get all
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

# kubectl run -it bustbox --restart=Never --rm --image=busybox sh

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

# for pod in $(kubectl get pods | awk 'NR>1 {print $1}' | grep web-deploy); do kubectl exec $pod -- /bin/sh -c "hostname>/usr/share/nginx/html/index.html"; done

# kubectl run -it bustbox --restart=Never --rm --image=busybox sh

/ # while true; do wget -q -O - http://web-service; sleep 1; done
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-j82db
web-deploy-86cd4d65b9-nshfc
web-deploy-86cd4d65b9-j82db
web-deploy-86cd4d65b9-nshfc
```
각 Pod에 hostname을 index.html에 저장하는 script를 실행하고 while로 무한 루프를 돌면서 ClusterIP Service를 통해 Pod에 요청을 한다.

## Session Affinity
sessionAffinity를 설정하면 동일한 Client에서 온 요청을 늘 같은 Pod에게 전달하게 된다.

[ svc-sa.yml ]
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
# kubectl apply -f svc-sa.yml
service/web-service configured

# kubectl get all
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

# kubectl run -it bustbox --restart=Never --rm --image=busybox sh

/ # while true; do wget -q -O - http://web-service; sleep 1; done
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
web-deploy-86cd4d65b9-65dqb
```

## NodePort
서버를 운영할때 Iptable를 이용해서 Port를 열어주듯이 NodePort를 이용해서 Node에 접근할 수 있는 외부포트를 열어 줄수 있다.

[ svc-np.yml ]
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

[ minikube ]
```bash
# kubectl apply -f deploy.yml
deployment.apps/web-deploy created

# kubectl apply -f svc-np.yml
service/web-service-np created

# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        25h
web-service-np   NodePort    10.108.182.122   <none>        80:31115/TCP   46s

# curl http://192.168.49.2:31115
curl: (7) Failed to connect to 192.168.49.2 port 31115: Timed out

# minikube service web-service-np
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

# curl http://127.0.0.1:53371
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

[ Virtual Machine ]
```bash
[root@m-k8s Kubernetes]# kubectl get svc
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        3d1h
web-service-np   NodePort    10.97.196.156   <none>        80:30551/TCP   12m

[root@m-k8s Kubernetes]# kubectl get nodes -o wide
NAME     STATUS   ROLES    AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
m-k8s    Ready    master   3d1h   v1.18.4   192.168.1.10    <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w1-k8s   Ready    <none>   3d1h   v1.18.4   192.168.1.101   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w2-k8s   Ready    <none>   3d1h   v1.18.4   192.168.1.102   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w3-k8s   Ready    <none>   3d1h   v1.18.4   192.168.1.103   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1

[root@m-k8s Kubernetes]# curl 192.168.1.101:30551
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

[root@m-k8s Kubernetes]# curl 192.168.1.102:30551
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

[root@m-k8s Kubernetes]# curl 192.168.1.103:30551
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
minikube의 경우 자체 서비스를 생성해주어야 Node에 접근이 가능하다.

## LoadBalancer
우리가 잘 알고 있는 로드벨런스의 역활을 하는 서비스이다.

[ svc-lb.yml ]
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
[root@m-k8s Kubernetes]# kubectl apply -f deploy.yml
deployment.apps/web-deploy created

[root@m-k8s Kubernetes]# kubectl apply -f svc-lb.yml 
service/web-service-lb created

[root@m-k8s Kubernetes]# kubectl get svc
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP      10.96.0.1       <none>        443/TCP        3d2h
web-service-lb   LoadBalancer   10.106.187.64   <pending>     80:31865/TCP   7m5s

[root@m-k8s Kubernetes]# kubectl describe svc web-service-lb                                                                                       Name:                     web-service-lb
Namespace:                default
Labels:                   <none>
Annotations:              Selector:  app=web
Type:                     LoadBalancer
IP:                       10.106.187.64
Port:                     webserver  80/TCP
TargetPort:               80/TCP
NodePort:                 webserver  31865/TCP
Endpoints:                172.16.103.135:80,172.16.132.19:80,172.16.221.143:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

[root@m-k8s Kubernetes]# kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-756987f8f4-5kcl4   1/1     Running   0          36m
web-deploy-756987f8f4-6jtkh   1/1     Running   0          36m
web-deploy-756987f8f4-hqj7p   1/1     Running   0          36m

[root@m-k8s Kubernetes]# for pod in $(kubectl get pods | awk 'NR>1 {print $1}' | grep web-deploy); do kubectl exec $pod -- /bin/sh -c "hostname>/usr/share/nginx/html/index.html"; done

[root@m-k8s Kubernetes]# while true; do curl http://10.106.187.64; sleep 1; done                                                                   web-deploy-756987f8f4-5kcl4
web-deploy-756987f8f4-hqj7p
web-deploy-756987f8f4-hqj7p
web-deploy-756987f8f4-6jtkh
web-deploy-756987f8f4-hqj7p
web-deploy-756987f8f4-hqj7p
```

## ExternalName
Kubernetes 내부에서 외부의 어플리케이션에 접속하는 경우 사용이 되는데... 문제는 제대로 작동하지 않는다.

[ svc-ext.yml ]
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
# kubectl apply -f svc-ext.yml
service/apl-on-baremetal created

# kubectl get svc
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)   AGE
apl-on-baremetal   ExternalName   <none>       10.132.253.7   <none>    8s
kubernetes         ClusterIP      10.96.0.1    <none>         443/TCP   25h

# kubectl run -it bustbox --restart=Never --rm --image=busybox sh

/ # ping apl-on-baremetal
ping: bad address 'apl-on-baremetal'
```