---
title: Docker Compose
weight: 6
---
0. 컨테이너 와 이미지 모두 삭제
- 모든 컨테이너 중지: `docker stop $(docker ps -a -q)`
- 모든 컨테이너 삭제: `docker rm $(docker ps -a -q)`
- 모든 이미지 삭제: `docker rmi $(docker images -q)`
- 
## Dockerfile 작성 주의 사항
#### 안바뀔것 같은 것을 위에 작성
#### RUN 명령어는 한 번에 작성할 수 있는 것은 한번에 작성하는 것이 좋음, 이미지 용량을 줄이기 위해
#### GO는 다단계 빌드가 필요(빌드한 다음에 RUN하는지 그냥 RUN 하는지 확인 필요)
#### 빌드하고 RUN하는 언어는 다단계 빌드가 필요, 용량 줄이기 위해
#### GO는 아키텍쳐 주의 해야
#### ENTRYPOINT는 마지막 CMD명령어 받아서 진입점으로 실행
#### RUN과 CMD ENTRYPOINT는 유사

## DEVOPS
#### 개발을 클라우드환경에서 해보는 것 -> 운영 환경의 차이를 줄이기 위해

## docker compose 명령어
### 주의
- `docker compose`명령어는 docker compose 관련 yaml 파일이 있는 디렉터리에서 수행해야 함

### 명령어: docker compose help
- `build`: dockerfile을 이용한 빌드 또는 재빌드
- `config`: 도커 컴포즈 구성 파일의 내용 확인
- `create`: 서비스 생성
- `down`: 도커 컴포즈 자원을 일괄 정지 후 삭제
- `events`: 컨테이너에서 실시간으로 이벤트 정보를 수신
- `exec`: 컨테이너에게 명령을 실행(가장 많이 사용하는 명령은 `/bin/bash`나 `bash`)
- `help`: 도움말
- `images`: 사용된 이미지 정보
- `kill`: `docker kill`과 같은 명령으로 실행 중인 컨테이너 강제 중지
- `logs`: 컨테이너의 실행 로그 정보를 출력
- `pause`: 일시 정지
- `port`: 포트 바인된 외부로 연결된 포트 출력
- `ps`: 실행 중인 컨테이너 서비스 출력
- `pull`: 서비스 이미지 가져오기
- `push`: 서비스 이미지 업로드
- `restart`: 컨테이너 서비스 재시작(컨테이너 내부의 설정 파일을 변경한 경우)
- `rm`: 서비스 삭제
- `run`: 실행 중인 컨테이너에 일회성 명령어 실행
- `scale`: 컨테이너 서비스에 대한 컨테이너 수 설정
- `start`: 컨테이너 서비스 시작
- `stop`: 컨테이너 서비스 중지
- `top`: 실행 중인 프로세스 출력
- `unpause`: pause 된 서비스 정지 해제
- `up`: 컨테이너 서비스 생성 과 시작(옵션 활용 `-d`)
- `version`: 버전 정보 표시 및 종료

## Flask와 redis를 연동해서 액세스 카운팅 하는 앱을 docker compose로 생성
1. 디렉터리 생성 및 프롬프트 이동
```bash
$ mkdir flask_redis && cd $_
```

2. 파이썬 파일을 생성하고 작성(app 디렉토리를 생성하고 그 안에 py_app.py)
```bash
$ mkdir app
$ nano py_app.py
```

```python
import time
import redis
from flask import Flask

py_app = Flask(__name__)
db_cache = redis.Redis(host='redis', port=6379)

def web_hit_cnt():
        return db_cache.incr('hits')

@py_app.route('/')
def python_flask():
        cnt = web_hit_cnt()

        return  '''<h1 style="text-align:center; color:deepskyblue;">docker-compose app:
Flask & Redis</h1>
<p style="text-align:center; color:deepskyblue;">Good Container Service</p>
<p style="text-align:center; color:red;">Web access count : {} times</p>'''.format(cnt)

if __name__ == '__main__':
        py_app.run(host='0.0.0.0', port=9000, debug=True)
```

3. app 디렉토리의 requirements.txt 파이썬 패키지의 의존성 목록을 작성
```{filename="app/requirements.txt"}
Flask
redis
```

