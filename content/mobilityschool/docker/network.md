---
title: Network &  Dockerfile
weight: 4
---
0. 컨테이너 와 이미지 모두 삭제
- 모든 컨테이너 중지: `docker stop $(docker ps -a -q)`
- 모든 컨테이너 삭제: `docker rm $(docker ps -a -q)`

- 모든 이미지 삭제: `docker rmi $(docker images -q)`

## Docker Network
- 도커 컨테이너 및 서비스는 도커 네트워크를 통해 격리된 컨테이너 간의 네트워크 연결 뿐 아니라 도커 외의 다른 애플리케이션 워크로드 와도 연결이 가능
- 도커 네트워크의 하위 시스템 연결을 위해 도커 네트워크 드라이버를 이용해서 상호 간 통신이 가능
- 도커 컨테이너는 별도의 설정을 하지 않는 한 docker0 이라는 브릿지에 연결되어 172.17.0.0/16의 CIDR 범위로 IP 주소가 할당됨
- 컨테이너를 생성하면 eth0 인터페이스 카드에 172.17.0.2 형태로 IP주소를 할당해서 생성됨
- 확인: `ifconfig docker0(ifconfig 명령이 없으면 apt install net-tools)`

### 컨테이너를 생성하고 네트워크 설정을 확인
- ubuntu:14:04 이미지를 가지고 컨테이너 2개 생성
```
docker run --name container1 -dit ubuntu:14.04
docker run --name container2 -dit ubuntu:14.04
```
- 컨테이너 상세보기에서 IPAddress(MAC, route 정보)라는 항목을 확인하면 IP할당을 확인
- 컨테이너 상세보기에서 .NetworkSettings.IPAddress 그룹을 확인하면 네트워크 설정을 확인할 수 있음
- 리눅스 명령어에서 ifconfig(route, ip addr 등)를 확인하면 네트워크 정보를 확인할 수 있음 
  - `docker exec container1 ifconfig`
- 호스트 컴퓨터에서 ifconfig를 확인해보면 컨테이너 개수 만큼 인터페이스가 추가된 것을 확인할 수 있음

### 도커 네트워크 드라이버
- 도커 네트워크 드라이버를 사용할 때는 `--net`이나 `--network` 명령을 이용해서 선택할 수 있고 `docker network` 명령을 통해서 호출해서 사용
  - bridge: 기본 네트워크 드라이버로 컨테이너로 실행 시 별도의 네트워크 지정없이 독립적으로 실행되는 애플리케이션 컨테이너를 실행하는 경우 사용하는데 브릿지 모드는 동일 호스트 상의 도커 컨테이너에만 적용
  - host: 컨테이너 와 호스트 간의 네트워크 격리를 제거하고 호스트의 네트워킹을 직접 사용할 수 있는데 이렇게 하면 컨테이너 애플리케이션에 별도의 포트 연결 없이 호스트의 포트를 바로 연결할 수 있음
  - overlay: 도커 클러스터인 도커 스웜 구축시 호스트 와 호스트 간의 컨테이너 연결에 사용
  - none: 네트워크 사용 안 함
  - container:공유받을컨테이너이름: 다른 컨테이너의 정보를 공유
  - 사용자정의네트워크
- host 모드 와 bridge 모드의 차이
  - nginx를 포트 설정없이 컨테이너로 실행
```
Host Computer
    NIC - ifconfig
        IPTable - iptables
            Docker Bridge Network - bctl show
                Container - ifconfig / route / ip addr
```
```
PAT(NAT Overload) - 포트1개를 IP1개처럼 사용
NAT - 보안을 위해서 사용
```
- host 모드 와 bridge 모드의 차이
  - nginx를 포트 설정없이 컨테이너로 실행
    - `docker run -d --name=noport nginx`
    - 이 경우 컨테이너에서 서비스 하는 내용을 외부에서 사용할 수 없음
  - nginx를 브릿지 모드로 외부에서 접근할 수 있도록 컨테이너로 실행
    - `docker run -d --name=bridgeport -p 8001:80 nginx`
    - http://localhost:8001 로 서비스 받을 수 있습니다.
  - nginx를 호스트 모드로 외부에서 접근할 수 있도록 컨테이너로 실행
    - `docker run -d --name=host --net=host nginx`
    - http://localhost:80 로 서비스 받을 수 있습니다.

