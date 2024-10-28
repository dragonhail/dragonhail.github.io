---
title: Build & Registry & docker compose
weight: 5
---
```
0. 컨테이너 와 이미지 모두 삭제
=>모든 컨테이너 중지: docker stop $(docker ps -a -q)
=>모든 컨테이너 삭제: docker rm $(docker ps -a -q)

=>모든 이미지 삭제: docker rmi $(docker images -q)
```

## 빌드 의존성 제거와 다단계 빌드
- 다단계 빌드는 FROM 명령을 이용해서 여러 단계의 빌드 과정을 만들고 다른 단계에 AS를 이용해서 이름을 부여해서 사용할 수 있도록 하는 것
- 다른 단계에서 생성된 결과 중 애플리케이션 구동에 필요한 특정 데이터만 가져올 수 있기 때문에 이미지를 경량화
- 다단계 빌드로 작성된 이미지는 모든 빌드 의존성이 하나의 환경에 포함되므로 빌드 의존성을 제거할 수 있음

### go 애플리케이션을 다단계 빌드로 이미지를 생성
- 디렉토리를 생성하고 그 디렉토리로 프롬프트를 이동
```mkdir goapp && cd $_```
- go 파일을 생성(goapp.go)
```go {filename="goapp.go"}
package main

import(
"fmt"
"time"
)

func main(){
        for{
                fmt.Println("Hello World")
                time.Sleep(10 * time.Second)
        }
}
```
- golang 설치
  - 다운로드: wget https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
  - 압축해제: sudo tar -xzf go1.19.1.linux-amd64.tar.gz -C /usr/local
  - 압축해제된 go/bin 디렉토리를 PATH에 등록(명령어 또는 파일을 아무 곳에서나 사용할 수 있게 하기 위해서)
  - `sudo nano /etc/profile` 명령을 수행하고 아래 내용 추가
    - `export PATH=$PATH:/usr/local/go/bin`
  - 위의 내용을 추가한 후 저장하고 `source /etc/profile` 명령으로 변경 내용 적용
  - 버전 확인 `go version`
- 이전에 만든 go 파일 실행
  - 빌드: `go build 파일명`
  - 실행: ./파일명에서 확장자를 제거한 부분
  - node go C java는 빌드 과정이 있음, python은 없음
- Dockerfile을 생성해서 작성
```dockerfile
FROM golang:1.15-alpine3.12 AS gobuilder-stage

WORKDIR /usr/src/goapp

COPY goapp.go .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /usr/local/bin/gostart

FROM scratch AS runtime-stage

COPY --from=gobuilder-stage /usr/local/bin/gostart /usr/local/bin/gostart

CMD ["/usr/local/bin/gostart"]

```
- Dockerfile 빌드
  - docker build -t 이미지이름:태그 [-f Dockerfile 경로] .
- 이미지 확인
  - `docker images`
- 컨테이너로 실행
  - `docker run --name goapp-deploy goapp:1.0`

## Private Registry
- Docker Hub는 기본적으로 public으로 저장소가 만들어지기 때문에 주소만 알면 아무나 접근이 가능
- Docker Hub에서도 Private 저장소를 제공하는데 이 저장소는 1개만 무료이고 그 이후부터는 유료
- Docker Hub에서는 private registry를 위한 registry라는 이미지를 제공하는데 이 이미지를 컨테이너로 생성하고 이 컨테이너에 이미지를 저장하는 개념
- 단순 텍스트 방식만 지원하기 때문 웹 기반의 검색을 할려면 GUI 인터페이스를 제공하는 다른 컨테이너와 결합을 해야 함

### private registry를 만들고 이미지를 저장할 수 있는 서버 생성
- 이미지 조회
  - ```docker search registry```
- private registry를 구성할 수 있는 이미지를 다운로드
  - ```docker pull registry```
- 이미지를 업로드 할 수 있는 저장소 사용을 위한 설정을 /etc/init.d/docker 파일의 DOCKER_OPTS 항목에 추가
  - ```sudo nano /etc/init.d/docker```
  - DOCKER_OPTS=--insecure-registry 컴퓨터의IP:포트번호
  - ```DOCKER_OPTS=--insecure-registry 10.0.2.15:5000```
- 도커를 재시작
  - ```sudo service docker restart```
- registry를 컨테이너로 실행
  - ```docker run --name local-registry -d -p 5000:5000 registry```
- 확인
  - ```docker ps```

### private registry를 사용할 수 있는 클라이언트 작업
- private registry를 사용할 수 있도록 설정을 수정:  /etc/docker/daemon.json  없으면 생성
- `sudo nano /etc/docker/daemon.json` 명령으로 파일을 열고 추가
  - `{"insecure-registries":["localhost:5000"] }`
- 도커 재시작
  - `sudo service docker restart`
