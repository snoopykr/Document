# Vagrant

## Vagrant 시작

```bash
# vagrant init
```

[ Vagrantfile ]
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "sysnet4admin/CentOS-k8s"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

```
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["startvm", "e2a9f053-1bfd-44e9-bdae-65e347421142", "--type", "headless"]

Stderr: VBoxManage: error: The virtual machine 'Vagrant_default_1639445444446_76600' has terminated unexpectedly during startup with exit code 1 (0x1)
VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component MachineWrap, interface IMachine
```
macOS의 경우 `시스템 환경설정 - 보안 및 개인 정보 보호`에서 권한 설정 필요하다.

```
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

The error output from the command was:

mount: unknown filesystem type 'vboxsf'
```
VirtualBox Guest Additions 설치를 하지 않아서 발생한다. 무시해도 무관하다.


```bash
# vagrant up

# vagrant ssh

[vagrant@k8s ~]# ls -al
합계 16
drwx------. 3 vagrant vagrant  95  9월 15  2019 .
drwxr-xr-x. 3 root    root     21  9월 15  2019 ..
-rw-------. 1 vagrant vagrant  28 12월 24  2019 .bash_history
-rw-r--r--. 1 vagrant vagrant  18 10월 31  2018 .bash_logout
-rw-r--r--. 1 vagrant vagrant 193 10월 31  2018 .bash_profile
-rw-r--r--. 1 vagrant vagrant 231 10월 31  2018 .bashrc
drwx------. 2 vagrant vagrant  29 12월 14 10:41 .ssh

[vagrant@k8s ~]# uptime
 11:45:13 up 18 min,  1 user,  load average: 0.00, 0.01, 0.01

[vagrant@k8s ~]# date
2021. 12. 14. (화) 11:46:11 KST

[vagrant@k8s ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)

[vagrant@k8s ~]# exit
logout
Connection to 127.0.0.1 closed.

# vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
vagrant의 간단한 구동 및 제거 방법이다.

## 가상 머신 설정 구성

[ Vagrantfile ]
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config| 
  config.vm.define "m-k8s" do |cfg|
    cfg.vm.box = "sysnet4admin/CentOS-k8s"
    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "m-k8s(github_SysNet4Admin)"
      vb.cpus = 2
      vb.memory = 2048
      vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
    end
    cfg.vm.host_name = "m-k8s"
    cfg.vm.network "private_network", ip: "192.168.1.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
    cfg.vm.synced_folder "../data", "/vagrant", disabled: true
  end
end
```

```bash
# vagrant up

# vagrant port
    22 (guest) => 60010 (host)

# netstat -an | findstr 60010
  TCP    0.0.0.0:60010          0.0.0.0:0              LISTENING

# vagrant ssh

[vagrant@m-k8s ~]# ip addr show eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:bd:18:28 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.1.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febd:1828/64 scope link
       valid_lft forever preferred_lft forever

[vagrant@m-k8s ~]# exit
logout
Connection to 127.0.0.1 closed.
```

## 가상 머신에 추가 패키지 설치

[ Vagrantfile ]
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config| 
  config.vm.define "m-k8s" do |cfg|
    cfg.vm.box = "sysnet4admin/CentOS-k8s"
    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "m-k8s(github_SysNet4Admin)"
      vb.cpus = 2
      vb.memory = 2048
      vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
    end
    cfg.vm.host_name = "m-k8s"
    cfg.vm.network "private_network", ip: "192.168.1.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
    cfg.vm.synced_folder "../data", "/vagrant", disabled: true   
    cfg.vm.provision "shell", path: "install_pkg.sh" #add provisioning script
  end
end
```

[ install_pkg.sh ]
```bash
#!/usr/bin/env bash
# install packages 
yum install epel-release -y
yum install vim-enhanced -y
```

```bash
# vagrant provision

# vagrant ssh

[vagrant@m-k8s ~]# yum repolist
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.kakao.com
 * epel: ftp.riken.jp
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
repo id                                   repo name                                                               status
base/7/x86_64                             CentOS-7 - Base                                                         10,072
epel/x86_64                               Extra Packages for Enterprise Linux 7 - x86_64                          13,691
extras/7/x86_64                           CentOS-7 - Extras                                                          500
updates/7/x86_64                          CentOS-7 - Updates                                                       3,190
repolist: 27,453

[vagrant@m-k8s ~]# vi .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions

