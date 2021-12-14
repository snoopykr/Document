# Vagrant

$ vagrant init

vagrantfile

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

```ruby
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["startvm", "e2a9f053-1bfd-44e9-bdae-65e347421142", "--type", "headless"]

Stderr: VBoxManage: error: The virtual machine 'Vagrant_default_1639445444446_76600' has terminated unexpectedly during startup with exit code 1 (0x1)
VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component MachineWrap, interface IMachine
```

시스템 환경설정 - 보안 및 개인 정보 보호 : 권한 설정 필요

```ruby
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

무시...


```bash
snoopy_kr@Leeui-MacBookPro Vagrant % vagrant ssh

[vagrant@k8s ~]$ ls -al
합계 16
drwx------. 3 vagrant vagrant  95  9월 15  2019 .
drwxr-xr-x. 3 root    root     21  9월 15  2019 ..
-rw-------. 1 vagrant vagrant  28 12월 24  2019 .bash_history
-rw-r--r--. 1 vagrant vagrant  18 10월 31  2018 .bash_logout
-rw-r--r--. 1 vagrant vagrant 193 10월 31  2018 .bash_profile
-rw-r--r--. 1 vagrant vagrant 231 10월 31  2018 .bashrc
drwx------. 2 vagrant vagrant  29 12월 14 10:41 .ssh

[vagrant@k8s ~]$ uptime
 11:45:13 up 18 min,  1 user,  load average: 0.00, 0.01, 0.01

[vagrant@k8s ~]$ date
2021. 12. 14. (화) 11:46:11 KST

[vagrant@k8s ~]$ cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)

[vagrant@k8s ~]$ exit
logout
Connection to 127.0.0.1 closed.

snoopy_kr@Leeui-MacBookPro Vagrant % vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```