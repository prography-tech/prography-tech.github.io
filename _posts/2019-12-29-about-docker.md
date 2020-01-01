---
layout: post
title: "docker에 대해 알아보자!"
author: soom
date: 2019-11-11 00:00
tags: [docker]
image: 'https://live.staticflickr.com/7336/14098888813_1047e39f08.jpg'
---

## Docker

###### 세줄요약

```
- 컨테이너 기반 오픈소스 가상화 플랫폼
- 컴퓨터 성능을 효율적으로 사용할 수 있다
- 배포 관리에 편하다
```

![docker logo](https://live.staticflickr.com/7336/14098888813_1047e39f08.jpg)



기본적으로, 프로젝트를 진행하다보면 모든 사람들이 같은 환경을 가지고 있지 않다. 누구는 윈도우 / 누구는 맥 등등 ! 각 컴퓨터에 동일한 버전으로 설치를 했다고 할지라도, 뭔가 요상하게 깨지는 일들도 경험해 본적이 있을 것이다. 이런 고역을 피하기 위해 __Docker__ 를 활용해서 개발 환경을 맞추고 배포 관리도 편하게 할 수 있다! 



#### 기존 서버 구축

기존에 서버를 구축하려면 ubuntu 인스턴스 생성 / 웹 서버 설정 / db 설정 / 소스 복사 등 일일이 손으로 하나하나 설정해야한다. 도커를 사용하면 호스트 OS 와 서비스 운영 환경을 분리시켜 이런 번거로움을 해결해준다.



#### 가상화

도커는 가상화를 통해 하드웨어의 물리적 리소스를 최대한 활용한다.



![visualization](https://www.redhat.com/cms/managed-files/server-usage-500x131.png)

기존에는 1개의 서버에 30%의 서버를 사용하더라도 여러개의 서버를 하나의 서버에 처리하기 어려워, 비효율적이라는 단점이 있었다. 하지만 도커는 가상화를 통해 1개의 서버를 2개의 고유한 서버로 분할 해 각각의 독립적인 태스크를 처리하여, 효율적으로 서버를 사용할 수 있다.

![virtualization](https://www.redhat.com/cms/managed-files/server-usage-for-virtualization-500x131.png)

이렇게 하면 빈 서버를 사용해 다른 태스크를 처리하거나 / 비용을 절감할 수 있다. 가상화에는 호스트형 가상화 / 하이퍼바이저 형 가상화 / 컨테이너 가상화가 있다. 

__호스트형 가상화__

호스트 형 가상화는 하드웨어 위에 `HOST OS` 를 설치하고 가상화 소프트웨어를 이용해 OS 를 작동하는 방식. 가상 환경을 쉽게 구축할 수 있지만,OS 상에서 또다른 OS 가 돌아가므로 자원이 많이 소비되고 느리다. 

##### __하이퍼바이저형 가상화__

__VM__ 의 경우 `hypervisor` 라는 소프트웨어를 통해 물리 리소스를 분리하는 것. 해당 소프트웨어가 하드웨어와 가상 환경을 제어하는데, `host os` 를 사용하지 않지만, 각각의  VM 에서 guest os 가 돌아가기 때문에 가상환경이 시작할 때 걸리는 오버 헤드가 커진다. 

__container 가상화__ 

__VM__ 의 단점인 오버헤드를 최소화 하기 위한 방법으로, HOST OS 상에 독립적인 공간을 만들어, 별도의 서버인 것 처럼 사용한다. 각 어플리케이션이 HOST OS 를 공유하기 때문에 오버헤드가 작아진다. 



![vm vs docker](https://miro.medium.com/max/1400/1*wOBkzBpi1Hl9Nr__Jszplg.png)

VM 의 경우 hypervisor 위에 여러 guest os 가 올라가기 때문에 볼륨이 도커보다 크다. 하지만 도커는 host os 를 공유하기 때문에 해당 볼륨이 작을 뿐더러, docker 의 라이프 사이클은 docker hub 에서 image 를 pull 받으면서 시작하기 때문에, 용량이 훨신 작아 네트워크를 덜 쓰기 때문에 비용도 줄어든다. 



##### 컨테이너

다양한 프로그램 / 실행환경을 컨테이너에 묶어 동일한 인터페이스 제공하여 
프로그램의 배포 / 관리를 쉽게 해줄 수 있는 도구



##### 도커 이미지

도커 이미지란 컨테이너를 생성하기 위해 필요한 설계도. 컨테이너를 생성하기 위한 파일. 
해당 파일은 docker hub 에 저장됨. 이미지는 클래스, 컨테이너는 객체 개념인 것 같다. 



## Getting started

도커 설치 (mac osx)

<https://docs.docker.com/docker-for-mac/install/>

##### terminal 

도커가 잘 설치되어있는지 테스트 해보기 

```bash
$ docker --version
# Docker version 18.09.2, build 6247962
```

##### 도커 이미지 만들기 

```bash
$ docker run hello-world
#Unable to find image 'hello-world:latest' locally
#latest: Pulling from library/hello-world
#1b930d010525: Pull complete 
#Digest: sha256:6540fc08ee6e6b7b63468dc3317e3303aae178cb8a45ed3123180328bcc1d20f
#Status: Downloaded newer image for hello-world:latest

#Hello from Docker!
#This message shows that your installation appears to be working correctly.

```

##### 이미지 다운로드 

```bash
$ docker pull ubuntu
$ cat /etc/issue # 우분투 버전 확인
```

내 레포에 없으면 공식적인 문서를 가져온다. run 을 했을 경우 없으면 pull 후 실행.

##### 도커 실행 컨테이너 확인

```bash
$ docker ps
```

##### 도커 이미지 리스트

```bash
$ docker image ls
```

##### 도커 컨테이너 생성

```bash
$ docker run -i -t --name ubuntu-ruby ubuntu:16.04 /bin/bash
```

##### 도커 컨테이너 종료

```bash
$ docker kill [이름] /bin/bash 
```

##### 도커 이미지 / 컨테이너 강제 삭제

```bash
$ docker rmi -f
```

##### 컨테이너 재부팅 

```bash
$ docker restart [container_id]
```

##### 컨테이너 접속하기

```bash
$ docker start [name]
$ docker attach [container_id]
```

##### 컨테이너 삭제

```bash
# 현재 실행중인 모든 컨테이너 강제 종료
$ docker kill $(docker ps -q -f status=running)   
# 현재 실행중인 모든 컨테이너 종료
$ docker stop $(docker ps -q -f status=running)   
# 종료된 모든 컨테이너 삭제
$ docker rm $(docker ps -q -f status=exited)      
```



## Dockerfile

도커에 여러 설치할 내용들을 넣고 한번에 컨테이너를 만들 수 있다. 여러 설정 컨피그를 더하기 위해 이미지 빌드용 DSL(Domain Specific Language) 파일을 사용한다



#### 도커파일 만들기

- **Dockerfile** 이름으로 만들면 된다.

##### sample

아래 샘플은 django 환경 관련 구성 설정이 들어있는 도커 파일이다 

```dockerfile
FROM python:3.7

##########################################################
# set timezone
##########################################################
ENV TZ="/usr/share/zoneinfo/Asia/Seoul"
ENV PYTHONUNBUFFERED 0

RUN apt-get update && apt-get -y install libpq-dev
RUN apt-get -y install postgresql-client

ARG PROJECT_DIR="/app"
##########################################################
# install dependencies via pip
##########################################################
COPY requirements.txt requirements.txt
RUN pip install --upgrade pip && pip install -r requirements.txt

##########################################################
# set working directory
##########################################################
WORKDIR ${PROJECT_DIR}

##########################################################
# add proejct files
##########################################################
COPY . $PROJECT_DIR

##########################################################
# expose port for single container(Elastic Beanstalk)
##########################################################
EXPOSE 8000

##########################################################
# command
##########################################################
CMD ["sh", "./entry_point.sh"]

```

django 환경 내에 dockerfile 을 만들면 `COPY . $PROJECT_DIR` 를 통해 도커 컨테이너에 장고 프로젝트를 옮겨준다. `CMD` 는 도커 내에서 실행하는 명령어인데, 할 명령이 많아 쉘파일을 만들어서 작성해주었다. 

#### 도커파일 실행

```
$ docker build -t [name] .
```

도커파일 이름을 찾아 해당 파일 구성으로 이루어진 도커 이미지를 생성한다. 

```
$ docker images
```

##### sample

django 에서 제공하는 샘플 파일

```dockerfile
# version: docker-compose version 
version: "3.7"
# service: 실행할 여러 컨테이너들을 넣는다.
services:
  # redis container
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  # database container
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

도커 컴포즈 시작

```
$ docker-compose up
```

도커 컴포즈 종료

```
$ docker-compose down 
```



##### 참고자료

- <https://www.redhat.com/ko/topics/virtualization/what-is-virtualization>
