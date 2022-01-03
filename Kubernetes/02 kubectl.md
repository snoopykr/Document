# Kubernetes Manifest
Manifest는 kubernetes의 오브젝트를 생성하기 위해 메타 정보를 YAML, JSON으로 기술한 파일이다.

## nginx-pod

[ nginx-pod.yml ]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

## kubectl apply -f

```bash
# kubectl apply -f nginx-pod.yml
pod/nginx created


# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          22s



# kubectl get po nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m13s   172.17.0.3   minikube   <none>           <none>

# curl -m 3 http://172.17.0.3
```
curl에서 사용한 `-m 3`은 -max-time으로 3초후 timeout를 의미한다.

## kubectl delete -f 

```bash
# kubectl delete -f nginx-pod.yml
pod "nginx" deleted
```

## Health check
Liveness Probe (활성) : 정상적으로 실행중인 것을 검사<br>
Readiness Probe(준비) : 요청을 받을 준비가 되었는지를 검사

[ webapl-pod.yml ]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapl
spec:
  containers:
  - name: webapl
    image: welovefish/webapl:0.1    # 핸들러를 구현한 애플리케이션 
    livenessProbe:                  # 애플리케이션이 살아있는지 확인
      httpGet:
        path: /healthz              # 확인 경로
        port: 3000
      initialDelaySeconds: 3        # 검사 개시 대기 시간
      periodSeconds: 5              # 검사 간격
    readinessProbe:                 # 애플리케이션이 준비되었는지 확인
      httpGet:
        path: /ready                # 확인 경로
        port: 3000
      initialDelaySeconds: 15
      periodSeconds: 6
```

[ webapi/Dockerfile ]
```dockerfile
## Alpine Linux  https://hub.docker.com/_/alpine/
FROM alpine:latest

## Node.js  https://pkgs.alpinelinux.org/package/edge/main/x86_64/nodejs
RUN apk update && apk add --no-cache nodejs npm

## 의존 모듈
WORKDIR /
ADD ./package.json /
RUN npm install
ADD ./webapl.js /

## 애플리케이션 기동
CMD node /webapl.js
```

[ webapi/package.json ]
```json
{
  "name": "webapl",
  "version": "1.0.0",
  "description": "",
  "main": "webapl.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.3"
  }
}
```

[ webapi/webapl.js ]
```js
// 모의 애플리케이션
//
const express = require('express')
const app = express()
var start = Date.now()

// Liveness 프로브 핸들러
// 기동 후 40초가 되면, 500 에러를 반환한다.
// 그 전까지는 HTTP 200 OK를 반환한다.
// 즉, 40초가 되면, Liveness프로브가 실패하여 컨테이너가 재기동한다. 
//
app.get('/healthz', function(request, response) {
    var msec = Date.now() - start
    var code = 200
    if (msec > 40000 ) {
	code = 500
    }
    console.log('GET /healthz ' + code)
    response.status(code).send('OK')
})

// Rediness 프로브 핸들러
// 애플리케이션의 초기화 시간으로 
// 기동 후 20초 지나고 나서부터 HTTP 200을 반환한다. 
// 그 전까지는 HTTPS 200 OK를 반환한다.
app.get('/ready', function(request, response) {
    var msec = Date.now() - start
    var code = 500
    if (msec > 20000 ) {
	code = 200
    }
    console.log('GET /ready ' + code)
    response.status(code).send('OK')
})

// 첫 화면
//
app.get('/', function(request, response) {
    console.log('GET /')
    response.send('Hello from Node.js')
})

// 서버 포트 번호
//
app.listen(3000);
```

```bash
# docker build --tag welovefish/webapi:0.1 .

# docker login

# docker push welovefish/webapi:0.1

# kubectl apply -f webapl-pod.yml

# kubectl get pod
NAME     READY   STATUS              RESTARTS   AGE
webapl   0/1     ContainerCreating   0          12s

# kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
webapl   0/1     Running   0          26s

# kubectl logs pod/webapl
GET /healthz 200
GET /healthz 200
GET /healthz 200
GET /ready 500
GET /healthz 200

