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

[ service-account.yml ]
```yaml
# 노드 감시용 서비스 어카운트(SA) 작성
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: tkr-system
  name: high-availability
```

[ role-base-access-ctl.yml ]
```yaml
# 클러스터 롤
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nodes
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","delete"]
---
# 클러스터롤과 서비스 어카운트의 바인딩
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nodes
subjects:
- kind: ServiceAccount
  name: high-availability  # 서비스 어카운트 이름
  namespace: tkr-system    # 네임스페이스 지정은 필수
roleRef:
  kind: ClusterRole
  name: nodes
  apiGroup: rbac.authorization.k8s.io
```

[ namespace.yml ]
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tkr-system    # 전용 네임스페이스
```

[ daemonset.yml ]
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: liberator
  namespace: tkr-system    # 시스템 전용 네임스페이스
spec:
  selector:
    matchLabels:
      name: liberator
  template:
    metadata:
      labels:
        name: liberator
    spec:
      serviceAccountName: high-availability # 권한을 가진 서비스 어카운트
      containers:
      - name: liberator
        image: maho/liberator:0.1
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

```bash
# kubectl get no -o=custom-columns=NAME:.metadata.name,STATUS:.status.conditions[4].type
NAME     STATUS
m-k8s    Ready
w1-k8s   Ready
w2-k8s   Ready
w3-k8s   Ready

# kubectl describe clusterrole admin -n kube-system
Name:         admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources                                       Non-Resource URLs  Resource Names  Verbs
  ---------                                       -----------------  --------------  -----
  rolebindings.rbac.authorization.k8s.io          []                 []              [create delete deletecollection get list patch update watch]
  roles.rbac.authorization.k8s.io                 []                 []              [create delete deletecollection get list patch update watch]
  configmaps                                      []                 []              [create delete deletecollection patch update get list watch]
  endpoints                                       []                 []              [create delete deletecollection patch update get list watch]
  persistentvolumeclaims                          []                 []              [create delete deletecollection patch update get list watch]
  pods                                            []                 []              [create delete deletecollection patch update get list watch]
  replicationcontrollers/scale                    []                 []              [create delete deletecollection patch update get list watch]
  replicationcontrollers                          []                 []              [create delete deletecollection patch update get list watch]
  services                                        []                 []              [create delete deletecollection patch update get list watch]
  daemonsets.apps                                 []                 []              [create delete deletecollection patch update get list watch]
  deployments.apps/scale                          []                 []              [create delete deletecollection patch update get list watch]
  deployments.apps                                []                 []              [create delete deletecollection patch update get list watch]
  replicasets.apps/scale                          []                 []              [create delete deletecollection patch update get list watch]
  replicasets.apps                                []                 []              [create delete deletecollection patch update get list watch]
  statefulsets.apps/scale                         []                 []              [create delete deletecollection patch update get list watch]
  statefulsets.apps                               []                 []              [create delete deletecollection patch update get list watch]
  horizontalpodautoscalers.autoscaling            []                 []              [create delete deletecollection patch update get list watch]
  cronjobs.batch                                  []                 []              [create delete deletecollection patch update get list watch]
  jobs.batch                                      []                 []              [create delete deletecollection patch update get list watch]
  daemonsets.extensions                           []                 []              [create delete deletecollection patch update get list watch]
  deployments.extensions/scale                    []                 []              [create delete deletecollection patch update get list watch]
  deployments.extensions                          []                 []              [create delete deletecollection patch update get list watch]
  ingresses.extensions                            []                 []              [create delete deletecollection patch update get list watch]
  networkpolicies.extensions                      []                 []              [create delete deletecollection patch update get list watch]
  replicasets.extensions/scale                    []                 []              [create delete deletecollection patch update get list watch]
  replicasets.extensions                          []                 []              [create delete deletecollection patch update get list watch]
  replicationcontrollers.extensions/scale         []                 []              [create delete deletecollection patch update get list watch]
  ingresses.networking.k8s.io                     []                 []              [create delete deletecollection patch update get list watch]
  networkpolicies.networking.k8s.io               []                 []              [create delete deletecollection patch update get list watch]
  poddisruptionbudgets.policy                     []                 []              [create delete deletecollection patch update get list watch]
  deployments.apps/rollback                       []                 []              [create delete deletecollection patch update]
  deployments.extensions/rollback                 []                 []              [create delete deletecollection patch update]
  localsubjectaccessreviews.authorization.k8s.io  []                 []              [create]
  pods/attach                                     []                 []              [get list watch create delete deletecollection patch update]
  pods/exec                                       []                 []              [get list watch create delete deletecollection patch update]
  pods/portforward                                []                 []              [get list watch create delete deletecollection patch update]
  pods/proxy                                      []                 []              [get list watch create delete deletecollection patch update]
  secrets                                         []                 []              [get list watch create delete deletecollection patch update]
  services/proxy                                  []                 []              [get list watch create delete deletecollection patch update]
  bindings                                        []                 []              [get list watch]
  events                                          []                 []              [get list watch]
  limitranges                                     []                 []              [get list watch]
  namespaces/status                               []                 []              [get list watch]
  namespaces                                      []                 []              [get list watch]
  persistentvolumeclaims/status                   []                 []              [get list watch]
  pods/log                                        []                 []              [get list watch]
  pods/status                                     []                 []              [get list watch]
  replicationcontrollers/status                   []                 []              [get list watch]
  resourcequotas/status                           []                 []              [get list watch]
  resourcequotas                                  []                 []              [get list watch]
  services/status                                 []                 []              [get list watch]
  controllerrevisions.apps                        []                 []              [get list watch]
  daemonsets.apps/status                          []                 []              [get list watch]
  deployments.apps/status                         []                 []              [get list watch]
  replicasets.apps/status                         []                 []              [get list watch]
  statefulsets.apps/status                        []                 []              [get list watch]
  horizontalpodautoscalers.autoscaling/status     []                 []              [get list watch]
  cronjobs.batch/status                           []                 []              [get list watch]
  jobs.batch/status                               []                 []              [get list watch]
  daemonsets.extensions/status                    []                 []              [get list watch]
  deployments.extensions/status                   []                 []              [get list watch]
  ingresses.extensions/status                     []                 []              [get list watch]
  replicasets.extensions/status                   []                 []              [get list watch]
  ingresses.networking.k8s.io/status              []                 []              [get list watch]
  poddisruptionbudgets.policy/status              []                 []              [get list watch]
  serviceaccounts                                 []                 []              [impersonate create delete deletecollection patch update get list watch]

