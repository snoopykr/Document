# 구성요소 기능 검증

```bash
# vagrant up
```
master, node1, node2, node3을 구동시켜 준다.

## kubectl

[ SecureCRT at node3 ]
```bash
[root@w3-k8s ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?

[root@w3-k8s ~]# scp root@192.168.1.10:/etc/kubernetes/admin.conf .
The authenticity of host '192.168.1.10 (192.168.1.10)' can't be established.
ECDSA key fingerprint is SHA256:l6XikZFgOibzSygqZ6+UYHUnEmjFEFhx7PpZw0I3WaM.
ECDSA key fingerprint is MD5:09:74:43:ef:38:3e:36:a1:7e:51:76:1a:ac:2d:7e:0c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.10' (ECDSA) to the list of known hosts.
root@192.168.1.10's password: 
admin.conf                                                                                        100% 5452     1.1MB/s   00:00    

[root@w3-k8s ~]# kubectl get nodes --kubeconfig admin.conf
NAME     STATUS   ROLES    AGE   VERSION
m-k8s    Ready    master   20h   v1.18.4
w1-k8s   Ready    <none>   20h   v1.18.4
w2-k8s   Ready    <none>   20h   v1.18.4
w3-k8s   Ready    <none>   20h   v1.18.4
```
master가 아닌 node3에서 `kubectl get nodes`를 실행하면 원하는 결과를 얻을수 없다. 

## kubelet

[ nginx-pod.yaml ]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: container-name
    image: nginx
```

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# kubectl create -f nginx-pod.yaml 
pod/nginx-pod created

[root@m-k8s 3.1.6]# kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          10s

[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          14s   172.16.132.4   w3-k8s   <none>           <none>
```

[ SecureCRT at node3 ]
```bash
[root@w3-k8s ~]# systemctl stop kubelet                            
```
pod가 실행되는 노드에서 kubelet 서비스를 죽인다.

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          2m28s

[root@m-k8s 3.1.6]# kubectl delete pod nginx-pod
pod "nginx-pod" deleted
^C

[root@m-k8s 3.1.6]# kubectl get pod             
NAME        READY   STATUS        RESTARTS   AGE
nginx-pod   1/1     Terminating   0          3m13s
```
node3의 kubelet이 죽어 있는 상태이므로 nginx-pod가 삭제가 되지 않고 시간만 경과 되고 STATUS는 `Terminating`으로 변경된다.

[ SecureCRT at node3 ]
```bash
[root@w3-k8s ~]# systemctl start kubelet                            
```
kubelet을 정상 작동을 시키지만...

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# kubectl get pod
No resources found in default namespace.
```
kubelet에 문제가 발생되어 제대로 작동이 되지 않는다.

## kube-proxy

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# kubectl create -f nginx-pod.yaml 
pod/nginx-pod created

[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          37s   172.16.221.129   w1-k8s   <none>           <none>

[root@m-k8s 3.1.6]# curl 172.16.221.129
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

[ SecureCRT at node1 ]
```bash
[root@w1-k8s ~]# modprobe -r br_netfilter

[root@w1-k8s ~]# systemctl restart network
```
pod가 실행되는 node1에서 network관련 사항을 재 구동한다.

참고로 Vagrantfile에서 사용되는 config.sh에<br><br>
net.bridge.bridge-nf-call-ip6tables = 1<br>
net.bridge.bridge-nf-call-iptables = 1<br>
EOF<br>
modprobe br_netfilter<br><br>
br_netfilter 커널 모듈을 적재하고 iptables를 거쳐 통신하도록 설정되었다.

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# curl 172.16.221.129
curl: (7) Failed connect to 172.16.221.129:80; Connection timed out

[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          23m   172.16.221.129   w1-k8s   <none>           <none>
```
pod는 정상 작동하는 것처럼 보이지만 통신은 연결이 되지 않는 상태이다.

[ SecureCRT at node1 ]
```bash
[root@w1-k8s ~]# modprobe br_netfilter

[root@w1-k8s ~]# reboot
```
br_netfilter 모듈을 적재하고 node1를 reboot한다.

[ SecureCRT at master ]
```bash
[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME        READY   STATUS      RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx-pod   0/1     Completed   0          27m   <none>   w1-k8s   <none>           <none>

[root@m-k8s 3.1.6]# kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   1          27m   172.16.221.130   w1-k8s   <none>           <none>

[root@m-k8s 3.1.6]# curl 172.16.221.130
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
node1은 reboot를 종료하면 pod를 실행하게 된다. RESTARTS가 1로 바뀌었고 IP 또한 변경되었다.