[vagrant@m-k8s ~]# exit
logout
Connection to 127.0.0.1 closed.
```

## 가상 머신 추가 구성

[ Vagrantfile ]
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config| 
  config.vm.define "m-k8s" do |cfg|
    cfg.vm.box = "sysnet4admin/CentOS-k8s"
    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "m-k8s(github_SysNet4Admin)"
      vb.cpus = 2
      vb.memory = 2048
      vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
    end
    cfg.vm.host_name = "m-k8s"
    cfg.vm.network "private_network", ip: "192.168.1.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
    cfg.vm.synced_folder "../data", "/vagrant", disabled: true   
    cfg.vm.provision "shell", path: "install_pkg.sh"
    cfg.vm.provision "file", source: "ping_2_nds.sh", destination: "ping_2_nds.sh"
    cfg.vm.provision "shell", path: "config.sh"
  end
  
  #=============#
  # Added Nodes #
  #=============#

  (1..3).each do |i|
    config.vm.define "w#{i}-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "w#{i}-k8s(github_SysNet4Admin)"
        vb.cpus = 1
        vb.memory = 1024
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "w#{i}-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10#{i}"
      cfg.vm.network "forwarded_port", guest: 22, host: "6010#{i}",auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true
      cfg.vm.provision "shell", path: "install_pkg.sh"
    end
  end
end
```

[ install_pkg.sh ]
```bash
#!/usr/bin/env bash
# install packages 
yum install epel-release -y
yum install vim-enhanced -y
```

[ ping_2_nds.sh ]
```bash
# ping 3 times per nodes
ping 192.168.1.101 -c 3
ping 192.168.1.102 -c 3
ping 192.168.1.103 -c 3
```

[ config.sh ]
```bash
#!/usr/bin/env bash
# modify permission  
chmod 744 ./ping_2_nds.sh
```

```bash
# vagrant up

# vagrant ssh
This command requires a specific VM name to target in a multi-VM environment.

# vagrant ssh m-k8s

[vagrant@m-k8s ~]# ./ping_2_nds.sh
\PING 192.168.1.101 (192.168.1.101) 56(84) bytes of data.
64 bytes from 192.168.1.101: icmp_seq=1 ttl=64 time=1.46 ms
64 bytes from 192.168.1.101: icmp_seq=2 ttl=64 time=1.38 ms
64 bytes from 192.168.1.101: icmp_seq=3 ttl=64 time=1.93 ms

--- 192.168.1.101 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2032ms
rtt min/avg/max/mdev = 1.381/1.592/1.932/0.242 ms
PING 192.168.1.102 (192.168.1.102) 56(84) bytes of data.
64 bytes from 192.168.1.102: icmp_seq=1 ttl=64 time=1.16 ms
64 bytes from 192.168.1.102: icmp_seq=2 ttl=64 time=0.480 ms
64 bytes from 192.168.1.102: icmp_seq=3 ttl=64 time=2.22 ms

--- 192.168.1.102 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.480/1.289/2.225/0.718 ms
PING 192.168.1.103 (192.168.1.103) 56(84) bytes of data.
64 bytes from 192.168.1.103: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.1.103: icmp_seq=2 ttl=64 time=2.14 ms
64 bytes from 192.168.1.103: icmp_seq=3 ttl=64 time=2.18 ms

--- 192.168.1.103 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2021ms
rtt min/avg/max/mdev = 1.232/1.853/2.186/0.442 ms
```

[ SecureCRT ]
```
m-k8s  127.0.0.1 60010 root vagrant
w1-k8s 127.0.0.1 60101 root vagrant
w2-k8s 127.0.0.1 60102 root vagrant
w3-k8s 127.0.0.1 60103 root vagrant
```

## Kubernetes 구성

