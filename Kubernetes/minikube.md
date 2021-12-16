# minikube

minikube는 로컬에서 kubernetes환경을 간단하게 구성할수 있기 때문에 많이 사용이 된다.

사용법도 간단해서 kubenetes를 학습하기 위한 최적의 방법을 제공해 준다.

[Minikube Download](https://minikube.sigs.k8s.io/docs/start/ "minikube installation")

## 버전확인
```bash
$ minikube version
```

## 가상머신 시작 
```bash
$ minikube start                                  // 기본
$ minikube start --driver=docker                  // docker desktop 이용
$ minikube start --driver=hyperv                  // hyperv 이용
$ minikube start --driver=virtualbox              // virtual box 이용
$ minikube start --kubernetes-version=v1.20.0     // kubenetes 버전 지정
```

## 상태확인
```bash
$ minikube status
```

## 정지
```bash
$ minikube stop
```

## 삭제
```bash
$ minikube delete
```

## ssh 접속
```bash
$ minikube ssh
```

## ip 확인
```bash
$ minikube ip
```

## 다중 노드
```bash
$ minikube start
$ minikube start -n 3     // 다중 노드
```

## 프로필
```bash
$ minikube start                  // minikube profile로 생성
$ minikube start -p helloworld    // helloworld profile로 생성
```

## profile 목록
```bash
$ minikube profile list
```

## 현재 profile 확인
```bash
$ minikube profile
```

## profile로 변경
```bash
$ minikube profile helloworld     // helloworld profile로 변경
$ minikube profile minikube       // minikube profile로 변경
```

## 가상머신 제거
```bash
$ minikube delete                 // 현재 profile 가상머신 제거
$ minikube delete --all           // 전체 제거
```

## 대쉬보드
```bash
$ minikube dashboard
```

## 일시정지
```bash
$ minikube pause                  // 일시정지
$ minikube unpause                // 일시정지 해제
```

## 설정
```bash
$ minikube config set memory 16384        // 메모리 설정
$ minikube config unset memory            // 메모리 설정 초기화
$ minikube config view                    // 설정 보기
```

설정 옵션
| 값 | 설명 |
|---|---|
| `defaults` | Lists all valid default values for PROPERTY_NAME |
| `get` | Gets the value of PROPERTY_NAME from the minikube config file |
| `set` | Sets an individual value in a minikube config file |
| `unset` | unsets an individual value in a minikube config file |
| `view` | Display values currently set in the minikube config file |

설정 가능한 항목
 * driver
 * vm-driver
 * container-runtime
 * feature-gates
 * v
 * cpus
 * disk-size
 * host-only-cidr
 * memory
 * log_dir
 * kubernetes-version
 * iso-url
 * WantUpdateNotification
 * WantBetaUpdateNotification
 * ReminderWaitPeriodInHours
 * WantNoneDriverWarning
 * profile
 * bootstrapper
 * insecure-registry
 * hyperv-virtual-switch
 * disable-driver-mounts
 * cache
 * EmbedCerts
 * native-ssh

[참고자료] https://minikube.sigs.k8s.io/docs/start/

## node

```bash
$ minikube start

$ minikube node add node1

$ minikube node add node1

$ minikube node add node3

$ kubectl get node
NAME           STATUS   ROLES                  AGE    VERSION
minikube       Ready    control-plane,master   23h    v1.22.3
minikube-m02   Ready    <none>                 104s   v1.22.3
minikube-m03   Ready    <none>                 58s    v1.22.3
minikube-m04   Ready    <none>                 14s    v1.22.3

$ minikube delete --all

$ minikube start -n 4

$ kubectl get node
NAME           STATUS   ROLES                  AGE    VERSION
minikube       Ready    control-plane,master   2m7s   v1.22.3
minikube-m02   Ready    <none>                 92s    v1.22.3
minikube-m03   Ready    <none>                 56s    v1.22.3
minikube-m04   Ready    <none>                 22s    v1.22.3

$ minikube node list
minikube        192.168.49.2
minikube-m02    192.168.49.3
minikube-m03    192.168.49.4
minikube-m04    192.168.49.5
```

node command

| 값 | 설명 |
|---|---|
| `add` | Adds a node to the given cluster config, and starts it |
| `start` | Starts an existing stopped node in a cluster |
| `stop` | Stops a node in a cluster |
| `delete` | Deletes a node from a cluster |
| `list` | List existing minikube nodes |
| `help` | Help about any command |