- 도커 정보 확인
  - `docker info`
- private registry 의 이미지 확인
  - `curl -XGET localhost:5000/v2/_catalog`
- 이미지 다운로드
  - `docker image pull mysql`

- 업로드 할 이미지 태그 만들기
  - `docker tag mysql:latest localhost:5000/mysql:latest`

- 이미지 푸시
  - `docker push localhost:5000/mysql`

- 저장소 확인
```
curl -XGET localhost:5000/v2/_catalog
curl -XGET localhost:5000/v2/mysql/tags/list
```

### private registry를 GUI를 이용해서 조회
- 이미지 다운로드
  - hyper/docker-registry-web
- 컨테이너로 생성 
  - `docker run -dit -p 8080:8080 --name registry-web --link local-registry -e REGISTRY_URL=http://localhost:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web`
- UTM 이나 Cloud 환경에서의 작업을 수행할 때는 localhost 부분을 실제 IP로 대체해서 수행

## Docker-Compose
- 도커 컴포즈는 공통성을 갖는 컨테이너 애플리케이션 스택을 yaml 코드로 정의하는 정의서이고 그것을 실행하기 위한 다중 컨테이너 실행 도구
- 공통성은 동일한 목적을 달성하기 위한 성질
- Web Application을 만들고자 하는 경우 3-Tier로 구성을 하게 되는데 데이터를 저장하기 위해서 MySQL 이나 Oracle을 사용하고 API Application 설정을 위해서 백엔드를 Flask 나 Fast API, Node 등으로 구성을 할 것이고 사용자 인터페이스는 react 나 vue를 사용하게 됨
이 3개의 애플리케이션이 공통성을 갖는다고 볼 수 있음
- 위처럼 공통성을 갖는 애플리케이션 스택을 도커 컴포즈 야믈 코드로 정의해서 한 번에 서비스를 올리고 관리할 수 있는 도구가 도커 컴포즈
- 도커 컴포즈로 실행된 컨테이너는 독립된 기능을 가지며 공통 네트워크로 구성되기 때문에 컨테이너 간 통신이 쉬움
- 도커 컴포즈는 테스트, 개발, 운영의 모든 환경에서 구성이 가능한 오케스트레이션 도구 중 하나
- 다양한 관리 기능을 가지고 있지 않기 때문에 테스트 와 개발 환경에 적합
- 실제 운영환경에서는 많은 관리적 요소가 필요하기 때문에 도커 스웜이나 쿠버네티스 와 같은 오케스트레이션 도구가 가지고 있는 자동 확장, 모니터링, 복구 등의 운영에 필요한 기능과 함께 사용하는 것을 권장
- 운영환경에서는 애플리케이션 사이트에서 다운로드에 많이 사용

### 설치
- Windows 나 Mac 에서 Docker Desktop을 다운로드 받아서 사용하는 경우는 포함되어 있음
- Linux에서는 github에서 다운로드 받아서 사용
  - `sudo curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
  - 실행 권한 부여 `sudo chmod +x /usr/local/bin/docker-compose`
  - 실제 위치를 직접 입력하지 않고 실행할 수 있도록 설정(PATH에 추가하거나 기존에 설정된 PATH(/usr/bin)에 심볼릭 링크를 설정)
    - `sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`
  - 버전 확인 `docker-compose -v`
  - 다른 버전을 설치하고자 해서 삭제하고자 하는 경우
    ```
    sudo rm /usr/local/bin/docker-compose
    sudo rm /usr/bin/docker-compose
    ```

### yaml(야믈, yml)
- 사용자가 쉽게 읽고 쓸 수 있도록 만든 텍스트 구조
- 야믈은 들여쓰기를 통해서 계층을 나누며 공백 수 로 블록을 구분
- YAML Ain’t Markup Language
- JSON은 주석이 안되지만 YAML은 주석이 가능
- JSON은 한글을 사용할 때 인코딩을 해야 하지만 YAML은 그대로 사용
- JSON은 주로 API 작성 시 사용하지만 YAML은 환경 구성 등의 설정 파일 작성 시 이용(이전에 XML로 수행)
- 들여쓰기는 탭이 아니고 공백으로 정확하게 구분
- 배열은 -를 앞에 붙여서 작성

### 실습
- 디렉터리를 생성: compose/db-data
- docker-compose.yaml이나 docker-compose.yml, compose.yaml, compose.yml
```yml
version: '3.3'
services:
  mydb:
    image: mariadb:10.4.6
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=wnddkd
      - MYSQL_DATABASE=appdb
    volumes:
      - ./db-data:/var/lib/mysql
    ports:
      - '3306:3306'