### 네트워크 생성 및 할당
- 네트워크 생성
  - docker network create [--driver=네트워크드라이버종류] 네트워크이름
- 컨테이너에 네트워크 설정을 할 때는 컨테이너를 만들 때 --net 옵션에 네트워크 이름을 설정해주면 됨
- 실습
  - 네트워크 생성(webapp-vnet, mobileapp-vnet)
  ```bash
  docker network create webapp-vnet
  docker network create --driver=bridge mobileapp-vnet
  ```
  - 네트워크 확인 `docker network ls`
  - 네트워크를 설정해서 컨테이너 생성
  ```bash
  docker run -dit --name=webapp --net=webapp-vnet ubuntu:14.04
  docker run -dit --name=mobileapp --net=mobileapp-vnet ubuntu:14.04
  ```
  - IP 확인 - 서브넷 마스크가 기본적으로 /16 이므로 위 2개의 컨테이너는 서로 다른 네트워크가 됨
  ```bash
    dh@dh:~$ docker inspect webapp | grep -i ipaddress
                "SecondaryIPAddresses": null,
                "IPAddress": "",
                        "IPAddress": "172.18.0.2",
    dh@dh:~$ docker inspect mobileapp | grep -i ipaddress
                "SecondaryIPAddresses": null,
                "IPAddress": "",
                        "IPAddress": "172.19.0.2",
  ```
  - 네트워크 정보를 상세히 출력(할당된 컨테이너 확인 가능): `docker network inspect webapp-vnet`
- 브릿지네트워크의 IP 대역은 순차적으로 할당되는데 이를 원하는 대역으로 설정하는 것이 가능한데 이 경우는 네트워크를 생성할 때 `--subnet` 그리고 `--ip-range`  와 `--gateway`를 설정해주면 됨
  - 네트워크 생성
    ```bash
    docker network create --ip-range 172.100.1.0/24 --subnet 172.100.1.0/24 --gateway 172.100.1.1 custom-net
    ```
  - 네트워크 확인 `docker network inspect custom-net`
  - 컨테이너 생성
    ```bash
    docker run -dit --net=custom-net --name=cust-net1 ubuntu:14.04
    docker run -dit --net=custom-net --name=cust-net2 --ip 172.100.1.100 ubuntu:14.04
    ```
  - 하나의 컨테이너에 접속해서 ping 명령을 수행
    ```bash
    dh@dh:~$ docker exec -it cust-net1 /bin/bash
    root@b35f06aaf319:/# ping -c 3 cust-net2
    PING cust-net2 (172.100.1.100) 56(84) bytes of data.
    64 bytes from cust-net2.custom-net (172.100.1.100): icmp_seq=1 ttl=64 time=12.2 ms
    64 bytes from cust-net2.custom-net (172.100.1.100): icmp_seq=2 ttl=64 time=0.179 ms
    64 bytes from cust-net2.custom-net (172.100.1.100): icmp_seq=3 ttl=64 time=0.080 ms
    ```
### 사용자 정의 네트워크를 이용해서 Load Balancing
- 네트워크 생성(172.200.1.0/24)
```bash
docker network create --subnet 172.200.1.0/24 --ip-range 172.200.1.0/24 --gateway 172.200.1.1 netlb
```
- 네트워크 확인 `docker network ls`
- docker run 수행 시 `--net-alias` 또는 `--link` 옵션으로 묶인 컨테이너에는 기본적으로 서비스를 검색할 수 있는 내장 DNS 서버가 제공되는데 이를 자동화 DNS라고 한는데 이 DNS 때문에 사용자 정의 네트워크 안에서 컨테이너 이름과 IP주소가 매칭이 되는 것
- ubuntu:14.04 이미지를 이용해서 netlb 네트워크에 속한 3개의 컨테이너를 생성하는데 net-alias를 inner-dns-net으로 설정
```bash
docker run -dit --name=nettest1 --net=netlb --net-alias inner-dns-net ubuntu:14.04

docker run -dit --name=nettest2 --net=netlb --net-alias inner-dns-net ubuntu:14.04

docker run -dit --name=nettest3 --net=netlb --net-alias inner-dns-net ubuntu:14.04
```
- ping을 전송할 새로운 컨테이너를 생성하는데 네트워크는 netlb
```bash
docker run -it --name=frontend --net=netlb ubuntu:14.04 /bin/bash
```
- 컨테이너가 생성되면 안에서 아래 명령을 여러 번 수행하면서 IP를 확인
  - `ping -c 2 inner-dns-net`
  - ip가 변하는 것을 확인
