# docker

## docker run
| 값 | 설명 |
|---|:---:|
| `-i` | 키보드 입력을 컨테이너의 표준 입력에 연결하여 키보드 입력을 컨테이너의 셀 등에 보낸다. |
| `-t` | 터미널을 통해 대화형 조작이 가능하게 한다. |
| `-d` | 백그라운드로 컨테이너를 돌려 터미널과 연결하지 않는다. |
| `--name` | 컨테이너에 이름을 설정한다. 시스템에서 유일한 이름이어야 하며, 옵션을 생략하면 자동으로 만들어진 이름이 부여된다. |
| `--rm` | 컨테이너가 종료하면 종료 상태의 컨테이너를 자동으로 삭제한다. |

## docker stop
정상적인 종료를 실행한다. 

## docker kill
kill은 정상적인 종료가 원활하지 못한 경우 종료하기 위해 사용된다. (stop를 권장)

1번 터미널
```bash
// stop에서 사용 (6d4fd22c3865)
$ docker run  -it ubuntu bash
root@6d4fd22c3865:/# exit
```

2번 터미널
```bash
// kill에서 사용 (3e98a3014884)
$ docker run  -it ubuntu bash
root@3e98a3014884:/#
```

3번 터미널
```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS                      PORTS     NAMES
3e98a3014884   ubuntu    "bash"    9 seconds ago   Up 11 seconds                         priceless_dijkstra
6d4fd22c3865   ubuntu    "bash"    2 minutes ago   Up 7 seconds                          focused_bardeen

$ docker stop 6d4fd22c3865
6d4fd22c3865

$ docker kill 3e98a3014884
3e98a3014884

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                       PORTS     NAMES
3e98a3014884   ubuntu    "bash"    24 seconds ago   Exited (137) 3 seconds ago             priceless_dijkstra
6d4fd22c3865   ubuntu    "bash"    2 minutes ago    Exited (0) 49 seconds ago              focused_bardeen
```
STATUS 부분을 보면 Exited (137), Exited (0)로 정상 종료와 비정상 종료 상태가 다른 것을 확인할 수 있다.

## docker start
정지 상태인 컨테이너를 재기동 한다.

```bash
$ docker start -i 3e98a3014884
root@3e98a3014884:/#

// 다음을 위한 준비
root@3e98a3014884:/# apt update
// <생략>
root@3e98a3014884:/# apt install git
// <생략>
```
-i로 표준출력, 표준에러를 터메널에 표시하도록 한다.

## docker commit
새로운 도커 이미지 생성한다.

```bash
$ docker diff 3e98a3014884
// 중간 생략...
C /var/cache
C /var/cache/debconf
C /var/cache/debconf/config.dat
A /var/cache/debconf/config.dat-old
C /var/cache/debconf/templates.dat
A /var/cache/debconf/templates.dat-old
C /var/cache/ldconfig
C /var/cache/ldconfig/aux-cache

$ docker commit 3e98a3014884 ubuntu:git
sha256:99edbc97d61da96ec2c6c4a5d30fe5dac0450eacabad0b6e8c33b27a3023d74d

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
ubuntu                               git                                                     99edbc97d61d   29 seconds ago   207MB
ubuntu                               latest                                                  ba6acccedd29   7 weeks ago      72.8MB
```
diff를 사용해 변경된 내용을 확인할 수 있고, 이미지를 보면 사이즈가 변경된 것을 확인할 수 있다.

## docker push
이미지를 원격 리포지토리에 보관

```bash
$ docker login
Authenticating with existing credentials...
Login Succeeded

Logging in with your password grants your terminal complete access to your account.
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/

$ docker tag ubuntu:git welovefish/ubuntu:git

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED         SIZE
ubuntu                               git                                                     99edbc97d61d   6 minutes ago   207MB
welovefish/ubuntu                    git                                                     99edbc97d61d   6 minutes ago   207MB
ubuntu                               latest                                                  ba6acccedd29   7 weeks ago     72.8MB

$ docker push welovefish/ubuntu:git
The push refers to repository [docker.io/welovefish/ubuntu]
1b4231260725: Pushed
9f54eef41275: Mounted from library/ubuntu
git: digest: sha256:4c66d644effa1531f17dd0d5eb17d9f39306655fdc4f78c99fcf7100142de9c5 size: 741
```
welovefish는 github 계정이다. docker-desktop를 사용하면서 로그인이 되어 있다면 id, password는 입력하지 않아도 된다.