```
- 컨테이너로 실행하고자 하는 경우는 `docker compose up`
- docker-compose로 실행하게 되면 자동으로 network가 생성됨
- 컨테이너를 확인하는 명령은 `docker ps` 또는 compose 파일이 있는 디렉터리에서 `docker compose ps`
- 컨테이너의 bash에 접속
```bash
docker exec -it 컨테이너이름 /bin/bash
```
- mysql에 루트 계정으로 접속
```
mysql -uroot -p
```
- 데이터베이스 확인
```
show databases;
```
- docker-copose중지
  - compose 파일이 존재하는 디렉터리에서 `docker compose down`

### 확인
- `docker compose up` 명령을 수행할 때 `-d` 옵션이 없으면 포그라운드 수행인데 프롬프트가 작업이 끝날때 까지 돌아오지 않음
계속해서 작업을 수행해야 하는 경우에 프롬프트로 돌아오고자 하면 `-d` 옵션을 사용
- `docker compose up` 명령을 사용하면 기본적으로 디렉토리이름_default 라는 네트워크를 생성
- 자동으로 생성된 네트워크 때문에 내부에서 통신할 때는 IP가 아닌 서비스명(컨테이너이름)으로 통신이 가능 
- 네트워크는 직접 생성해서 연결하는 것도 가능

### docker compose 파일 작성 요령
- version
  - 버전을 명시하는 부분으로 도커 엔진 릴리즈와 연관이 됨
  - 3.3을 적었는데 이렇게 되면 도커 엔진이 17 버전 이상이어야
  - 도커 컴포즈 버전 과 도커 엔진 버전이 어느 정도 맞아야함
  - 버전이 맞지 않으면 ERROR: Version in  "./docker-compose.yaml" is unsupported 에러가 발생함
  - 이 에러가 발생하면 도커 컴포즈가 오래되었거나 도커 엔진과 버전이 맞지 않거나 들여쓰기의 공백 수가 하위 레벨과 맞지 않을 때
- services
  - 실행할 컨테이너 서비스를 작성하는 영역
  - 프로젝트에서 별도 이미지 개발없이 도커 허브에서 제공하는 공식 이미지를 사용하는 경우는 image:공식이미지 이름만 기재하면 됨
  - 공식 이미지 nginx를 이용하고자 하는 경우
    ```yml
    services:
     myweb:
      image: nginx:latest
     mydb:
      image: mariadb:10.4.6
    ```
  - Dockerfile로 별도의 이미지를 만드는 경우에는 build 옵션을 이용, 현재 디렉터리에 Dockerfile이라는 이름으로 만들어진 경우에는 build: .을 설정해주면 되고 파일 이름이 다르거나 다른 디렉터리에 존재하는 경우에는 context와 dockerfile 이라는 하위 옵션을 이용
    ```yml
    web:
     build: .

    web:
     build:
      context: .
      dockerfile: ./compose/pyfla/Dockerfile-py
    ```
  - container_name: 컨테이너 이름으로 생략하면 "디렉토리이름_서비스이름_숫자" 으로 생성되는데 docker run 의 --name 옵션과 동일
  - ports: 서비스 내부 포트 와 외부 호스트 포트를 지정하여 바인딩 하는 옵션으로 docker run 의 -p 와 동일
  - expose: 호스트 운영체제 와 직접 연결하는 포트를 구성하지 않고 서비스만 포트를 노출하는 것으로 필요시 링크로 연결된 서비스 와 서비스 간의 통신을 할 때 사용
  - networks: 최상위 레벨의 networks에 정의된 네트워크 이름을 작성하는 것으로 docker run --net 또는 network 와 동일
생략하면 디렉토리이름_default 네트워크에 연결
  - volumes: 서비스 내부 디렉토리 와 호스트 디렉토리 연결할 때 사용하는 것으로 docker run 의 -v 나 --volume 옵션과 동일
  - environment: 서비스 내부의 환경 변수를 설정하는 것으로 환경 변수가 많은 경우는 파일로 만들어서 env_file 옵션에 파일 명을 지정하는데 docker run -e 옵션과 동일
    ```yaml
    services:
     myweb:
      environemnt:
       env_file:./envfile.env
    ```
  - command: 서비스가 구동 된 다음 실행할 명령어를 작성하는 것으로 docker run 마지막에 작성한 내용
    ```yaml
    command: /bin/bash
    ```
  - restart: 서비스 재시작 옵션으로 docker run 의 --restart 와 동일
    ```
    no: 수동으로 재시작
    always: 컨테이너 수동 제어를 제외하고 항상 재시작
    on-failure: 오류가 있을 때 재시작
    ```
  -  depends_on: 서비스 간의 종속성을 의미하며 먼저 실행해야 하는 서비스를 지정, 이 옵션에 지정된 서비스가 먼저 시작됨
    ```yml
    services:
     myweb:
      depends_on: mydb
     mydb:
    ```
- 네트워크 정의
  - 다중 컨테이너들이 사용할 최상위 네트워크 키를 정의하고 이하 하위 서비스 단위로 이 네트워크를 선택할 수 있음
  - networks 옵션을 지정하지 않으면 기본 네트워크가 자동으로 생성됨
  - 도커에서 생성한 기존 네트워크를 지정하는 경우는 external 옵션에 name 속성에 기재
  - 이미 도커 엔진에서 만든 vswitch-ap라는 네트워크를 docker-compose에서 사용하고자 하는 경우는
    ```yml
    services:

    networks:
     default:
      external:
       name: vswitch-ap
    ```
- 볼륨 정의
```yml
volumes:
  db_data:{}
  web_data:{}