- frontend에서 DNS 정보를 확인
  - dnsutils 패키지를 설치
    ```bash
    sudo apt-get update
    sudo apt-get -y install dnsutils
    ```
  - 확인 `dig inner-dns-net`
- 다른 터미널을 실행시켜서 컨테이너를 하나 추가
```
docker run -dit --name=nettest4 --net=netlb --net-alias inner-dns-net ubuntu:14.04
```
- 이전 터미널에서 DNS 정보를 확인 `dig inner-dns-net`

## Docker kill 명령과 초기화
### kill 명령
- 강제 종료를 위한 명령
- docker stop은 컨테이너 내에 메인 프로세스에 SIGTERM으로 종료를 전달하고 10초 후까지 종료되지 않으면 SIGKILL을 전송
- docker kill은 바로 SIGKILL을 전송해서 강제로 종료

### 초기화
- 모든 컨테이너 중지: `docker stop $(docker ps -a -q)`, `docker kill $(docker ps -a -q)`
- 모든 컨테이너 삭제: `docker rm $(docker ps -a -q)`
- 모든 이미지 삭제: `docker rmi $(docker images -q)`

## Dockerfile
### IaC & Dockerfile
- IaC가 필요한 이유
  - 명령어 기반의 인프라 구성 시 사용자 실수와 같은 인적 오류 가능성이 높음
  - 이런 인적 오류 가능성을 낮추기 위해서 인프라 구성을 코드로 수행

### 최적의 Dockerfile
- 경량의 컨테이너 서비스를 제공
- 레이어를 최소화
- 하나의 애플리케이션은 하나의 컨테이너로 실행
- 캐시 기능을 활용
- 디렉터리 단위로 작업
- 서버리스 환경으로 개발: 별도의 서버를 이용하지 않도록 작성

### Dockerfile 명령어
- FROM
  - 필수 항목
  - 이미지의 베이스 이미지를 지정하는데 hub.docker.com에서 제공하는 이미지를 권장
  - 이미지를 선택할 때 slim(작은 크기의 이미지) 이나 Alpine(리눅스 배포판) 이미지를 권장
- MAINTAINER
  - 이미지를 빌드한 작성자 이름 과 이메일을 작성
  - MAINTAINER adam <itstudy@kakao.com>
- LABEL
  - 이미지를 작성하는 목적으로 버전이나 타이틀 등을 작성
- RUN
  - 설정된 기본 이미지에 패키지 업데이트, 각종 패키지 설치, 명령 실행 등을 작성
  - 여러 개 작성 가능
  - RUN 명령어의 개수를 최소화하는 것을 권장
  - RUN 명령 하나의 하나의 레이어가 됨
  - 2가지 방법으로 작성하는데 하나는 shell 방식이고 다른 하나는 exec 방식
  - Shell 방식
    - `RUN apt update && apt install -y nginx`
  - exec 방식
  ```dockerfile
  RUN ["/bin/bash", "-c", "apt update"]
  RUN ["/bin/bash", "-c", "apt install -y nginx"]
  ```
- CMD
  - 생성한 이미지를 컨테이너로 실행할 때 실행되는 명령이고 ENTRYPOINT 명령으로 지정된 커맨드에 디폴트로 넘길 파라미터를 지정할 때 사용
  - 여러 개의 CMD를 작성해도 마지막 하나만 처리
  - 일반적으로 이미지의 컨테이너 실행 시 애플리케이션 데몬이 실행되도록 하는 경우 유용
    ```dockerfile
    CMD ["python", "app.py"]
    ```
- ENTRYPOINT
  - CMD와 마찬가지로 생성된 이미지가 컨테이너로 실행될 때 사용되지만 컨테이너가 실행될 때 명령어 및 인자 값을 전달받아 실행된다는 것이 다름
  - CMD 컨테이너 실행 시 다양한 명령어를 지정하는 경우에 유용하고 ENTRYPOINT는 컨테이너를 실행할 때 반드시 수행해야 하는 명령어를 지정
- COPY
  - 호스트 환경의 파일이나 디렉터리를 복사할 때 사용