https://hub.docker.com/repositories 에서 보관된 도커 이미지를 확인할 수 있다.

## docker rm
종료된 컨테이너 제거

```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                      PORTS     NAMES
3e98a3014884   ubuntu    "bash"    40 minutes ago   Up 24 minutes                         priceless_dijkstra
6d4fd22c3865   ubuntu    "bash"    42 minutes ago   Exited (0) 40 minutes ago             focused_bardeen

$ docker rm 3e98a3014884
Error response from daemon: You cannot remove a running container 3e98a3014884c84509444285cf6791b907f75c771c35eb3d403850b57c805167. Stop the container before attempting removal or force remove

$ docker rm 6d4fd22c3865
6d4fd22c3865
```
종료가 되지 않은 컨테이너는 제거할 수 없다.

## docker rmi
이미지를 로컬 리포지터리에서 삭제

```bash
$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
ubuntu                               git                                                     99edbc97d61d   21 minutes ago   207MB
welovefish/ubuntu                    git                                                     99edbc97d61d   21 minutes ago   207MB
ubuntu                               latest                                                  ba6acccedd29   7 weeks ago      72.8MB

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                          PORTS     NAMES
3e98a3014884   ubuntu    "bash"    49 minutes ago   Exited (0) About a minute ago             priceless_dijkstra

$ docker rm 3e98a3014884
3e98a3014884

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker rmi 99edbc97d61d
Error response from daemon: conflict: unable to delete 99edbc97d61d (must be forced) - image is referenced in multiple repositories

$ docker rmi ba6acccedd29
Error response from daemon: conflict: unable to delete ba6acccedd29 (cannot be forced) - image has dependent child images

$ docker rmi -f 99edbc97d61d
Untagged: ubuntu:git
Untagged: welovefish/ubuntu:git
Untagged: welovefish/ubuntu@sha256:4c66d644effa1531f17dd0d5eb17d9f39306655fdc4f78c99fcf7100142de9c5
Deleted: sha256:99edbc97d61da96ec2c6c4a5d30fe5dac0450eacabad0b6e8c33b27a3023d74d
Deleted: sha256:723e2fb2728119f027b9386c4e4c506e35d28ae73487e24580032b8b8c4ec4cc

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
ubuntu                               latest                                                  ba6acccedd29   7 weeks ago      72.8MB
```
이미지가 삭제되지 않은 이유는 리포지토리에서 참조되거나 의존적인 경우에는 삭제를 하지 못한다. 이 경우 `-f`를 사용해서 삭제가 가능하다.

## docker inspect
도커 Object의 로우레벨 정보를 확인한다.

1번 터미널

```bash
$ docker run -it ubuntu bash
root@2aea93886f2c:/# apt update
// <생략>
root@2aea93886f2c:/# apt install -y iputils-ping net-tools
// <생략>
root@2aea93886f2c:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 14319  bytes 21132406 (21.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7699  bytes 422558 (422.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

2번 터미널

```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
2aea93886f2c   ubuntu    "bash"    4 minutes ago   Up 4 minutes                         optimistic_dubinsky

$ docker inspect --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" 2aea93886f2c
172.17.0.2

$ docker inspect --format="{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}" 2aea93886f2c
02:42:ac:11:00:02

$ docker inspect --format="{{.LogPath}}" 2aea93886f2c
/var/lib/docker/containers/2aea93886f2ce1be0b4e3b46e7fd0fe6a160314161745b17af11ac72d53df271/2aea93886f2ce1be0b4e3b46e7fd0fe6a160314161745b17af11ac72d53df271-json.log

$ docker inspect --format="{{.Config.Image}}" 2aea93886f2c
ubuntu

$ docker inspect --format="{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}" 2aea93886f2c

