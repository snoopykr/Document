# Kubernetes Storage

## 퍼시스턴트 볼륨

[ pvc.yml ]
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data1
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 2Gi
```

[ pod.yml ]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  volumes:
  - name: pvc1
    persistentVolumeClaim:
      claimName: data1
  containers:
  - name: ubuntu
    image: ubuntu:16.04
    volumeMounts:
    - name: pvc1
      mountPath: /mnt
    command: ["/usr/bin/tail","-f","/dev/null"]      
```

```bash
# kubectl apply -f pvc.yml
persistentvolumeclaim/data1 created

# kubectl get pvc,pv
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data1   Bound    pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            standard       31s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            Delete           Bound    default/data1   standard                31s

# kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  3d23h

# kubectl apply -f pod.yml
pod/pod1 created

# kubectl get pvc,pv,po
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data1   Bound    pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            standard       5m38s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            Delete           Bound    default/data1   standard                5m38s

NAME       READY   STATUS    RESTARTS   AGE
pod/pod1   1/1     Running   0          15s

# kubectl exec -it pod1 -- sh

# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         251G   15G  224G   7% /
tmpfs            64M     0   64M   0% /dev
tmpfs            13G     0   13G   0% /sys/fs/cgroup
/dev/sdc        251G   15G  224G   7% /mnt
shm              64M     0   64M   0% /dev/shm
tmpfs            25G   12K   25G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            13G     0   13G   0% /proc/acpi
tmpfs            13G     0   13G   0% /sys/firmware
```

## NFS

[ nfs-pv.yml ]
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1
  labels:
    name: pv-nfs-1
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 172.16.20.10
    path: /export
```

[ nfs-pvc.yml ]
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-1
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: "100Mi"
  selector:
    matchLabels:
      name: pv-nfs-1
```