- ADD
  - 파일 과 디렉토리를 이미지 안에 복사할 뿐 아니라 URL 주소에서 직접 다운로드 받아서 이미지 안에 넣을 수 있고 tar 나 tar.gz의 경우는 지정한 경로에 압축을 풀어서 추가
  - 빌드 작업 디렉토리 외부의 파일을 ADD 할 수 없고 디렉토리는 /로 끝나야 함
- ENV
  - 환경변수 설정에 이용
- EXPOSE
  - 포트나 프로토콜을 외부로 개방하기 위해서 사용
- VOLUME
  - 볼륨 지정을 위해서 사용
- USER
  - 기본 사용자인 root 이외의 사용자를 지정할 때 사용
- WORKDIR
  - 컨테이너 상에서 작업할 경로를 전환하기 위해서 사용하는데 이 디렉토리를 설정하면 RUN, CMD, ENTRYPOINT, COPY, ADD 명령문이 전부 이 디렉토리를 기준으로 수행됨
- ARG
  - build 시점에서 변수값을 전달하기 위해서 사용
  - build 할 때 `--build-arg 변수명=값`의 형태로 전달
- HEALTHCHECK
  - 프로세스의 상태를 체크하고자 하는 경우 사용
  - 하나의 명령만이 유효하고 여러 개가 지정된 경우 마지막에 선언된 HEALTHCHEK 만 적용
  - 옵션으로 interval 이 있는데 헬스 체크 간격으로 기본값은 30초이며 timeout은 기다리는 시간으로 기본값은 30초이며 retries는 타임 아웃 횟수로 기본값은 3
  - 상태코드는 0, 1, 2 세가지가 있는데 0은 성공이고 1은 올바르게 작동하지 않는 상태이며 2는 예약된 코드 `HEALTHCHECK --interval=1m --timeout=3s --retries=5 CMD curl -f http://localhost || exit 1`
- 시작은 FROM 부터 이지만 나머지는 순서가 없는데 명령 순서는 빌드 캐시의 무효화 와 연관되므로 변경 빈도 수가 적은 명령을 먼저 배치하는 것을 권장

### 이미지 빌드
- docker build [옵션] 이미지이름[:태그] 경로 | URL | 압축파일
- 옵션
  - -t: 태그를 지정하는 경우
  - -f: Dockerfile이 아닌 다른 파일명을 사용하는 경우
- 경로: 디렉토리 단위 개발을 권장하고 현재 경로에 Dockerfile 이 있으면 .을 사용하고 Dockerfile이 있는 경로를 설정해도 됨
- URL: 경로 대신에 Dockerfile이 포함된 URL을 제공하고자 하는 경우 사용
- 압축 파일을 사용하는 경우는 압축된 파일 안에 Dockerfile이 있는 경우 사용

### Dockerfile이 필요한 이유
- 도커 파일이 없는 경우
  - 우분투 위에 아파치 웹 서버 위에 PHP를 이용해서 웹 애플리케이션을 구현하는 경우에 우분투 이미지를 이용해서 컨테이너를 생성하고 컨테이너 안에 접속을 해서 아파치 웹 서버를 설치하고 아파치 웹 서버를 구동한 후 php를 설치하고 그 위에 코드를 작성해야 함
- 동일한 작업을 Dockerfile을 이용
  - Dockerfile에 모든 명령어를 작성해두고 한 번의 명령어로 수행

### Dockerfile 작성 및 실행
- 디렉터리를 생성해서 프롬프트를 이동
```
mkdir phpapp
cd phpapp
```
- 도커 파일을 생성하고 작성(vi Dockerfile)
```dockerfile
FROM ubuntu:14.04

MAINTAINER "dh"

LABEL title "IaC PHP application"

RUN apt-get update && apt-get -y install apache2 php5 git curl ssh wget

#Environment Variable
ENV APACHE2_RUN_USER www-data APACHE2_RUN_USER www-data APACHE2_LOG_DIR /var/log/apache2 APACHE2_WEB_DIR /var/www/html APACHE2_PID_FILE /var/run/apache2/apache2.pid

#basic web page
RUN echo 'Hello Docker Application' ? /var/www/html/index.html

#php 파일
RUN echo '<?php phpinfo(); ?>' > /var/www/html/index.php

EXPOSE 80

WORKDIR /var/www/html

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
- Dockerfile 빌드 - 현재 디렉터리에 Dockerfile로 존재
  - `docker build -t myphpapp:1.0 .`
- 컨테이너로 생성
  - `docker run -dit -p 8101:80 --name phpapp1 myphpapp:1.0`

### 이미지 빌드 과정
- 이미지 빌드는 사용자와의 대화식 처리가 아닌 자동화된 빌드
  - ubuntu 기반의 이미지를 생성하는 경우 패키지를 설치하고자 하면 반드시 apt-get update를 포함시켜야 하며 -y 옵션을 추가해서 자동으로 설치하도록 해주어야 함
- 이미지 빌드를 할 때는 현재 디렉터리에 있는 모든 파일 과 디렉터리의 컨텐츠는 도커 데몬에 빌드 컨텍스트로 전달되는데 이 때 제외하고 싶은 파일이나 디렉터리가 있으면 .dockerignore 파일에 기재하면 됨
- python 개발 환경을 만들기 위해서 ubuntu에 python 패키지를 설치
```
- 디렉토리를 생성하고 프롬프트 이동
mkdir python_lab
cd python_lab