$ docker inspect --format="{{(index (index .NetworkSettings.Ports '8787/tcp') 0).HostPort}}" 2aea93886f2c

$ docker inspect --format="{{json .Config}}" 2aea93886f2c
{"Hostname":"2aea93886f2c","Domainname":"","User":"","AttachStdin":true,"AttachStdout":true,"AttachStderr":true,"Tty":true,"OpenStdin":true,"StdinOnce":true,"Env":["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],"Cmd":["bash"],"Image":"ubuntu","Volumes":null,"WorkingDir":"","Entrypoint":null,"OnBuild":null,"Labels":{}}
```

## docker exec
실행 중인 컨테이너에 접속하기

1번 터미널

```bash
$ docker run -it ubuntu bash

root@2aea93886f2c:/# w
 01:13:40 up 34 min,  0 users,  load average: 0.00, 0.03, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

root@2aea93886f2c:/# ps
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
  334 pts/0    00:00:00 ps
root@2aea93886f2c:/# tty
/dev/pts/0

root@2aea93886f2c:/# ps aw
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss     0:00 bash
  320 pts/1    Ss+    0:00 bash
  336 pts/0    R+     0:00 ps aw
```

2번 터미널

```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                      PORTS     NAMES
2aea93886f2c   ubuntu    "bash"    19 minutes ago   Up 19 minutes                         optimistic_dubinsky

$ docker exec -it 2aea93886f2c bash

root@2aea93886f2c:/# w
 01:12:33 up 33 min,  0 users,  load average: 0.00, 0.04, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

root@2aea93886f2c:/# ps
  PID TTY          TIME CMD
  320 pts/1    00:00:00 bash
  330 pts/1    00:00:00 ps

root@2aea93886f2c:/# tty
/dev/pts/1

root@2aea93886f2c:/# ps aw
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss+    0:00 bash
  320 pts/1    Ss     0:00 bash
  332 pts/1    R+     0:00 ps aw
```

## docker build
컨테이너 개발

Dockerfile
```dockerfile
FROM alpine:latest
RUN apk update && apk add figlet
ADD ./message /message
CMD cat /message | figlet
```

```bash
$ echo "Hello World" > message

$ cat message
Hello World

$ docker build --tag hello:1.0 .
// <생략>

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
hello                                1.0                                                     0e664f242f0d   29 seconds ago   8.55MB

$ docker run hello:1.0
 _   _      _ _        __        __         _     _
| | | | ___| | | ___   \ \      / /__  _ __| | __| |
| |_| |/ _ \ | |/ _ \   \ \ /\ / / _ \| '__| |/ _` |
|  _  |  __/ | | (_) |   \ V  V / (_) | |  | | (_| |
|_| |_|\___|_|_|\___/     \_/\_/ \___/|_|  |_|\__,_|

```

## Dockerfile
| 커멘드 | 설명 |
|---|:---:|
| FROM <이미지>[:태그] | 컨테이너의 베이스 이미지를 지정 |
| RUN <커맨드><br>RUN ["커맨드", "파라미터1", "파라미터2"] | FROM의 베이스 이미지에서 커맨드를 실행 |
| ADD <소스><컨테이너 내부 경로><br>ADD ["소스", ... "<컨테이너 내부 경로>"] | 소스(파일, 디렉토리, tar 파일, URL)를 컨테이너 내부 경로에 복사 |
| COPY <소스><컨테이너 내부 경로><br>COPY ["소스", ... "<컨테이너 내부 경로>"] | 소스(파일, 디렉토리)를 컨테이너 내부 경로에 복사 |
| ENTRYPOINT ["실행가능한 것", "파라미터1", "파라미터2"]<br>ENTRYPOINT 커맨드 파라미터1 파라미터2(셀 형식) | 컨테이너가 실행하는 파일을 설정 |
| CMD ["실행 바이너리", "파라미터1", "파라미터2"]<br>CMD <커맨드>(셀 형식)<br>CMD [파라미터1", "파라미터2"](ENTRYPOINT의 파라미터) | 컨테이너 기동 시 실행될 커맨드를 지정 |
| ENV <키> <밸류><br>ENV <키>=<밸류> ... | 환경 변수 설정 |
| EXPOSE <포트> [<포트>...] | 공개 포트 설정 |
| USER <유저명> 또는 <유아이디> | RUN, CMD, ENTRYPOINT 실행 유저 지정 |
| VOLUME ["/path"] | 공유 가능한 불륨을 마운트 |
| WORKDIR /path | RUN, CMD, ENTRYPOINT, COPY, ADD의 작업 디렉토리 지정 |
| ARG <이름>[=<디폴트 값>] | 빌드할 때 넘길 인자를 정의<br>--build-arg <변수명>=<값> |
| LABEL <키>=<밸류> <키>=<밸류> | 이미지의 메타데이터에 라벨을 추가 |
| MAINTAINER <이름>] | 이미지의 메타데이터에 저작권을 추가 |


## docker 내부 네트워크

1번 터미널

```bash
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9a5ddadbec91   bridge    bridge    local
6630cfb37ac9   host      host      local
e0f78c2f1ab0   none      null      local

$ docker network create my-network
1b1266d16d16406596c44c6b4d3fe0fe347d7d6b7bfa5b032bf11c945a63291d

$ docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
9a5ddadbec91   bridge       bridge    local
6630cfb37ac9   host         host      local
1b1266d16d16   my-network   bridge    local
e0f78c2f1ab0   none         null      local

$ docker run -d --name webserver1 --network my-network nginx:latest
7cdaee6a285a6a66e68f508d4d14e9e9fc342149a5c43b87b42c1cfdd4ab49e8

$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
7cdaee6a285a   nginx:latest   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    webserver1

$ docker run -it --rm --name net-tool --network my-network ubuntu bash
root@d86aac5008d6:/# apt-get update
// <생략>
root@d86aac5008d6:/# apt-get install dnsutils
// <생략>
root@d86aac5008d6:/# nslookup webserver1
Server:         127.0.0.11
Address:        127.0.0.11#53

Non-authoritative answer:
Name:   webserver1
Address: 172.18.0.2

root@d86aac5008d6:/# apt-get install -y curl
// <생략>

root@d86aac5008d6:/# curl http://webserver1
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

2번 터미널

```bash
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
d86aac5008d6   ubuntu         "bash"                   8 minutes ago    Up 8 minutes              net-tool
7cdaee6a285a   nginx:latest   "/docker-entrypoint.…"   20 minutes ago   Up 20 minutes   80/tcp    webserver1

$ docker commit d86aac5008d6 ubuntu:network
sha256:0e9903d314af9165138ea4c8fb26264b3b97ee44827bc052083f7be59ec465c3

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED             SIZE
ubuntu                               network                                                 0e9903d314af   7 seconds ago       164MB
nginx                                latest                                                  f652ca386ed1   4 days ago          141MB
ubuntu                               latest                                                  ba6acccedd29   7 weeks ago         72.8MB
```

1번 터미널
```bash
root@d86aac5008d6:/# exit
exit

$ docker run -it --rm --name net-tool --network bridge ubuntu:network bash
root@9ba6033c78b8:/# nslookup webserver1
;; connection timed out; no servers could be reached


root@9ba6033c78b8:/# nslookup 172.18.0.2
** server can't find 2.0.18.172.in-addr.arpa: NXDOMAIN

root@9ba6033c78b8:/# curl http://172.18.0.2
curl: (28) Failed to connect to 172.18.0.2 port 80: Connection timed out
```

## docker 포트 외부 노출

```bash
$ docker run -d --name webserver1 -p 8080:80 nginx:latest
9fe2c3187e932c1de08dfc2a31c553f846b52c110bfc1c1372a98d7b9e09ce09

$ curl http://localhost:8080
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

$ curl http://192.168.60.18:8080
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

| 커멘드 | 설명 |
|---|:---:|
| docker network ls | 컨테이너 네트워크를 리스트로 표시 |
| docker network inspect | 네트워크명을 지정해서 자세한 내용을 표시 |
| docker network create | 컨테이너 네트워크를 생성 |
| docker network rm | 컨테이너 네트워크를 삭제 |
| docker network connect | 컨테이너를 컨테이너 네트워크에 접속 |
| docker network disconnect | 컨테이너를 컨테이너 네트워크에서 분리 |


## 컨테이너 네트워크를 연결 및 외부 포트 노출

php/index.php

```php
<html>
<head><title>PHP CONNECTION TEST</title></head>
<body>

<?php
$servername = "mysql";
$database = "mysql";

$username = getenv('MYSQL_USER');
$password = getenv('MYSQL_PASSWORD');

try {
    $dsn = "mysql:host=$servername;dbname=$database";
    $conn = new PDO($dsn, $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    print("<p>접속에 성공했습니다.</p>");
} catch(PDOException $e) {
    print("<p>접속에 실패했습니다.</p>");
    echo $e->getMessage();
}

$conn = null; 
print('<p>종료합니다.</p>');
?>

</body>
</html>
```

Dockerfile

```dockerfile
FROM php:7.0-apache
RUN apt-get update && apt-get install -y \
    && apt-get install -y libmcrypt-dev mysql-client \
    && apt-get install -y zip unzip git vim
RUN docker-php-ext-install pdo_mysql session json mbstring
COPY php/ /var/www/html/
```

```bash
$ docker network create api-net
11a23b84869a971a4a18ff95aa2327e1de5e889270effafda5850f075a8191f1

$ docker run -d --name mysql --network api-net -e MYSQL_ROOT_PASSWORD=qwerty mysql:5.7
08f26dc3354a4e4df6d5eded9d29f184ae9a1fb1c1b75a47ab436084ea97db85

$ docker build -t php-api:0.1 .
// <생략>

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
php-api                              0.1                                                     9c06dcde6bcd   56 seconds ago   574MB
mysql                                5.7                                                     738e7101490b   4 days ago       448MB

$ docker run -d --name php --network api-net -p 8080:80 -e MYSQL_USER=root -e MYSQL_PASSWORD=qwerty php-api:0.1
ee64e71baf41a3681b484c23bb457742e15202489e9e50466d7122f38bf227c3

$ curl http://localhost:8080/
<html>
<head><title>PHP CONNECTION TEST</title></head>
<body>

<p>접속에 성공했습니다.</p><p>종료합니다.</p></body>
</html>

$ docker logs php
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.19.0.3. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.19.0.3. Set the 'ServerName' directive globally to suppress this message
[Tue Dec 07 08:02:46.091982 2021] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.25 (Debian) PHP/7.0.33 configured -- resuming normal operations
[Tue Dec 07 08:02:46.092035 2021] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
172.19.0.1 - - [07/Dec/2021:08:03:09 +0000] "GET / HTTP/1.1" 200 412 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36"
172.19.0.1 - - [07/Dec/2021:08:04:01 +0000] "-" 408 0 "-" "-"
172.19.0.1 - - [07/Dec/2021:08:04:26 +0000] "-" 408 0 "-" "-"
```

## 환경 변수 API #1

Dockerfile

```dockerfile
FROM alpine:latest
RUN apk update && apk add bash
ADD ./my_daemon /my_daemon
CMD ["/bin/bash", "/my_daemon"]
```

my_daemon

```bash
# 카운터 초기화
COUNT=0

# 환경변수가 없으면 설정 
if [ -z "$INTERVAL" ]; then
    INTERVAL=3
fi

# 메인 루프
while [ ture ];
do
    TM=`date|awk '{print $4}'`
    printf "%s : %s \n" $TM $COUNT
    let COUNT=COUNT+1
    sleep $INTERVAL
done
```

```bash
$ docker build --tag my_daemon:0.1 .
// <생략>

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
my_daemon                            0.1                                                     70291626f678   20 seconds ago   10MB

$ docker run --name myd my_daemon:0.1
08:46:42 : 0
08:46:45 : 1
08:46:48 : 2
08:46:51 : 3 # <- docker stop myd
08:46:54 : 4
08:46:57 : 5
08:47:00 : 6

$ docker start -i myd
08:49:42 : 0
08:49:45 : 1
08:49:48 : 2
08:49:51 : 3 # <- docker stop myd
08:49:54 : 4
08:49:57 : 5
08:50:00 : 6
```

## 환경 변수 API #2

my_daemon2 (개선 버전)

```bash
# 카운터 초기화
COUNT=0

# 환경변수가 없으면 설정 
if [ -z "$INTERVAL" ]; then
    INTERVAL=3
fi

# 기동시 상태 취득
if [ -f save.dat ]; then
   COUNT=`cat save.dat`
   rm -f save.dat
fi

# SIGTERM 시그널 처리 
save() {
  echo $COUNT > save.dat
  exit
}
trap save TERM


# 메인 루프
while [ ture ];
do
    TM=`date|awk '{print $4}'`
    printf "%s : %s \n" $TM $COUNT
    let COUNT=COUNT+1
    sleep $INTERVAL
done
```

Dockerfile2

```dockerfile
FROM alpine:latest
RUN apk update && apk add bash
ADD ./my_daemon2 /my_daemon
CMD ["/bin/bash", "/my_daemon"]
```

```bash
$ docker build --tag my_daemon:0.2 -f Dockerfile2 .
// <생략>

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
my_daemon                            0.1                                                     70291626f678   20 seconds ago   10MB
my_daemon                            0.2                                                     c8143e9795e1   11 seconds ago   10MB

$ docker run --name myd my_daemon:0.2
09:00:32 : 0
09:00:35 : 1
09:00:38 : 2 # <- docker stop myd

$ docker start -i myd
09:01:11 : 3
09:01:14 : 4
09:01:17 : 5 # <- docker stop myd
```

## 환경 변수 API #3

my_daemon3 (개선 버전)

```bash
# 카운터 초기화
COUNT=0

# 퍼시스턴트 볼륨
PV=/pv/save.dat

# 환경변수가 없으면 설정
if [ -z "$INTERVAL" ]; then
    INTERVAL=3
fi

# 기동 시 상태 취득 
if [ -f $PV ]; then
   COUNT=`cat $PV`
   rm -f $PV
fi

# SIGTERM 시그널 처리
save() {
  echo $COUNT > $PV
  exit
}
trap save TERM


# 메인 루프
while [ ture ];
do
    TM=`date|awk '{print $4}'`
    printf "%s : %s \n" $TM $COUNT
    let COUNT=COUNT+1
    sleep $INTERVAL
done
```

Dockerfile3

```dockerfile
FROM alpine:latest
RUN apk update && apk add bash
ADD ./my_daemon3 /my_daemon
CMD ["/bin/bash", "/my_daemon"]
```

```bash
$ docker build --tag my_daemon:0.3 -f Dockerfile3 .
// <생략>

$ docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
my_daemon                            0.1                                                     70291626f678   20 seconds ago   10MB
my_daemon                            0.2                                                     c8143e9795e1   11 seconds ago   10MB
my_daemon                            0.3                                                     03f7acaebf77   1 second ago     10MB

$ docker run --name myd -v `pwd`/data:/pv my_daemon:0.3
09:00:32 : 0
09:00:35 : 1
09:00:38 : 2 # <- docker stop myd

$ docker start -i myd
09:01:11 : 3
09:01:14 : 4
09:01:17 : 5 # <- docker stop myd

$ docker rm myd

$ docker run --name myd -v `pwd`/data:/pv my_daemon:0.3
09:01:53 : 6
09:01:56 : 7
09:01:59 : 8 # <- docker stop myd

$ docker run -d --name myd -v `pwd`/data:/pv my_daemon:0.3
9ab72b5dc5f1433f39d654c0cc5af736c7c87ee0b14dd9ac6b9d5c04530ed614

$ docker logs myd
09:31:51 : 14 
09:31:54 : 15 
09:31:57 : 16 
09:32:00 : 17 
09:32:03 : 18

$ docker attach --sig-proxy=false myd
09:32:33 : 28 
09:32:36 : 29 
09:32:39 : 30 
09:32:42 : 31 
```