[ nfs-client.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu:16.04
        volumeMounts:
        - name: nfs
          mountPath: /mnt
        command: ["/usr/bin/tail","-f","/dev/null"]
      volumes:
      - name: nfs
        persistentVolumeClaim:
          claimName: nfs-1
```

```bash
# kubectl apply -f nfs-pv.yml
persistentvolume/nfs-1 created

# kubectl run -it bb --image=busybox sh

/ # ping 172.16.20.10
PING 172.16.20.10 (172.16.20.10): 56 data bytes
64 bytes from 172.16.20.10: seq=3 ttl=36 time=7.445 ms
64 bytes from 172.16.20.10: seq=4 ttl=36 time=7.136 ms
64 bytes from 172.16.20.10: seq=5 ttl=36 time=6.930 ms
64 bytes from 172.16.20.10: seq=6 ttl=36 time=7.095 ms
^C
--- 172.16.20.10 ping statistics ---
14 packets transmitted, 11 packets received, 21% packet loss
round-trip min/avg/max = 6.787/7.434/8.702 ms
/ #

# kubectl apply -f nfs-pvc.yml
persistentvolumeclaim/nfs-1 created

# kubectl get pv,pvc
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/nfs-1   100Mi      RWX            Retain           Bound    default/nfs-1                           6m35s

NAME                          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nfs-1   Bound    nfs-1    100Mi      RWX                           22s

# kubectl apply -f nfs-client.yml
deployment.apps/nfs-client created

# kubectl get po
NAME                         READY   STATUS              RESTARTS   AGE
bb                           1/1     Running             0          7m46s
nfs-client-7ff95d88b-79gg7   0/1     ContainerCreating   0          59s
nfs-client-7ff95d88b-m4mrq   0/1     ContainerCreating   0          59s
```

## GlusterFS

[ gfs-sc.yml ]
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: "gluster-heketi"
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://172.20.1.20:8080"
  restuser: "admin"
  restuserkey: "admin"
```

[ gfs-pvc.yml ]
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: gvol-1
spec:
 storageClassName: gluster-heketi
 accessModes:
  - ReadWriteMany
 resources:
   requests:
     storage: 10Gi
```

[ gfs-client.yml ]
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gfs-client
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ubuntu
  template:
    metadata:
      labels:
        app: ubuntu
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        volumeMounts:
        - name: gfs
          mountPath: "/mnt"
        command: ["/usr/bin/tail","-f","/dev/null"]
      volumes:
      - name: gfs
        persistentVolumeClaim:
          claimName: gvol-1
```

```bash
# ping 172.20.1.20
PING 172.20.1.20 (172.20.1.20) 56(84) bytes of data.
64 bytes from 172.20.1.20: icmp_seq=1 ttl=63 time=0.983 ms
64 bytes from 172.20.1.20: icmp_seq=2 ttl=63 time=0.927 ms
64 bytes from 172.20.1.20: icmp_seq=3 ttl=63 time=0.942 ms
64 bytes from 172.20.1.20: icmp_seq=4 ttl=63 time=1.37 ms
^C
--- 172.20.1.20 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 0.927/1.057/1.376/0.185 ms

# kubectl apply -f gfs-sc.yml
storageclass.storage.k8s.io/gluster-heketi created

# kubectl get sc
NAME             PROVISIONER               RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
gluster-heketi   kubernetes.io/glusterfs   Delete          Immediate           false                  15s

# kubectl apply -f gfs-pvc.yml 
persistentvolumeclaim/gvol-1 created

# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS     REASON   AGE
pvc-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9   10Gi       RWX            Delete           Bound    default/gvol-1   gluster-heketi            5s

# kubectl get pvc
NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
gvol-1   Bound    pvc-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9   10Gi       RWX            gluster-heketi   13s

# kubectl apply -f gfs-client.yml 
deployment.apps/gfs-client created

# kubectl get po
NAME                         READY   STATUS              RESTARTS   AGE
gfs-client-ddfc99bb7-gn4p5   0/1     ContainerCreating   0          2m17s
gfs-client-ddfc99bb7-mzcd8   0/1     ContainerCreating   0          2m17s

# kubectl get all
NAME                             READY   STATUS              RESTARTS   AGE
pod/gfs-client-ddfc99bb7-s2782   0/1     ContainerCreating   0          10m
pod/gfs-client-ddfc99bb7-v2pw7   0/1     ContainerCreating   0          10m

NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/glusterfs-dynamic-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9   ClusterIP   10.96.127.111   <none>        1/TCP     12m
service/kubernetes                                               ClusterIP   10.96.0.1       <none>        443/TCP   7d

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gfs-client   0/2     2            0           10m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/gfs-client-ddfc99bb7   2         2         0       10m

# kubectl logs gfs-client-ddfc99bb7-mzcd8
Error from server (BadRequest): container "ubuntu" in pod "gfs-client-ddfc99bb7-mzcd8" is waiting to start: ContainerCreating

# kubectl describe pod/gfs-client-ddfc99bb7-s2782
Name:           gfs-client-ddfc99bb7-s2782
Namespace:      default
Priority:       0
Node:           w3-k8s/192.168.1.103
Start Time:     Wed, 29 Dec 2021 10:34:32 +0900
Labels:         app=ubuntu
                pod-template-hash=ddfc99bb7
Annotations:    <none>
Status:         Pending
IP:             
IPs:            <none>
Controlled By:  ReplicaSet/gfs-client-ddfc99bb7
Containers:
  ubuntu:
    Container ID:  
    Image:         ubuntu
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/bin/tail
      -f
      /dev/null
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /mnt from gfs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9jvzm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  gfs:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  gvol-1
    ReadOnly:   false
  default-token-9jvzm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9jvzm
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason       Age                      From             Message
  ----     ------       ----                     ----             -------
  Warning  FailedMount  38m (x83 over 5h37m)     kubelet, w3-k8s  Unable to attach or mount volumes: unmounted volumes=[gfs], unattached volumes=[gfs default-token-9jvzm]: timed out waiting for the condition
  Warning  FailedMount  8m24s (x22 over 5h32m)   kubelet, w3-k8s  Unable to attach or mount volumes: unmounted volumes=[gfs], unattached volumes=[default-token-9jvzm gfs]: timed out waiting for the condition
  Warning  FailedMount  3m53s (x199 over 5h37m)  kubelet, w3-k8s  (combined from similar events): Unable to attach or mount volumes: unmounted volumes=[gfs], unattached volumes=[default-token-9jvzm gfs]: timed out waiting for the condition

# kubectl describe pvc gvol-1
Name:          gvol-1
Namespace:     default
StorageClass:  gluster-heketi
Status:        Bound
Volume:        pvc-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/glusterfs
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Mounted By:    gfs-client-ddfc99bb7-s2782
               gfs-client-ddfc99bb7-v2pw7

# kubectl describe sc gluster-heketi
Name:            gluster-heketi
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"gluster-heketi"},"parameters":{"resturl":"http://172.20.1.20:8080","restuser":"admin","restuserkey":"admin"},"provisioner":"kubernetes.io/glusterfs"}

Provisioner:           kubernetes.io/glusterfs
Parameters:            resturl=http://172.20.1.20:8080,restuser=admin,restuserkey=admin
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>

# kubectl describe pv pvc-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9
Name:            pvc-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9
Labels:          <none>
Annotations:     Description: Gluster-Internal: Dynamically provisioned PV
                 gluster.kubernetes.io/heketi-volume-id: 86c6c76cc5946a8ed404be5b492a5cd1
                 gluster.org/type: file
                 kubernetes.io/createdby: heketi-dynamic-provisioner
                 pv.beta.kubernetes.io/gid: 2000
                 pv.kubernetes.io/bound-by-controller: yes
                 pv.kubernetes.io/provisioned-by: kubernetes.io/glusterfs
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    gluster-heketi
Status:          Bound
Claim:           default/gvol-1
Reclaim Policy:  Delete
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        10Gi
Node Affinity:   <none>
Message:         
Source:
    Type:                Glusterfs (a Glusterfs mount on the host that shares a pod's lifetime)
    EndpointsName:       glusterfs-dynamic-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9
    EndpointsNamespace:  default
    Path:                vol_86c6c76cc5946a8ed404be5b492a5cd1
    ReadOnly:            false
Events:                  <none>  
```


[ sudo yum install -y glusterfs-fuse 이후 ]
```bash
# kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/gfs-client-ddfc99bb7-gt857   1/1     Running   0          14s
pod/gfs-client-ddfc99bb7-m79gd   1/1     Running   0          14s

NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/glusterfs-dynamic-bf9a73cb-b4b0-4fdb-9eff-054f1ca477d9   ClusterIP   10.96.127.111   <none>        1/TCP     6h41m
service/kubernetes                                               ClusterIP   10.96.0.1       <none>        443/TCP   7d6h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gfs-client   2/2     2            2           14s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/gfs-client-ddfc99bb7   2         2         2       14s

# kubectl describe pod/gfs-client-ddfc99bb7-gt857 
Name:         gfs-client-ddfc99bb7-gt857
Namespace:    default
Priority:     0
Node:         w2-k8s/192.168.1.102
Start Time:   Wed, 29 Dec 2021 17:13:36 +0900
Labels:       app=ubuntu
              pod-template-hash=ddfc99bb7
Annotations:  cni.projectcalico.org/podIP: 172.16.103.134/32
Status:       Running
IP:           172.16.103.134
IPs:
  IP:           172.16.103.134
Controlled By:  ReplicaSet/gfs-client-ddfc99bb7
Containers:
  ubuntu:
    Container ID:  docker://9e2e2f59e8961464a7fcd515a4da845c4e1280d4b4dc0576ecd4807688c2254a
    Image:         ubuntu
    Image ID:      docker-pullable://docker.io/ubuntu@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/bin/tail
      -f
      /dev/null
    State:          Running
      Started:      Wed, 29 Dec 2021 17:13:41 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /mnt from gfs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9jvzm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  gfs:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  gvol-1
    ReadOnly:   false
  default-token-9jvzm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9jvzm
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  4m5s  default-scheduler  Successfully assigned default/gfs-client-ddfc99bb7-gt857 to w2-k8s
  Normal  Pulling    4m4s  kubelet, w2-k8s    Pulling image "ubuntu"
  Normal  Pulled     4m    kubelet, w2-k8s    Successfully pulled image "ubuntu"
  Normal  Created    4m    kubelet, w2-k8s    Created container ubuntu
  Normal  Started    4m    kubelet, w2-k8s    Started container ubuntu

# kubectl exec -it gfs-client-ddfc99bb7-gt857 sh 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.

# df -h
Filesystem                                        Size  Used Avail Use% Mounted on
overlay                                            37G  2.7G   35G   8% /
tmpfs                                             1.3G     0  1.3G   0% /dev
tmpfs                                             1.3G     0  1.3G   0% /sys/fs/cgroup
172.20.1.23:vol_86c6c76cc5946a8ed404be5b492a5cd1   10G  146M  9.9G   2% /mnt
/dev/mapper/centos_k8s-root                        37G  2.7G   35G   8% /etc/hosts
shm                                                64M     0   64M   0% /dev/shm
tmpfs                                             1.3G   12K  1.3G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                             1.3G     0  1.3G   0% /proc/acpi
tmpfs                                             1.3G     0  1.3G   0% /proc/scsi
tmpfs                                             1.3G     0  1.3G   0% /sys/firmware  

# ls -lR / > /mnt/test.data
ls: reading directory '/proc/1/map_files': Operation not permitted
ls: reading directory '/proc/12/map_files': Operation not permitted
ls: reading directory '/proc/6/map_files': Operation not permitted

# md5sum /mnt/test.data
db145e5c18a161fb1e076e4db6421747  /mnt/test.data

# exit

# kubectl exec -it gfs-client-ddfc99bb7-m79gd bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
groups: cannot find name for group ID 2000

# df -h
Filesystem                                        Size  Used Avail Use% Mounted on
overlay                                            37G  2.7G   35G   8% /
tmpfs                                             1.3G     0  1.3G   0% /dev
tmpfs                                             1.3G     0  1.3G   0% /sys/fs/cgroup
172.20.1.23:vol_86c6c76cc5946a8ed404be5b492a5cd1   10G  147M  9.9G   2% /mnt
/dev/mapper/centos_k8s-root                        37G  2.7G   35G   8% /etc/hosts
shm                                                64M     0   64M   0% /dev/shm
tmpfs                                             1.3G   12K  1.3G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                             1.3G     0  1.3G   0% /proc/acpi
tmpfs                                             1.3G     0  1.3G   0% /proc/scsi
tmpfs                                             1.3G     0  1.3G   0% /sys/firmware

root@gfs-client-ddfc99bb7-m79gd:/# md5sum /mnt/test.data 
db145e5c18a161fb1e076e4db6421747  /mnt/test.data
root@gfs-client-ddfc99bb7-m79gd:/# exit
```
Work Node에서 `sudo yum install -y glusterfs-fuse` 패키지를 설치해야 된다.

~~`sudo yum install -y nfs-utils nfs-utils-lib`의 영향을 받는지는 모르겠다.~~

2개의 Pod가 /mnt폴더를 공유하고 한쪽 Pod에서 파일을 생성해도 다른쪽 Pod도 동일한 파일에 접근이 가능하다. 즉 스토리지가 공유되고 있다.