- nano Dockerfile 명령으로 파일을 생성하고 작성
FROM ubuntu:18.04

RUN apt-get install python

- 이미지 빌드: 실패(apt-get update를 하지 않아서 실패)
docker build -t mypyapp:1.0 .

- nano Dockerfile 명령으로 파일을 생성하고 수정
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install python

이미지 빌드: 실패(대화식으로 작업하도록 해서 실패: 패키지를 설치할 때는 -y 옵션을 추가해서 자동으로 설치되도록 해주어야 합니다.)
docker build -t mypyapp:1.0 .

- nano Dockerfile 명령으로 파일을 생성하고 수정
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install python -y

- 이미지 빌드: 성공
docker build -t mypyapp:1.0 .
```
- 빌드 캐시 사용을 위한 실습
```
- Dockerfile을 생성
FROM ubuntu:latest

RUN apt-get update && apt-get install -y nginx curl vim

RUN echo 'Docker Container Application.' > /var/www/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

- 이미지 빌드
docker build -f Dockerfile -t webapp:1.0 .

- 이미지 빌드
docker build -f Dockerfile -t webapp:2.0 .

- 동일한 내용을 가지고 두번째 빌드를 하게 되면 cached 라고 하는 문장이 보이게 되는데 빌드 속도 향상을 위해서 실행 중간에 있는 이미지 캐시를 사용하게 되는데 이를 빌드 캐시라고 함
- 빌드 캐시를 할 때는 동일한 명령어의 연속된 부분까지 빌드 캐시를 이용함 
```
- Dockerfile 최적화
  - Ubuntu에 python을 설치하고 app.py 파일을 복사해서 실행하는 Dockerfile을 작성
    ```dockerfile
    FROM ubuntu:20.04
    COPY app.py /app
    RUN apt-get update && apt-get -y install python python-pip
    RUN pip install -r requirements.txt
    CMD python /app/app.py
    ```
  - Dockerfile을 빌드할 때 RUN, ADD, COPY 이 명령어들은 각각의 레이어를 생성하고 CMD, LABEL, ENV, EXPOSE는 레이어를 생성하지 않음
  - 빌드하면 시간이 오래 걸림
  - RUN 명령어들은 하나로 합칠 수 있다면 하나로 합치는게 좋음
  - 기반 이미지를 사용할 때 alpine 버전을 사용하면 용량이 작기 때문에 성능 향상에 도움이 됨
    ```dockerfile
    FROM python:3.9.2-alpine
    COPY app.py /app
    RUN apt-get update && apt-get -y install python-pip
    RUN pip install -r requirements.txt
    CMD python /app/app.py 
    ```
  - 기반 이미지에 개발 환경이 설치된 이미지를 활용하면 빌드 성능 향상에 도움이 됨
  - 빌드 캐시는 수정된 코드 이전까지만 적용되므로 변경될 가능성이 낮은 옵션을 상단에 배치해야 함

### 다양한 방법의 Dockerfile 작성
- 쉘 스크립트를 이용한 환경 구성: 환경 변수들을 sh 파일에 작성하고 이 파일을 실행시켜서 환경 구성을 수행
  - 디렉터리를 생성하고 프롬프트 이동
  - Dockerfile 생성 및 작성
```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get -y install apache2

RUN echo 'Docker Container Application' > /var/www/html/index.html

RUN mkdir /webapp

