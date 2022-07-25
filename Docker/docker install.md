# Docker & Docker-compose 설치 (Ubuntu 20.04.3)

## Docker
설치 참조 https://docs.docker.com/engine/install/ubuntu/

```bash
// 패키지 업데이트
# sudo apt-get update

// 필수 패키지 설치
# sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

// GPG Key 인증
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

// 아키텍쳐 검사
# arch
x86_64

// docker repository 등록
# sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

// docker 설치
# sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io

// 유저 권한 주기
# sudo usermod -a -G docker $USER

// 버전 체크
# docker -v
Docker version 20.10.12, build e91ed57
```
서버의 아키텍쳐별로 설치해야 할 설치파일이 다르기 때문에 아키텍쳐 검사를 한후 `[arch=amd64]` 부분을 지정해야 한다.

## Docker 서비스 등록 (옵션)
```bash
// 서비스 등록
# sudo systemctl enable docker && service docker start

// 서비스 등록 취소
# sudo systemctl disable docker.service
# sudo systemctl disable docker.socket

// 서비스 등록 취소 확인
# systemctl list-unit-files | grep -i docker
docker.service                         disabled        enabled      
docker.socket                          disabled        enabled  
```
Portainer를 구축하려면 Docker의 서비스 등록이 필요하다.

## Docker-compose
설치할 버전체크 https://github.com/docker/compose/releases/

```bash
// 다운로드
# sudo curl -L https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# sudo curl -L https://github.com/docker/compose/releases/download/1.26.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

// 권한 처리
# sudo chmod +x /usr/local/bin/docker-compose

// 버전 체크
# docker-compose -v
Docker Compose version v2.2.2
```
설치할 버전을 미리 체크해서 최신 버전을 설치하는 것이 좋다.