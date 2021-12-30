# Kubernetes Install (Ubuntu 20.04)

[ 참고 ]
https://medium.com/finda-tech/overview-8d169b2a54ff

`Virtual Machine 사용시 Network는 Bridge로 설정`

## 스왑 메모리 비활성화

```bash
$ sudo swapoff -a

$ sudo vi /etc/fstab

$ sudo cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Sun Sep 15 07:11:23 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos_k8s-root /                       xfs     defaults        0 0
UUID=cb32cbce-c213-4954-83da-0f8fd4e96a8f /boot                   xfs     defaults        0 0
#/dev/mapper/centos_k8s-swap swap                    swap    defaults        0 0
```

## Docker 데몬 드라이버 변경

```bash
$ docker info
// [ 생략 ]
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: journald
Cgroup Driver: cgroupfs
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
// [ 생략 ]

$ sudo vi /etc/docker/daemon.json

$ sudo cat /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
}

$ sudo mkdir -p /etc/systemd/system/docker.service.d

$ sudo systemctl daemon-reload

$ sudo systemctl restart docker

$ docker info
// [ 생략 ]
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: journald
Cgroup Driver: systemd
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
// [ 생략 ]
```

## Kubernetes 설치

[ 버전 확인 ]
https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages

```bash
// 다운로드
$ wget https://packages.cloud.google.com/apt/pool/cri-tools_1.19.0-00_amd64_b6fdfd86c8a3665ab10b9bd9565354977cd5abbaefeb2ee953bc4a13fe7d3326.deb

$ wget https://packages.cloud.google.com/apt/pool/kubeadm_1.23.1-00_amd64_6bc970cf9bf5349ba18526f77c6ac16caf2a52b6a7b0e40753541ebef52ad99f.deb

$ wget https://packages.cloud.google.com/apt/pool/kubectl_1.23.1-00_amd64_369729660de023320c13ce87a05218dee845e08741bd549f2ef434a40a2d50d8.deb

$ wget https://packages.cloud.google.com/apt/pool/kubelet_1.23.1-00_amd64_4bb6c13d6a2cccc2c70f8c9bc5bc8daf519d2ea51cf33e8329fa355bada1a1d7.deb

$ wget https://packages.cloud.google.com/apt/pool/kubernetes-cni_0.8.7-00_amd64_ca2303ea0eecadf379c65bad855f9ad7c95c16502c0e7b3d50edcb53403c500f.deb

// 설치
$ sudo apt-get install socat conntrack ebtables

$ sudo dpkg --install ./kubernetes-cni_0.8.7-00_amd64_ca2303ea0eecadf379c65bad855f9ad7c95c16502c0e7b3d50edcb53403c500f.deb 

$ sudo dpkg --install ./kubelet_1.23.1-00_amd64_4bb6c13d6a2cccc2c70f8c9bc5bc8daf519d2ea51cf33e8329fa355bada1a1d7.deb 

$ sudo dpkg --install ./cri-tools_1.19.0-00_amd64_b6fdfd86c8a3665ab10b9bd9565354977cd5abbaefeb2ee953bc4a13fe7d3326.deb 

$ sudo dpkg --install ./kubectl_1.23.1-00_amd64_369729660de023320c13ce87a05218dee845e08741bd549f2ef434a40a2d50d8.deb

$ sudo dpkg --install ./kubeadm_1.23.1-00_amd64_6bc970cf9bf5349ba18526f77c6ac16caf2a52b6a7b0e40753541ebef52ad99f.deb 
```
일반적으로 Repository를 추가하고 설치를 하라고 하지만 간단히 버전 확인 후 직접 설치도 가능하다.

## Kubeadm 실행

```bash
$ ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:0f:f7:2d brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 65123sec preferred_lft 65123sec
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:bf:c2:55 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:9f:eb:83:4c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
5: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1440 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 172.16.171.64/32 brd 172.16.171.64 scope global tunl0
       valid_lft forever preferred_lft forever
6: caliebcab1c12ce@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 0
7: cali4c2064cf6dd@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 1
8: caliceac7facb7a@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1440 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 2

$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.1.10

// Kubeadm에서 제시하는 명령어 Master Node에서 실행
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

// Kubeadm에서 제시하는 명령어 각 Worker Node에서 실행
$ sudo kubeadm join 192.168.1.10:6443 --token t4tcwj.22xh9lzstu56qyrb \
    --discovery-token-ca-cert-hash sha256:eb3765b58c9140c9a89daf7ea21444ca44a142939ebb93aedad1ebc03202a1d7

$ kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-64897985d-mrz67          1/1     Running   0          19m   10.244.0.3     master   <none>           <none>
kube-system   coredns-64897985d-p9x7z          1/1     Running   0          19m   10.244.0.2     master   <none>           <none>
kube-system   etcd-master                      1/1     Running   0          20m   192.168.0.40   master   <none>           <none>
kube-system   kube-apiserver-master            1/1     Running   0          20m   192.168.0.40   master   <none>           <none>
kube-system   kube-controller-manager-master   1/1     Running   0          20m   192.168.0.40   master   <none>           <none>
kube-system   kube-flannel-ds-ngzdt            1/1     Running   0          17m   192.168.0.41   node1    <none>           <none>
kube-system   kube-flannel-ds-pmtb8            1/1     Running   0          17m   192.168.0.43   node3    <none>           <none>
kube-system   kube-flannel-ds-xqv5j            1/1     Running   0          17m   192.168.0.42   node2    <none>           <none>
kube-system   kube-flannel-ds-zhlql            1/1     Running   0          17m   192.168.0.40   master   <none>           <none>
kube-system   kube-proxy-fx5xm                 1/1     Running   0          18m   192.168.0.43   node3    <none>           <none>
kube-system   kube-proxy-wn6xl                 1/1     Running   0          18m   192.168.0.42   node2    <none>           <none>
kube-system   kube-proxy-zfqbx                 1/1     Running   0          19m   192.168.0.40   master   <none>           <none>
kube-system   kube-proxy-zzr8p                 1/1     Running   0          19m   192.168.0.41   node1    <none>           <none>
kube-system   kube-scheduler-master            1/1     Running   0          20m   192.168.0.40   master   <none>           <none>

$ kubectl get nodes
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   21m   v1.23.1
node1    Ready    <none>                 20m   v1.23.1
node2    Ready    <none>                 19m   v1.23.1
node3    Ready    <none>                 19m   v1.23.1

// kube-flannel.yml를 다운 받기 위해 한번만 실행
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

$ kubectl apply -f kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```