RUN echo './etc/apache2/envvars' > /webapp/run_http.sh && echo 'mkdir -p /var/run/apache2' >> '/webapp/ru>

EXPOSE 80

CMD /webapp/run_http.sh
```
- ADD 명령어의 자동 합축 해제 기능 활용
```
- 압축 파일 가져오기
sudo apt install git

git clone https://github.com/brayanlee/webapp.git

- 압축 파일이 있는 곳으로 프롬프트를 이동
cd webapp 

ls 명령으로 압축 파일이 있는지 확인

- Dockerfile이 저장될 디렉터리를 생성
mkdir dockerfiles

- Dockerfile 생성하고 작성: nano dockerfiles/Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get -y install apache2 vim curl

ADD webapp.tar.gz /var/www/html

WORKDIR /var/www/html

EXPOSE 80

CMD /usr/sbin/apachectl -D FOREGROUND

- 이미지 빌드
docker build -t webapp:8.0 -f ./dockerfiles/Dockerfile .

- 컨테이너 생성
docker run -dit -p 8201:80 --name webapp8 webapp:8.0

- 압축된 파일이 해제되었는지 확인
docker exec -it webapp8 /bin/bash

ls

- 소스 파일을 이미지에 복사하고자 하는 경우 파일 단위로 복사하는 것은 자원의 낭비이므로 디렉토리 전체를 복사하던가 압축된 파일의 형태로 복사하는 것이 효율적
- Dockerfile의 경로가 현재 디렉터리가 아니거나 현재 디렉터리에 있더라도 Dockerfile이라는 이름이 아닌 경우는 -f 옵션을 이용해서 파일 경로를 설정해 주어야 함
```
- Ubuntu Linux를 기본 이미지로 해서 패키지를 설치한 경우 용량을 축소하고자 하는 경우에는 RUN 명령에서 `apt-get clean`(설치에 사용한 패키지 라이브러리, 임시 파일, 오래된 파일을 삭제), `apt-get autoremove`(다른 패키지들의 종속성을 충족시키기 위해 자동으로 설치된 패키지를 삭제), `rm -rfv`(apt와 관련된 캐시를 삭제) 명령을 수행해서 불필요한 패키지를 제거해주면 됨
  - 이전에 사용한 도커 파일을 수정(nano dockerfiles/Dockerfile)
    ```dockerfile
    FROM ubuntu:latest

    RUN apt-get update && apt-get -y install apache2 vim curl && apt-get clean -y && apt-get autoremove -y && rm -rfv /var/lib/apt/lists/* /tmp/* /var/tmp/*

    ADD webapp.tar.gz /var/www/html

    WORKDIR /var/www/html

    EXPOSE 80

    CMD /usr/sbin/apachectl -D FOREGROUND


    - 이미지 빌드
    docker build -t webapp:9.0 -f ./dockerfiles/Dockerfile .

    - 이미지 크기 확인
    docker images   
    ```
### Python  Flask 애플리케이션 빌드
- ENV부분을 제외하고 ENTRYPOINT와 마지막 CMD 부분을 수정하면 모든 Python 애플리케이션을 빌드하는 방법은 비슷(Flask 는 python 파일명으로 실행)
- 디렉터리를 생성 `mkdir py_flask && cd py_flask`
- 소스 코드 작성(app/py_app.py)
```
=>디렉토리를 생성
mkdir py_flask && cd py_flask

=>소스 코드 작성(app/py_app.py)
mkdir app && nano app/py_app.py
from flask import Flask

py_app = Flask(__name__)

@py_app.route('/')
def python_flask():
        return "<h1>Hello Flask</h1>"

if __name__ == "__main__":
        py_app.run(host="0.0.0.0", port=9000, debug=True)
```
- 패키지 의존성을 위한 app/requirements.txt 생성
```
Flask==1.12
```
- 현재 디렉터리에 Dockerfile을 만들고 작성
```dockerfile
FROM python:3.8-alpine

RUN apk update && apk add --no-cache bash

RUN apk add --update build-base python3-dev py-pip

ENV LIBRARY_PATH=lib:/usr/lib
ENV FLASK_APP=py_app
ENV FLASK_ENV=development

EXPOSE 9000

WORKDIR /py_app
COPY ./app/ .

RUN pip install -r requirements.txt

ENTRYPOINT ["python"]
CMD ["py_app.py"]
```
- 이미지 빌드 `docker build -t py_flask:1.0 .`