4. python application을 배포할 수 있는 Dockerfile을 생성하고 작성
- Dockerfile을 만드는 경우는 애플리케이션 코드를 직접 작성한 경우 나 별도의 개발환경을 만들기 위해서 입니다.
기본 이미지를 바탕으로 소스 코드를 추가해서 새로운 이미지를 만들거나 팀 단위 개발에서 동일한 개발환경을 만들고자 할 때 사용합니다.
```Dockerfile {filename="Dockerfile"}
FROM python:3.8-alpine

RUN apk update && apk add --no-cache bash

RUN apk add --update build-base python3-dev py-pip

ENV LIBRARY_PATH=/lib:/usr/lib
ENV FLASK_APP=py_app
ENV FLASK_ENV=development

EXPOSE 9000

WORKDIR /py_app
COPY ./app/ .
RUN pip install -r requirements.txt
ENTRYPOINT ["python"]
CMD ["py_app.py"]
```

5. 앱을 빌드해서 로컬에서 테스트
- 레디스 컨테이너 실행
```bash
$ docker run --name myredis -d -p 6379:6379 redis
```
- 애플리케이션 실행
```bash
$ python ./app/py_app.py
```
- 에러가 발생하면 redis의 접속 위치를 수정 - 자신의 IP로 수정

6. Dockerfile을 빌드해서 이미지로 생성해서 실행
- 이미지 빌드
```bash
$ docker build -t flaskapp .
```
- 이미지가 생성되었는지 확인
```bash
$ docker images
```
- 이미지를 컨테이너로 실행
```
$ docker run -d -p 9000:9000 --name=flaskapp flaskapp
```
- 컨테이너 확인
```
$ docker ps
```
- 브라우저에서 자신의IP:9000 으로 확인

- 모든 컨테이너 중지
```
$ docker stop $(docker ps -aq)
```

7. docker compose를 이용한 실행
- docker-compose.yml 파일을 생성하고 작성
```yml
version: '3'
services:
  redis:
    image: redis
    ports:
      - 6379:6379
    restart: always

  flask:
    build: .
    ports:
      - 9000:9000
    restart: always
    depends_on:
      - redis
```
- 도커 컴포즈 실행
```
$ docker-compose up -d
```
- 하나의 docker-compose 파일에 묶인 서비스들은 하나의 네트워크로 만들어지게 되는데 이 때 서브넷 마스크만 조정되는 것이 아니고 하나의 DNS가 만들어져서 서비스 이름을 이용해서 접근이 가능합니다.

- 현재 실행 중인 컨테이너를 제거
```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```
- 소스 코드에서 redis 접속 IP를 redis로 수정
```python {filename="app/py_app.py"}
import time
import redis
from flask import Flask

py_app = Flask(__name__)
db_cache = redis.Redis(host='redis', port=6379)

def web_hit_cnt():
        return db_cache.incr('hits')

@py_app.route('/')
def python_flask():
        cnt = web_hit_cnt()

        return  '''<h1 style="text-align:center; color:deepskyblue;">docker-compose app:
Flask & Redis</h1>
<p style="text-align:center; color:deepskyblue;">Good Container Service</p>
<p style="text-align:center; color:red;">Web access count : {} times</p>'''.format(cnt)

if __name__ == '__main__':
        py_app.run(host='0.0.0.0', port=9000, debug=True)
```
- docker-compose 파일을 수정해서 redis를 외부에 노출시키지 않고 사용
  - 동일 네트워크로 묶이면 IP 대신에 서비스 이름을 사용할 수 있고 ports 대신에 expose를 이용해서 동일 네트워크 내부에서만 접근하도록 할 수 있음
  - docker-compose.yml 파일을 수정
```yml
version: '3'
services:
  redis:
    image: redis
    expose:
      - "6379"
    restart: always
    networks:
      - our_net
  flask:
    build: .
    ports:
      - 9000:9000
    restart: always
    links:
      - redis
    depends_on:
      - redis
    networks:
      - our_net
networks:
  our_net: {}
```
8. docker compose up
- 옵션
```
-d(--detach) : 백그라운드 컨테이너 서비스를 실행하고 새로 생성된 컨테이너 이름을 화면에 출력
--build : 컨테이너 서비스를 시작하기 전에 이미지를 빌드하는 것으로 Dockerfile이나 기타 소스 코드 변동이 있을 때 수행
--force-recreate : 도커 컴포즈 야믈 코드 및 이미지가 변경되지 않은 경우에도 컨테이너를 다시 생성
-t(--timeout) : 현재 실행 중인 컨테이너를 종료하는 경우 이 시간을 이용해서 타임아웃이 발생(기본값은 10)
--scale 서비스이름=개수: 컨테이너 서비스의 개수를 지정 수 만큼 확장
-f : docker compose 파일의 경로가 다르거나 이름이 다를 때 이 옵션을 이용해서 지정
```
- scale up dowdn
  - 디렉터리를 생성하고 이동
  - docker-compose.yml 파일을 생성하고 작성
