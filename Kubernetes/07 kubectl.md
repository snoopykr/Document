# Kubernetes StatefulSet

## StatefulSet 구동
[ mysql-sts.yml ]
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql        ## 이 이름이 k8s내 DNS에 등록됨. 
  labels:
    app: mysql-sts
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None    ## 헤드리스 서비스 설정
  selector:
    app: mysql-sts   ## 후술하는 스테이트풀셋과 연결시키는 라벨
---
## MySQL 스테이트풀셋
#
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql        ## 연결할 서비스의 이름 설정
  replicas: 1               ## 파드 기동 개수
  selector:
    matchLabels:
      app: mysql-sts
  template:
    metadata:
      labels:
        app: mysql-sts
    spec:
      containers:           
      - name: mysql
        image: mysql:5.7    ## Docker Hub MySQL 리포지터리 지정
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: qwerty
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:       ## 컨테이너상의 마운트 경로 설정 
        - name: pvc
          mountPath: /var/lib/mysql
          subPath: data     ## 초기화 시 빈 디렉터리가 필요
        livenessProbe:      ## MySQL 기동 확인 
          exec:
            command: ["mysqladmin","-p$MYSQL_ROOT_PASSWORD","ping"]
          initialDelaySeconds: 60
          timeoutSeconds: 10
  volumeClaimTemplates:     ## 볼륨 요구 템플릿
  - metadata:
      name: pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      ## 환경에 맞게 선택하여, sotrage의 값을 편집
      #storageClassName: ibmc-file-bronze   # 용량 20Gi IKS
      storageClassName: gluster-heketi     # 용량 12Gi GlusterFS
      #storageClassName: standard            # 용량 2Gi  Minikube/GKE
      resources:
        requests:
          storage: 2Gi
```

```bash
# kubectl apply -f mysql-sts.yml 
service/mysql created
statefulset.apps/mysql created

# kubectl get svc,sts,po
NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/glusterfs-dynamic-7a40287b-1b50-4bff-83d5-6de2894406b7   ClusterIP   10.110.17.144   <none>        1/TCP      95s
service/kubernetes                                               ClusterIP   10.96.0.1       <none>        443/TCP    8d
service/mysql                                                    ClusterIP   None            <none>        3306/TCP   95s

NAME                     READY   AGE
statefulset.apps/mysql   1/1     95s

NAME          READY   STATUS    RESTARTS   AGE
pod/mysql-0   1/1     Running   0          95s

# kubectl exec -it mysql-0 -- bash

root@mysql-0:/# mysql -u root -pqwerty
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database hello;
Query OK, 1 row affected (0.03 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hello              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> exit
Bye

root@mysql-0:/# exit
exit

# kubectl delete -f mysql-sts.yml 
service "mysql" deleted
statefulset.apps "mysql" deleted

# kubectl get svc,sts,po
NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/glusterfs-dynamic-7a40287b-1b50-4bff-83d5-6de2894406b7   ClusterIP   10.110.17.144   <none>        1/TCP     5m10s
service/kubernetes                                               ClusterIP   10.96.0.1       <none>        443/TCP   8d

# kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
pvc-mysql-0   Bound    pvc-7a40287b-1b50-4bff-83d5-6de2894406b7   2Gi        RWO            gluster-heketi   5m17s

# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS     REASON   AGE
pvc-7a40287b-1b50-4bff-83d5-6de2894406b7   2Gi        RWO            Delete           Bound    default/pvc-mysql-0   gluster-heketi            5m15s

# kubectl apply -f mysql-sts.yml 
service/mysql created
statefulset.apps/mysql created

# kubectl exec -it mysql-0 -- bash

root@mysql-0:/# mysql -u root -pqwerty
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hello              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.04 sec)
```
StatefulSet은 Pod와 퍼시스턴트 볼륨의 대응 관계를 더욱 엄격하게 관리하여, 퍼시스턴트 볼륨의 데이터 보관을 우선시하여 동작한다.

## 수동 테이크 오버

```bash
# kubectl get po mysql-0 -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          5m14s   172.16.132.5   w3-k8s   <none>           <none>

// 스케일 금지
# kubectl cordon w3-k8s
node/w3-k8s cordoned

// 파드 이동
# kubectl drain w3-k8s --ignore-daemonsets
node/w3-k8s already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-74wdp, kube-system/kube-proxy-xwh9h
evicting pod default/mysql-0
pod/mysql-0 evicted
node/w3-k8s evicted

# kubectl get po mysql-0 -o wide          
NAME      READY   STATUS              RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
mysql-0   0/1     ContainerCreating   0          8s    <none>   w1-k8s   <none>           <none>
```
노드가 일시적으로 정지해야 하는 경우 Pod를 이동할때 테이크 오버가 활용된다.

`kubectl uncordon w3-k8s`를 사용해서 스케일을 정상화 할수 있다.

## Node Handling

[ Master Node ]
```bash
# kubectl get no    
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   8d    v1.18.4
w1-k8s   Ready    <none>   8d    v1.18.4
w2-k8s   Ready    <none>   8d    v1.18.4
w3-k8s   Ready    <none>   8d    v1.18.4

