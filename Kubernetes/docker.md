# docker

### docker run
| 값 | 설명 |
|---|:---:|
| `-i` | 키보드 입력을 컨테이너의 표준 입력에 연결하여 키보드 입력을 컨테이너의 셀 등에 보낸다. |
| `-t` | 터미널을 통해 대화형 조작이 가능하게 한다. |
| `-d` | 백그라운드로 컨테이너를 돌려 터미널과 연결하지 않는다. |
| `--name` | 컨테이너에 이름을 설정한다. 시스템에서 유일한 이름이어야 하며, 옵션을 생략하면 자동으로 만들어진 이름이 부여된다. |
| `--rm` | 컨테이너가 종료하면 종료 상태의 컨테이너를 자동으로 삭제한다. |

### docker stop
정상적인 종료를 실행한다. 

### docker kill
kill은 정상적인 종료가 원활하지 못한 경우 종료하기 위해 사용된다. (stop를 권장)

1번 터미널
```bash
// stop에서 사용 (6d4fd22c3865)
$ docker run  -it ubuntu bash

// kill에서 사용 (3e98a3014884)
$ docker run  -it ubuntu bash
```
1번 터미널에서 stop용 docker를 실행시키고 2번 터미널에서 stop후에 kill용 docker를 실행시킨다.


2번 터미널
```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS              PORTS     NAMES
6d4fd22c3865   ubuntu    "bash"    About a minute ago   Up About a minute             focused_bardeen

$ docker stop 6d4fd22c3865
6d4fd22c3865

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS                      PORTS     NAMES
6d4fd22c3865   ubuntu    "bash"    About a minute ago   Exited (0) 14 seconds ago             focused_bardeen

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS                      PORTS     NAMES
3e98a3014884   ubuntu    "bash"    9 seconds ago   Up 11 seconds                         priceless_dijkstra
6d4fd22c3865   ubuntu    "bash"    2 minutes ago   Exited (0) 33 seconds ago             focused_bardeen

$ docker kill 3e98a3014884
3e98a3014884

$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                       PORTS     NAMES
3e98a3014884   ubuntu    "bash"    24 seconds ago   Exited (137) 3 seconds ago             priceless_dijkstra
6d4fd22c3865   ubuntu    "bash"    2 minutes ago    Exited (0) 49 seconds ago              focused_bardeen
```
STATUS 부분을 보면 Exited (137), Exited (0)로 정상 종료와 비정상 종료 상태가 다른 것을 확인할 수 있다.

### docker start
정지 상태인 컨테이너를 재기동 한다.

```bash
$ docker start -i 3e98a3014884
root@3e98a3014884:/#

// 다음을 위한 준비
root@3e98a3014884:/# apt update
root@3e98a3014884:/# apt upgrade
root@3e98a3014884:/# apt install git
```
-i로 표준출력, 표준에러를 터메널에 표시하도록 한다.

### docker commit
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

### docker push
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

### docker rm
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

### docker rmi
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