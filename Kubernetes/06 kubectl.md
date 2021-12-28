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
$ kubectl apply -f pvc.yml
persistentvolumeclaim/data1 created

$ kubectl get pvc,pv
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data1   Bound    pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            standard       31s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            Delete           Bound    default/data1   standard                31s

$ kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  3d23h

$ kubectl apply -f pod.yml
pod/pod1 created

$ kubectl get pvc,pv,po
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data1   Bound    pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            standard       5m38s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-30712e65-159f-4dc2-ac1e-ff7104b44861   2Gi        RWO            Delete           Bound    default/data1   standard                5m38s

NAME       READY   STATUS    RESTARTS   AGE
pod/pod1   1/1     Running   0          15s

$ kubectl exec -it pod1 -- sh

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
$ kubectl apply -f nfs-pv.yml
persistentvolume/nfs-1 created

$ kubectl run -it bb --image=busybox sh

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

$ kubectl apply -f nfs-pvc.yml
persistentvolumeclaim/nfs-1 created

$ kubectl get pv,pvc
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/nfs-1   100Mi      RWX            Retain           Bound    default/nfs-1                           6m35s

NAME                          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nfs-1   Bound    nfs-1    100Mi      RWX                           22s

$ kubectl apply -f nfs-client.yml
deployment.apps/nfs-client created

$ kubectl get po
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
          mountPath: /mnt
        command: ["/usr/bin/tail","-f","/dev/null"]
      volumes:
      - name: gfs
        persistentVolumeClaim:
          claimName: gvol-1
```

```bash
$ ping 172.20.1.20
PING 172.20.1.20 (172.20.1.20) 56(84) bytes of data.
64 bytes from 172.20.1.20: icmp_seq=1 ttl=63 time=0.983 ms
64 bytes from 172.20.1.20: icmp_seq=2 ttl=63 time=0.927 ms
64 bytes from 172.20.1.20: icmp_seq=3 ttl=63 time=0.942 ms
64 bytes from 172.20.1.20: icmp_seq=4 ttl=63 time=1.37 ms
^C
--- 172.20.1.20 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 0.927/1.057/1.376/0.185 ms

$ kubectl apply -f gfs-sc.yml
storageclass.storage.k8s.io/gluster-heketi created

$ kubectl get sc  
NAME             PROVISIONER               RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
gluster-heketi   kubernetes.io/glusterfs   Delete          Immediate           false                  4m47s

$ kubectl apply -f gfs-pvc.yml 
persistentvolumeclaim/gvol-1 created

$ kubectl get pvc
NAME          STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS     AGE
gvol-1        Pending                                          gluster-heketi   111s
wildfly-pvc   Bound     wildfly-pv   10Ti       RWX                             6d2h

$ kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
wildfly-pv   10Ti       RWX            Retain           Bound    default/wildfly-pvc                           6d2h

$ kubectl apply -f gfs-client.yml 
deployment.apps/gfs-client created

$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
gfs-client-ddfc99bb7-4hzf9   0/1     Pending   0          35s
gfs-client-ddfc99bb7-vlc2x   0/1     Pending   0          35s


$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/gfs-client-ddfc99bb7-4hzf9   0/1     Pending   0          116s
pod/gfs-client-ddfc99bb7-vlc2x   0/1     Pending   0          116s

NAME                                                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/glusterfs-dynamic-54ead0b5-166f-4f9d-a444-5b0096e6eabe   ClusterIP   10.100.169.83   <none>        1/TCP     6m58s
service/kubernetes                                               ClusterIP   10.96.0.1       <none>        443/TCP   6d3h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gfs-client   0/2     2            0           117s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/gfs-client-ddfc99bb7   2         2         0       117s
```
문제가 있는지 Pod가 올라오지 않는다.