# kubectl get po -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   5          10m   172.16.103.135   w1-k8s   <none>           <none>
```
모든 상태 정상.

[ Terminal ]
```bash
# vagrant halt w1-k8s
```
w1-k8s Node를 정지시킴

[ Master Node ]
```bash
# kubectl get no    
NAME     STATUS      ROLES    AGE   VERSION
m-k8s    Ready       master   8d    v1.18.4
w1-k8s   NotReady    <none>   8d    v1.18.4
w2-k8s   Ready       <none>   8d    v1.18.4
w3-k8s   Ready       <none>   8d    v1.18.4

# kubectl get po -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Unknown   5          10m   172.16.103.135   w1-k8s   <none>           <none>
```
서비스 불가.

[ Master Node ]
```bash
// Node 삭제
# kubectl delete node w1-k8s
node "w1-k8s" deleted

# kubectl get no    
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   8d    v1.18.4
w2-k8s   Ready    <none>   8d    v1.18.4
w3-k8s   Ready    <none>   8d    v1.18.4

// Pod의 이동
# kubectl get po -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   7          15m   172.16.103.135   w2-k8s   <none>           <none>
```
w1-k8s Node가 삭제된 상태에서 서비스 정상화

## Node 추가

[ Master Node]
```bash
# kubeadm token list
TOKEN                     TTL         EXPIRES   USAGES                   DESCRIPTION                                                EXTRA GROUPS
123456.1234567890123456   <forever>   <never>   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token

# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
e61f573ea07a788f681f69fb8b0a7e36695c202b5fd9ca12fa216794ace77ecd
```
invalid하지 않는 토근이 없는 경우에는 `kubeadm token create(or generate)` 실행

[ w1-k8s Node]
```bash
# kubeadm reset

# kubeadm join 192.168.1.10:6443 --token 123456.1234567890123456 \
>     --discovery-token-ca-cert-hash sha256:e61f573ea07a788f681f69fb8b0a7e36695c202b5fd9ca12fa216794ace77ecd
```
Master에서 구한 토큰과 해쉬값을 사용해서 조인을 한다.

[ Master Node ]
```bash
# kubectl get no    
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   8d    v1.18.4
w1-k8s   Ready    <none>   5m    v1.18.4
w2-k8s   Ready    <none>   8d    v1.18.4
w3-k8s   Ready    <none>   8d    v1.18.4

# kubectl get po -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   7          12m   172.16.103.135   w2-k8s   <none>           <none>
```
정상적으로 조인된 것을 확인할 수 있다.

## 테이크 오버 자동화

[ Dockerfile ]
```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl apt-transport-https gnupg2

# pyhon
RUN apt-get install -y python python-pip
RUN pip install kubernetes

COPY main.py /main.py

WORKDIR /
CMD python /main.py
```

[ main.py ]
```py
# coding: UTF-8
#
# 상태 불명의 노드를 클러스터에서 제거 
#
import signal, os, sys
from kubernetes import client, config
from kubernetes.client.rest import ApiException
from time import sleep

uk_node = {}  # KEY는 상태 불명이 된 노드의 이름, 값은 카운트

## 정지 요청 시그널 처리 
def handler(signum, frame):
    sys.exit(0)

## 노드 제거 함수 
def node_delete(v1,name):
    body = client.V1DeleteOptions()
    try:
        resp = v1.delete_node(name, body)
        print("delete node %s done" % name)
    except ApiException as e:
        print("Exception when calling CoreV1Api->delete_node: %s\n" % e)

## 노드 감시 함수 
def node_monitor(v1):
    try:
        ret = v1.list_node(watch=False)
        for i in ret.items:
            n_name = i.metadata.name
            #print("%s" % (i.metadata.name)) #디버그 용 
            for j in i.status.conditions:
                #print("\t%s\t%s" % (j.type, j.status)) #디버그 용 
                if (j.type == "Ready" and j.status != "True"):
                    if n_name in uk_node:
                        uk_node[n_name] += 1
                    else:
                        uk_node[n_name] = 0
                    print("unknown %s  count=%d" % (n_name,uk_node[n_name]))
                    # 카운터가 3회 넘어서면 노드를 제거 
                    if uk_node[n_name] > 3:
                        del uk_node[n_name]
                        node_delete(v1,i.metadata.name)
                # 1번이라도 상태가 돌아오면 카운터를 초기화
                if (j.type == "Ready" and j.status == "True"):
                    if n_name in uk_node:
                        del uk_node[n_name]
    except ApiException as e:
        print("Exception when calling CoreV1Api->list_node: %s\n" % e)

## 메인
if __name__ == '__main__':
    signal.signal(signal.SIGTERM, handler) # 시그널 처리 
    config.load_incluster_config()         # 인증 정보 취득
    v1 = client.CoreV1Api()                # 인스턴스화
    # 감시 루프
    while True:
        node_monitor(v1)
        sleep(5) # 감시 간격
```

```bash
# docker build --tag welovefish/liberator:0.1 .

# docker login

# docker push welovefish/liberator:0.1

```