# kubectl describe pod webapl
Name:         webapl
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Wed, 08 Dec 2021 15:08:29 +0900
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:  172.17.0.3
Containers:
  webapl:
    Container ID:   docker://fe7f39f00c797cc46b690857b39ab9aed37c5f2d747decec422c39cf73278417
    Image:          maho/webapl:0.1
    Image ID:       docker-pullable://maho/webapl@sha256:2d90f1ef4d0b4b0dcf3372289e7552ef23de3a1910d747b3d42ee5e46b96d57a
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 08 Dec 2021 15:14:24 +0900
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Wed, 08 Dec 2021 15:12:59 +0900
      Finished:     Wed, 08 Dec 2021 15:14:24 +0900
    Ready:          True
    Restart Count:  4
    Liveness:       http-get http://:3000/healthz delay=3s timeout=1s period=5s #success=1 #failure=3
    Readiness:      http-get http://:3000/ready delay=15s timeout=1s period=6s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tz9c4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-tz9c4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  6m39s                  default-scheduler  Successfully assigned default/webapl to minikube
  Normal   Pulling    6m39s                  kubelet            Pulling image "maho/webapl:0.1"
  Normal   Pulled     6m25s                  kubelet            Successfully pulled image "maho/webapl:0.1" in 14.5106522s
  Normal   Created    3m35s (x3 over 6m25s)  kubelet            Created container webapl
  Normal   Started    3m35s (x3 over 6m25s)  kubelet            Started container webapl
  Normal   Pulled     3m35s (x2 over 5m)     kubelet            Container image "maho/webapl:0.1" already present on machine
  Warning  Unhealthy  3m16s (x3 over 6m10s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    2m40s (x3 over 5m30s)  kubelet            Container webapl failed liveness probe, will be restarted
  Warning  Unhealthy  85s (x10 over 5m40s)   kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
```

## 초기화 전용 컨테이너

[ init-sample.yml ]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-sample
spec:
  containers:
  - name: main           # 메인 컨테이너
    image: ubuntu
    command: ["/bin/sh"]
    args: [ "-c", "tail -f /dev/null"]
    volumeMounts:
    - mountPath: /docs   # 공유 볼륨 마운트 경로
      name: data-vol
      readOnly: false

  initContainers:        # 메인 컨테이너 실행 전에 초기화 전용 컨테이너를 기동 
  - name: init
    image: alpine
    ## 공유 볼륨에 디렉터리를 작성하고, 소유를 변경
    command: ["/bin/sh"]
    args: [ "-c", "mkdir /mnt/html; chown 33:33 /mnt/html" ]
    volumeMounts:
    - mountPath: /mnt    # 공유 볼륨 마운트 경로 
      name: data-vol
      readOnly: false

  volumes:               # 파드의 공유 볼륨
  - name: data-vol
    emptyDir: {}
```

```bash
# kubectl apply -f init-sample.yml

# kubectl exec -it init-sample -c main sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

#  ls -al /docs
total 12
drwxrwxrwx 3 root     root     4096 Dec  8 06:26 .
drwxr-xr-x 1 root     root     4096 Dec  8 06:26 ..
drwxr-xr-x 2 www-data www-data 4096 Dec  8 06:26 html
```
`-c`는 pod내 컨테이너를 지정하기 위해 사용

## 사이드카

[ contents-cloner ]
```bash
#!/bin/bash
# 최신 Web 데이터를 GitHub로부터 취득 

# 환경변수가 설정되어 있지 않으면 에러 종료
if [ -z $CONTENTS_SOURCE_URL ]; then
   exit 1
fi

# 처음에는 GitHub에서 클론 
git clone $CONTENTS_SOURCE_URL /data

# SIGTERM 수신 시 처리
save() {
  exit 0
}
trap save TERM

# 이후에는 1준 간격으로 git pull을 수행
cd /data
while true
do
   date
   sleep 60
   git pull
done
```

[ Dockerfile ]
```dockerfile
## Contents Cloner Image
FROM ubuntu:16.04
RUN apt-get update && apt-get install -y git
COPY ./contents-cloner /contents-cloner
RUN chmod a+x /contents-cloner
WORKDIR /
CMD ["/contents-cloner"]
```

[ webserver.yml ]
```yaml
## 사이드카 파드 예제
#
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:          ## 메인 컨테이너
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: contents-vol
      readOnly: true
      
  - name: cloner       ## 사이드카 컨테이너
    image: maho/c-cloner:0.1
    env:
    - name: CONTENTS_SOURCE_URL
      value: "https://github.com/takara9/web-contents"
    volumeMounts:
    - mountPath: /data
      name: contents-vol
      
  volumes:             ## 공유 볼륨
  - name: contents-vol
    emptyDir: {}
```

```bash
# docker build --tag welovefish/c-cloner:0.1 .

# docker login

# docker push welovefish/c-cloner:0.1

# kubectl apply -f webserver.yml

# kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
webserver     2/2     Running   0          58s

# kubectl get po -o wide
NAME          READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
webserver     2/2     Running   0          117s   172.17.0.4   minikube   <none>           <none>

# kubectl run busybox --image=busybox --restart=Never --rm -it sh

/ # wget -q -O - http://172.17.0.4
<!DOCTYPE html>
<html>
<head>
<title>ポッドテンプレート役割</title>
</head>
<body>
<h1>ポッドテンプレート役割</h1>

<p>ポッドテンプレートは、デプロイメント、レプリカセット、ジョブ、およびステートフルセットなどのコントローラに対するポッ ド仕様です。 これらコントローラは、ポッドテンプレートを使用して実際のポッドを作成します。</p>

<p><a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/">Pod Overview</a>.</p>

</body>
</html>
```
pod내에 2개의 컨테이너(nginx, cloner)를 만들고 볼륨을 서로 공유한다. cloner는 git를 주기적으로 pull해서 볼륨에 저장하고 nginx는 요청이 있을때 볼륨에서 읽어서 요청사항을 처리한다.