# Docker & Docker-compose 설치 (Ubuntu 20.04.3)

## Docker
설치 참조 https://docs.docker.com/engine/install/ubuntu/

```bash
// 피키지 업데이트
$ sudo apt-get update

// 필수 패키지 설치
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

// GPG Key 인증
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

// docker repository 등록
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

// docker 설치
$ sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io

// 유저 권한 주기
$ sudo usermod -a -G docker $USER

// 버전 체크
$ docker -v
Docker version 20.10.12, build e91ed57
```

## Docker 서비스 등록 (옵션)
```bash
// 서비스 등록
$ sudo systemctl enable docker && service docker start

// 서비스 등록 취소
$ sudo systemctl disable docker.service
$ sudo systemctl disable docker.socket

// 서비스 등록 취소 확인
$ systemctl list-unit-files | grep -i docker
docker.service                         disabled        enabled      
docker.socket                          disabled        enabled  
```

## Docker-compose
설치할 버전체크 https://github.com/docker/compose/releases/

```bash
// 다운로드
$ sudo curl -L https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo curl -L https://github.com/docker/compose/releases/download/1.26.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

// 권한 처리
$ sudo chmod +x /usr/local/bin/docker-compose

// 버전 체크
$ docker-compose -v
Docker Compose version v2.2.2
```