# kubectl apply -f k8s-rbac/
namespace/tkr-system created
clusterrole.rbac.authorization.k8s.io/nodes created
clusterrolebinding.rbac.authorization.k8s.io/nodes created
serviceaccount/high-availability created

# kubectl apply -f daemonset.yml 
daemonset.apps/liberator created

# kubectl get ds
No resources found in default namespace.

# kubectl get ds -n tkr-system
NAME        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
liberator   3         3         3       3            3           <none>          2m1s

# kubectl get po -n tkr-system           
NAME              READY   STATUS    RESTARTS   AGE
liberator-n25m8   1/1     Running   0          2m31s
liberator-wfkdv   1/1     Running   0          2m31s
liberator-wqkn8   1/1     Running   0          2m31s

# kubectl exec -it liberator-n25m8 -n tkr-system bash

root@liberator-n25m8:/# df -h
Filesystem                   Size  Used Avail Use% Mounted on
overlay                       37G  3.7G   34G  10% /
tmpfs                        1.3G     0  1.3G   0% /dev
tmpfs                        1.3G     0  1.3G   0% /sys/fs/cgroup
/dev/mapper/centos_k8s-root   37G  3.7G   34G  10% /etc/hosts
shm                           64M     0   64M   0% /dev/shm
tmpfs                        1.3G   12K  1.3G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                        1.3G     0  1.3G   0% /proc/acpi
tmpfs                        1.3G     0  1.3G   0% /proc/scsi
tmpfs                        1.3G     0  1.3G   0% /sys/firmware

root@liberator-n25m8:/# ls /run/secrets/kubernetes.io/serviceaccount/
ca.crt  namespace  token
```

```bash
# kubectl apply -f mysql-sts.yml 
service/mysql created
statefulset.apps/mysql created

# kubectl get sts
NAME    READY   AGE
mysql   1/1     12s

# kubectl get po
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   1/1     Running   0          42s

# kubectl get po mysql-0 -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          55s   172.16.132.7   w3-k8s   <none>           <none>

# kubectl exec -it mysql-0 bash

root@mysql-0:/# mysql -u root -p

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
5 rows in set (0.14 sec)

mysql> create database test123;
Query OK, 1 row affected (0.05 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hello              |
| mysql              |
| performance_schema |
| sys                |
| test123            |
+--------------------+
6 rows in set (0.01 sec)


mysql> exit
Bye

root@mysql-0:/# exit
exit

# while true; do date; kubectl get po -o wide; echo; sleep 15; done 

// # vagrant halt w3-k8s 실행 후
Mon Jan  3 14:03:26 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          16m   172.16.132.7   w3-k8s   <none>           <none>

Mon Jan  3 14:03:41 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          16m   172.16.132.7   w3-k8s   <none>           <none>

Mon Jan  3 14:03:56 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          16m   172.16.132.7   w3-k8s   <none>           <none>

// # kubectl delete node w3-k8s 실행 후
Mon Jan  3 14:06:59 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          8s    172.16.221.130   w1-k8s   <none>           <none>

Mon Jan  3 14:07:14 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          23s   172.16.221.130   w1-k8s   <none>           <none>

Mon Jan  3 14:07:29 KST 2022
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          38s   172.16.221.130   w1-k8s   <none>           <none>

# kubectl get node -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
m-k8s    Ready    master   12d     v1.18.4   192.168.1.10    <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w1-k8s   Ready    <none>   3d23h   v1.18.4   192.168.1.101   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1
w2-k8s   Ready    <none>   12d     v1.18.4   192.168.1.102   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://1.13.1

# kubectl get po mysql-0 -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
mysql-0   1/1     Running   0          4m38s   172.16.221.130   w1-k8s   <none>           <none>

# kubectl exec -it mysql-0 bash

root@mysql-0:/# mysql -u root -p

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hello              |
| mysql              |
| performance_schema |
| sys                |
| test123            |
+--------------------+
6 rows in set (0.02 sec)

mysql> exit
Bye

root@mysql-0:/# exit
exit

# kubectl get ep mysql
NAME    ENDPOINTS             AGE
mysql   172.16.221.130:3306   27m
```

```bash
# vagrant status
Current machine states:

m-k8s                     running (virtualbox)
w1-k8s                    running (virtualbox)
w2-k8s                    running (virtualbox)
w3-k8s                    poweroff (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```