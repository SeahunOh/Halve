---
title: Docker For OSX
description: how to setting docker in osx
categories:
 - docker
tags:
 - docker
 - usage
 - osx
---

> OSX 사용자를 위한 Docker 설치부터 접근 방법
>
> 이 게시글은 지속적으로 업데이트 예정

<!-- more --> 

## Install

> 명령어 작동 안할 시 sudo(root 권한)으로 진행

##### 1. [Docker 홈페이지](https://store.docker.com/editions/community/docker-ce-desktop-mac)에 접속하여 dmg 다운로드 및 설치

##### 2. `docker version` 을 입력하여 설치완료 확인

```
$ docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:13:02 2018
 OS/Arch:      darwin/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:22:38 2018
  OS/Arch:      linux/amd64
  Experimental: true
```

##### 3. git이나 타 사이트에서 받은 이미를 이용하여 docker를 통해 가상화 (e.g. [CVE-2018-2628](https://nvd.nist.gov/vuln/detail/CVE-2018-2628) Dockerfile)

```
$ docker pull zhiqzhao/ubuntu_weblogic1036_domain
$ docker run -d -p 7001:7001 -i -t --name wl zhiqzhao/ubuntu_weblogic1036_domain /bin/bash
$ docker attach wl

root@7f81d516676b:~/Oracle/Middleware#
```

##### 4. 가상화 종료 및 제거

```
$ docker stop wl # service stop
$ docker rm wl # remove docker
```

## Interact

- **Docker -> Host(OSX)**
  - docker.for.mac.localhost
  - e.g. 
    - nc docker.for.mac.localhost 8080
- **Host(OSX) -> Docker**
  - 127.0.0.1
  - e.g. 
    - nc 127.0.0.1 7001 (본인이 설정한 포트)

## Command

- docker run \[options\] \[image name\] \[process\]
  - 도커 서비스 **등록** 및 **실행**
    - -p : port forwarding,
    - --name : docker의 이름 설정 
    - -i : foreground mode (attch시 화면 출력)
    - -d : background mode (shell exit해도 계속 실행) 
    - -t : detached mode
  - e.g.
    - docker run -t -d -p 80:10101  --name ctf-main ctf-flatform:0.9 "/bin/bash"
- docker ps
  - 등록된 도커 서비스 **목록**
    - -a : all list
- docker stop *[container name]*
  - 서비스 중인 도커 **중지**
- docker start *[container name]*
  -  서비스가 중지된 도커 **실행**
- docker rm *[container name]*
  - 도커 서비스 목록에서 **제거**
- docker commit *\[option\] \[container name\] \[image name\]:\[tag\]* 
  - 도커 **커밋**, image로 **저장**
    - -m : 로그 저장
  - e.g.
    -  docker commit -m "apache2+php5.6+mysql" WebSetting webbase_php5:0.1
- 