```yml
version: '3.8'
services:
  server_web:
    image: httpd:2
  server_db:
    image: redis
```
  - docker-compose 실행 
    ```
    docker compose up -d
    ```
  - docker-compose 실행 후 docker-compose ps 로 확인
    ```
    docker-compose up --scale server_db=3 --scale server_web=3 -d
    ```
9. docker-compose down
- 컨테이너 서비스, 볼륨, 네트워크 모두를 정지시킨 후 서비스를 삭제
- `--rmi all` 을 사용하면 이미지도 삭제
- `--volumes`를 사용하면 볼륨도 삭제

9. docker-compose stop
- 특정 컨테이너를 중지시킬 때 사용

10. docker-compose start
- 정지된 컨테이너 실행할 때

11. docker-compose logs
- 로그를 출력해주는데 `-f` 옵션을 이용하면 실시간 로그를 출력

## Flask와 nginx를 이용한 컨테이너 로드밸런서 구축
- 도커는 HaProxy, nginx/apache load balancer 등 외부 서비스 와 컨테이너를 결합한 로드밸런싱 기능 구현이 가능
- nginx 로드밸런싱 알고리즘
  - Round Robin: 순서대로 번갈아 가면서 처리, 기본<br>
  - Least Connections: 연결 개수가 가장 적은 곳에 배정
  - Least Time: 평균 지연 시간이 가장 낮은 곳에 배정<br>
  - IP hash: IP를 해시 함수에 대입해서 그 결과를 가지고 배정
  - General hash: IP, Port, URI 문자열 등 사용자가 지정<br>
  - Random: 무작위

- nginx로드 밸런싱 파라미터
  - weight: 가중치를 설정해서 서버 간 요청 분배(기본값 1)
  - max_conns: 최대 클라이어언트 연결 개수
  - queue: 대기열 생성
  - max_fails: 최대 실패 횟수로 임계치 도달 시 해당 서버를 분배 대상에서 제외
  - fail_timeout: 응답 최소 시간을 설정하는 것으로 max_fails와 같이 사용
  - backup: 평소에는 동작하지 않고 모든 서버가 동작하지 않을 때 사용
  - down: 기본적으로는 사용되지 않지만 ip_hash인 경우만 사용

- nginx를 웹 서버가 아닌 프록시로 구성하고 애플리케이션은 Flask로 생성해서 nginx에게 요청을 하면 Flask 앱을 호출하는 방식으로 동작



2. 디렉터리 구조를 생성
- alb 디렉터리를 생성하고 그 안에 nginx_alb, pyfla_app1, pyfla_app2, pyfla_app3

3. nginx 작업
- nginx_alb 디렉터리로 이동
- Dockerfile 생성한 후 작성
```dockerfile
FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
- nginx 설정 파일을 생성(nginx.conf)
```conf {filename="nginx.conf"}
upsteream web-alb{
        server 172.17.0.1:5001;
        server 172.17.0.1:5002;
        server 172.17.0.1:5003;
}

server{
        location /{
                proxy_pass http://web-alb;
        }
}
```

4. flask 작업
- pyfla_app1에서 작업
  - requirements.txt 를 생성하고 작성
    ```
    blinker==1.6.3
    click==8.1.7
    Flask==3.0.0
    ```
  - Dockerfile을 생성하고 작성
    ```Dockerfile
    FROM python:3
    COPY ./requirements.txt /requirements.txt
    WORKDIR /
    RUN pip install -r requirements.txt
    COPY . /
    ENTRYPOINT ["python"]
    CMD ["pyfla_app1.py"]
    ```
  - pyfla_app1.py 파일을 생성하고 작성
```python {filename="pyfla_app1.py"}
from flask import request, Flask
import json
app1 = Flask(__name__)

