Kubernetes Install (Ubuntu 20.04)



[ 참고 ]
https://medium.com/finda-tech/overview-8d169b2a54ff



Virtual Machine 사용시 Network는 Bridge로 구성



$ sudo swapoff -a

$ sudo vi /etc/fstab



$ docker info

$ sudo vi /etc/docker/daemon.json
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



[ 버전 확인 ]
https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages

$ wget https://packages.cloud.google.com/apt/pool/cri-tools_1.19.0-00_amd64_b6fdfd86c8a3665ab10b9bd9565354977cd5abbaefeb2ee953bc4a13fe7d3326.deb

$ wget https://packages.cloud.google.com/apt/pool/kubeadm_1.23.1-00_amd64_6bc970cf9bf5349ba18526f77c6ac16caf2a52b6a7b0e40753541ebef52ad99f.deb

$ wget https://packages.cloud.google.com/apt/pool/kubectl_1.23.1-00_amd64_369729660de023320c13ce87a05218dee845e08741bd549f2ef434a40a2d50d8.deb

$ wget https://packages.cloud.google.com/apt/pool/kubelet_1.23.1-00_amd64_4bb6c13d6a2cccc2c70f8c9bc5bc8daf519d2ea51cf33e8329fa355bada1a1d7.deb

$ wget https://packages.cloud.google.com/apt/pool/kubernetes-cni_0.8.7-00_amd64_ca2303ea0eecadf379c65bad855f9ad7c95c16502c0e7b3d50edcb53403c500f.deb



$ sudo apt-get install socat conntrack ebtables

$ sudo dpkg --install ./kubernetes-cni_0.8.7-00_amd64_ca2303ea0eecadf379c65bad855f9ad7c95c16502c0e7b3d50edcb53403c500f.deb 

$ sudo dpkg --install ./kubelet_1.23.1-00_amd64_4bb6c13d6a2cccc2c70f8c9bc5bc8daf519d2ea51cf33e8329fa355bada1a1d7.deb 

$ sudo dpkg --install ./cri-tools_1.19.0-00_amd64_b6fdfd86c8a3665ab10b9bd9565354977cd5abbaefeb2ee953bc4a13fe7d3326.deb 

$ sudo dpkg --install ./kubectl_1.23.1-00_amd64_369729660de023320c13ce87a05218dee845e08741bd549f2ef434a40a2d50d8.deb

$ sudo dpkg --install ./kubeadm_1.23.1-00_amd64_6bc970cf9bf5349ba18526f77c6ac16caf2a52b6a7b0e40753541ebef52ad99f.deb 



$ ip add show

$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.0.40



$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config



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



// 한번 실행
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

$ kubectl apply -f kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created