[ Vagrantfile ]
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  N = 3 # max number of worker nodes
  Ver = '1.18.4' # Kubernetes Version to install

  #=============#
  # Master Node #
  #=============#

    config.vm.define "m-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "m-k8s(github_SysNet4Admin)"
        vb.cpus = 2
        vb.memory = 3072
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SgMST-1.13.1(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "m-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10"
      cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true 
      cfg.vm.provision "shell", path: "config.sh", args: N
      cfg.vm.provision "shell", path: "install_pkg.sh", args: [ Ver, "Main" ]
      cfg.vm.provision "shell", path: "master_node.sh"
    end

  #==============#
  # Worker Nodes #
  #==============#

  (1..N).each do |i|
    config.vm.define "w#{i}-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "w#{i}-k8s(github_SysNet4Admin)"
        vb.cpus = 1
        vb.memory = 2560
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SgMST-1.13.1(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "w#{i}-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10#{i}"
      cfg.vm.network "forwarded_port", guest: 22, host: "6010#{i}", auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true
      cfg.vm.provision "shell", path: "config.sh", args: N
      cfg.vm.provision "shell", path: "install_pkg.sh", args: Ver
      cfg.vm.provision "shell", path: "work_nodes.sh"
    end
  end
end
```

[ config.sh ]
```bash
#!/usr/bin/env bash

# vim configuration 
echo 'alias vi=vim' >> /etc/profile

# swapoff -a to disable swapping
swapoff -a
# sed to comment the swap partition in /etc/fstab
sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab

# kubernetes repo
gg_pkg="packages.cloud.google.com/yum/doc" # Due to shorten addr for key
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://${gg_pkg}/yum-key.gpg https://${gg_pkg}/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# RHEL/CentOS 7 have reported traffic issues being routed incorrectly due to iptables bypassed
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
modprobe br_netfilter

# local small dns & vagrant cannot parse and delivery shell code.
echo "192.168.1.10 m-k8s" >> /etc/hosts
for (( i=1; i<=$1; i++  )); do echo "192.168.1.10$i w$i-k8s" >> /etc/hosts; done

# config DNS  
cat <<EOF > /etc/resolv.conf
nameserver 1.1.1.1 #cloudflare DNS
nameserver 8.8.8.8 #Google DNS
EOF
```

[ install_pkg.sh ]
```bash
#!/usr/bin/env bash

# install packages 
yum install epel-release -y
yum install vim-enhanced -y
yum install git -y

# install docker 
yum install docker -y && systemctl enable --now docker

# install kubernetes cluster 
yum install kubectl-$1 kubelet-$1 kubeadm-$1 -y
systemctl enable --now kubelet

# git clone _Book_k8sInfra.git 
if [ $2 = 'Main' ]; then
  git clone https://github.com/sysnet4admin/_Book_k8sInfra.git
  mv /home/vagrant/_Book_k8sInfra $HOME
  find $HOME/_Book_k8sInfra/ -regex ".*\.\(sh\)" -exec chmod 700 {} \;
fi
```

[ master_node.sh ]
```bash
#!/usr/bin/env bash

# init kubernetes 
kubeadm init --token 123456.1234567890123456 --token-ttl 0 \
--pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.1.10 

# config for master node only 
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# config for kubernetes's network 
kubectl apply -f \
https://raw.githubusercontent.com/sysnet4admin/IaC/master/manifests/172.16_net_calico.yaml
```

[ work_nodes.sh ]
```bash
#!/usr/bin/env bash

# config for work_nodes only 
kubeadm join --token 123456.1234567890123456 \
             --discovery-token-unsafe-skip-ca-verification 192.168.1.10:6443
```

```bash
# vagrant up

// ScureCRT at master
[root@m-k8s ~]# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
m-k8s    Ready    master   8m38s   v1.18.4
w1-k8s   Ready    <none>   6m12s   v1.18.4
w2-k8s   Ready    <none>   3m35s   v1.18.4
w3-k8s   Ready    <none>   42s     v1.18.4

[root@m-k8s ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-99c9b6f64-ms2ml   1/1     Running   0          9m4s
kube-system   calico-node-58qp6                         1/1     Running   0          6m57s
kube-system   calico-node-fpkts                         1/1     Running   0          4m20s
kube-system   calico-node-gc62d                         1/1     Running   0          87s
kube-system   calico-node-xc8pm                         1/1     Running   0          9m5s
kube-system   coredns-66bff467f8-4pvhm                  1/1     Running   0          9m4s
kube-system   coredns-66bff467f8-bj6n8                  1/1     Running   0          9m4s
kube-system   etcd-m-k8s                                1/1     Running   0          9m19s
kube-system   kube-apiserver-m-k8s                      1/1     Running   0          9m19s
kube-system   kube-controller-manager-m-k8s             1/1     Running   0          9m19s
kube-system   kube-proxy-7jlvb                          1/1     Running   0          4m20s
kube-system   kube-proxy-8m6xp                          1/1     Running   0          6m57s
kube-system   kube-proxy-mkjz4                          1/1     Running   0          87s
kube-system   kube-proxy-wvcfg                          1/1     Running   0          9m5s
kube-system   kube-scheduler-m-k8s                      1/1     Running   0          9m19s
```
Kubernetes의 실습 환경이 버추얼 머신을 사용해서 구축이 되었다.