```

### EXPOSE와 ports의 차이
```{filename="expose와 ports 차이"}
expose는 내부에서만 사용하기 위해
ports는 외부로 개방하기 위해
------------------------------------------------------------
|db(1521) <------> backend(1521) <----> frontend (80 , 443)|------ 외부
------------------------------------------------------------
expose:1521        expose:1521           ports: -80 -443
```
```{filename="backend CDN이 많아질 경우"}
frontend가 기억해야 할 주소가 많아짐->API Gateway를 이용
|-------------------------------|
|backend API------->            |
|                    API Gateway---------> Client
|backend API------->            |
|-------------------------------|
```

### 도커 명령어 와 도커 컴포즈 야믈 비교
- wordpress: 블로그를 만드는 것을 도와주는 프레임워크로 데이터베이스(mysql 이나 mariadb) 연결이 필수
```
              도커 명령어	도커 컴포즈 옵션
컨테이너이름	--name		container_name
포트 연결		-p		    ports(배열)
네트워크구성	--net		networks(배열)
재시작		   --restart    restart
볼륨			-v		    volumes(배열)
환경변수		-e		    environment(배열)
컨테이너간연결  --link	    depends_on
이미지		이미지이름		image
```
  - 요구사항: mysql 8.0 과 wordpress 5.7 연동
- 도커 명령어로 구성
  - 각각의 데이터를 영구적으로 저장하기 위한 볼륨을 생성
    ```bash
    $ docker volume create mydb_data

    $ docker volume create myweb_data

    $ docker volume ls

    #실제 저장 위치 확인

    $ docker inspect --type volume mydb_data
    또는
    $ docker inspect volume mydb_data
    ```
  - 2개의 컨테이너를 묶어 줄 네트워크를 생성(IP는 자동 설정)
    ```bash
    $ docker network create myapp_net

    $ docker network ls

    $ docker network inspect myapp_net
    ```
  - MySQL8.0 컨테이너를 생성
    ```bash
    $ docker run -dit --name=mysql_app -v mydb_data:/var/lib/mysql --restart=always -p 3306:3306 --net=myapp_net -e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=wpdb -e MYSQL_USER=wpuser -e MYSQL_PASSWORD=wppassword mysql:8.0
    ```
  - wordpress:5.7컨테이너를 생성
    ```bash
    docker run -dit --name=wordpress_app -v mywebdata:/var/www/html -v ${PWD}/myweb-log:/var/log --restart=always -p 8888:80 --net=myapp_net --link mysql_app:mysql -e WORDPRESS_DB_HOST=mysql_app:3309 -e WORDPRESS_DB_NAME=wpdb -e WORDPRESS_DB_USER=wpuser -e WORDPRESS_DB_PASSWORD=wppassword wordpress:5.7
    ```
  - 구동 확인을 위해서 localhost:8888 로 접속
- docker-compose.yml 로 동일한 작업
  - 디렉토리 생성 및 이동
    ```bash
    mkdir my_wp && cd $_
    ```
  - docker-compose.yml 작성
```yml
version: "3.9"
services:
  mydb:
    image: mysql:8.0
    container_name: mysql_app
    volumes:
      - mydb_data:/var/lib/mysql
    restart: always
    ports:
      - "3306:3306"
    networks:
      - backend-net
    environment:
      MYSQL_ROOT_PASSWORD: password#
      MYSQL_DATABASE: wpdb
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
  myweb:
    image: wordpress:5.7
    container_name: wordpress_app
    ports:
      - "8888:80"
    networks:
      - backend-net
      - frontend-net
    volumes:
      - myweb_data:/var/www/html
      - ${PWD}/myweb-log:/var/log
    restart: always
    environment:
      WORDPRESS_DB_HOST: mydb:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wpdb
    depends_on:
      - mydb

volumes:
  mydb_data: {}
  myweb_data: {}

networks:
  frontend-net: {}
  backend-net: {}
```
  - 실행
```
docker-compose up -d
```
  - volume 에러가 나면 volume 지우고 수행