@app1.route("/")
def hello_world():
  return  "Web Application [1]" + "\n"

if __name__ == "__main__":
  app1.run(debug=True, host='0.0.0.0')
```
- pyfla_app2, pyfla_app3에서 똑같이 작업

5. alb 디렉토리에 docker-compose.yml 파일을 만들고 작성
```yml
version: "3"
services:
  pyfla_app1:
    build: ./pyfla_app1
    ports:
      - "5001:5000"

  pyfla_app2:
    build: ./pyfla_app2
    ports:
      - "5002:5000"

  pyfla_app3:
    build: ./pyfla_app3
    ports:
      - "5003:5000"

  nginx:
    build: ./nginx_alb
    ports:
      - "8080:80"
    depends_on:
      - pyfla_app1
      - pyfla_app2
      - pyfla_app3
```
6. 실행 및 확인
- `docker-compose up -d`

7. 변경 작업
- nginx.conf 파일을 수정
```conf
upstream web-alb {
        ip_hash;

        server 172.17.0.1:5001;
        server 172.17.0.1:5002;
        server 172.17.0.1:5003;
}

server{
        location /{
                proxy_pass http://web-alb;
        }
}
```
- docker-compose up을 재실행
  - 이전과 동일한 결과를 출력
  - 변경된 내용이 적용되지 않음

- 컨테이너를 다시 생성하고 docker-compose up 수행
```
docker-compose build
docker-compose up -d
```

## GitHub Action을 이용한 이미지를 DockerHub에 자동 배포
1. 리눅스에 git 설치: `sudo apt install git`
2. 유저 정보 등록
```
git config --global user.name itstudy001
git config --global user.email itstudy@kakao.com
```
3. github에 올릴 코드를 작성: main.go
```go
package main

import(
        "fmt"
        "time"
)

func main() {
        for{
                fmt.Println("Hello World")
                time.Sleep(10 * time.Second)
        }
}
```
- go 프로그램 빌드
```
go build.main.go
```
- go 프로그램 실행
```
./main
```

4. Dockerfile 작업
- Dockerfile 생성 및 작성
```dockerfile
FROM golang:1.13-alpine as builder
RUN apk update && apk add git

WORKDIR /usr/src/app
COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-s' -o main .

FROM scratch

COPY --from=builder /usr/src/app .
CMD ["/main"]
```
- 이미지 생성
```
docker build -t goapp:1.0 .
```
- 컨테이너 생성
```
docker run --name goapp-deploy goapp:1.0
```
5. 코드를 저장하기 위한 git hub 작업
- git hub에서 토큰 발급(한 번 발급받으면 토큰을 다시 보여주지 않음) 
- repository 생성 https://github.com/dragonhail/newrepo.git

6. code를 git에 업로드
- git 초기화
```
git init
```
- 업로드할 코드 추가
```
git add .
```

- commit
```
git commit -m "cicd"
```

- github에 올리기 위해서는 일반적으로 branch를 main으로 수정
예전에는 github의 기본 브랜치가 master 여서 바로 올리면 되었는데 지금은 main
```
git check out -b main
```
- 원격 저장소 연결
```
git remote add or origin 원격저장소경로
```
- push
```
git push origin main
```
- 이 때 github 아이디와 비밀번호를 물어보는데 비밀번호는 토큰 값을 입력해야 함

7. docker hub 준비(hub.docker.com)
- repository 생성
  - dockercicd
- token 발급

1. git hub에서 유저 이름 과 token 을 secret 으로 생성
- DOCKERHUB_TOKEN과 DOCKERHUB_USERNAME

1. 소스 코드가 있는 디렉터리에 .github/workflows 디렉터리를 생성하고 yaml파일을 만들어서 작성: 파일 이름은 상관 없음

```yaml
name: "dockercicd"

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set Up GO
        uses: actions/setup-go@v3
        with:
          go-version: 1.15
      - name: Build
        run: go build -v ./…

      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: build and release to DockerHUb
        env:
          NAME: dragonhailstone
          REPO: dockercide
        run: |
          docker build -t $REPO .
          docker tag $REPO:latest $NAME/$REPO:latest
          docker push $NAME/$REPO:latest
```