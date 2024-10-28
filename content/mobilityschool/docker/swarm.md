---
title: Docker Swarm
weight: 7
---
### 리뷰
0. 컨테이너 와 이미지 모두 삭제
- 모든 컨테이너 중지: `docker stop $(docker ps -a -q)`
- 모든 컨테이너 삭제: `docker rm $(docker ps -a -q)`

- 모든 이미지 삭제: `docker rmi $(docker images -q)`

### 1. Docker Network
- Local DNS 생성: 이름으로 서비스 접속 가능
- 서브네팅

### 2. Docker Layer
- 재사용이 가능 로컬 컴퓨터에서
- 클라우드 환경에서는 딱히
- 레이어는 줄이는게 좋음

### 3. CICD는 대화식 불가
- 자동화 불가, 사람이 계속 대기해야

### 4. Dockerfile
- Dockerfile에서 `apt install` 시 `-y` 옵션 항상 필요
- `RUN apt-get update && RUN apt-get install -y --no-install-recommends && RUN rm -rf /var/lib/apt//lists/*`
  - 리눅스 업데이트 명령어, 항상 같음
- Dockerfile의 WORKDIR은 아무거나 써도 상관 없음
### 5. RUN과 CMD 차이
- RUN은 일반적인 스크립트로 처리
- CMD는 묶어서 처리 가능

## Application을 github에 push 하고 이를 Dockerfile을 이용해서 이미지로 만들어서 DockerHub에 자동으로 배포하기

### 1. 애플리케이션 생성 및 테스트
- 가상환경 생성 패키지 설치
```bash
sudo apt install python3-venv
```
- 가상환경 생성
```bash
python3 -m venv 1022
```
- 가상환경 활성화: 맨 앞에 (가상환경이름) 이 보이는지 확인 
```bash
cd 1022
source myvenv/bin/activate
```
- 이 소스 코드를 실행할 때는 가상환경 안에서 실행을 하던가 아니면 이 가상환경에 설치된 외부 라이브러리를 다시 설치한 후 실행해야 합니다. 
- Django API Server를 만들기 위한 패키지를 설치 `pip install django djangorestframework`
- 프로젝트 생성
```bash
django-admin startproject apiserver
```
- 실행
```bash
cd apiserver

python manage.py runserver
```
- settings.py 파일을 수정
```{filename="apiserver/apiserver/settings.py"}
ALLOWED_HOSTS = ['*'] 추가
# *은 아무 컴퓨터에서나 실행이되고 IP를 직접 설정하면 그 IP에서만 실행이 됨

# 사용할 애플리케이션 등록
INSTALLED_APPS에 
'apiserver',
'rest_framework',
추가

# 사용할 시간대 설정
TIME_ZONE = 'Asia/Seoul' 수정
```

- 브라우저에서 확인 http://localhost:8000

- 사용자의 요청을 처리할 views.py 파일을 apiserver 디렉토리 내부에 생성하고 작성
```python {filename="apiserver/apiserver/views.py"}
from rest_framework.response import Response
from rest_framework.decorators import api_view
from rest_framework import status

@api_view(['GET'])
def index(request):
    data = {"result":"success", "data":[{"id":"itstudy", "name":"adam"}]}
    return Response(data, status=status.HTTP_200_OK)
```
- urls.py 파일을 수정해서 요청 과 요청 처리 함수를 연결
```python {filename="urls.py"}
from django.contrib import admin
from django.urls import path
from .views import index

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', index),
]
```
- 실행 `python manage.py runserver 0.0.0.0:8000`

### 2. Dockerfile을 이용해서 이미지를 만들고 컨테이너로 테스트
- apiserver/에서 의존성을 requirements.txt 파일에 출력
```pip freeze > requirements.txt```

- 번 도커 파일은 마지막 CMD 부분을 제외하고 모든 파이썬 파일에 적용 가능

```dockerfile
FROM --platform=linux/amd64 python:3.8-slim-buster as build

RUN apt-get update && apt-get install -y --no-install-recommends && rm -rf /var/lib/apt//lists/*

WORKDIR /usr/src/app

COPY requirements.txt ./

RUN pip install -r requirements.txt

COPY . .

EXPOSE 80

CMD ["python", "manage.py", "runserver", "0.0.0.0:80"]
```
- 빌드를 해서 도커 이미지로 생성
```bash
docker build -t apiserver .
```

- 패키지 설치하다가 실패하면 pip install django djangorestframework 로 수정해서 빌드

- 컨테이너로 실행
```bash
docker run -d -p 80:80 --name apiserver apiserver
```
### 3. 코드를 git hub에 주기적으로 업로드 - CI
- git hub에 접속해서 repository를 생성하고 token을 발급(1번만 해두면 토큰을 계속해서 사용하는 것이 가능: 이전에는 아이디 와 비번을 이용해서 로그인을 했는데 이제는 아이디 와 토큰을 이용해서 로그인)받기

- repository 이름: djangocicd
- apiserver/apiserver 디렉토리에서 수행
```bash
# git 초기화
git init

# git에 변경 내용을 적용하기 위한 파일 등록
git add .

# 로컬 git에 변경 내용 반영
git commit -m ‘메시지’

# 브랜치 변경: 한번만 수행
git branch -M main

# 원격 저장소 등록
git remote add origin 저장소URL

# 원격 저장소에 업로드
git push origin main
```

### 4. git hub에 업로드 될 때 docker hub에 배포
- docker hub에 로그인을 해서 repositoy 와 token을 생성
  - repo: djangocicd

- git이 업로드된 디렉토리에 .github/workflows 디렉토리를 생성하고 yml 파일을 생성해서 작성, 이름은 상관 없음
```yml
name: djangocicd

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Set up Python 3.10
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: |
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: build and release to DockerHub
        env:
          NAME: ${{ secrets.DOCKERHUB_USERNAME }}
          REPO: djangocicd
        run: |
          docker build -t $REPO .
          docker tag $REPO:latest $NAME/$REPO:latest
          docker push $NAME/$REPO:latest
```
- github의 repository 에 2개의 변수 등록
  - DOCKERHUB_USERNAME
  - DOCKERHUB_PASSWORD
- 코드를 수정한 경우
```bash
git add .
git commit -m '메시지'
git push origin main
```

- git hub의 설정을 변경한 경우
  - Actions 탭에서 rerun-jobs를 눌러서 빌드를 다시
- 확인
  - `docker run -d -p 80:80 --name djangocicd dragonhailstone/djangocicd`

## 교재 내용 중 Health Check 와 명령어 종속 및 모니터링
- 교재 샘플 복사: git clone https://github.com/gilbutITbook/080258
### HEALTHCHECK
- 컨테이너의 프로세스 상태를 체크하고자 하는 경우에 작성
- 여러 개의 명령어를 지정할 수 있지만 마지막 명령어만 적용
- Dockerfile에 작성하게 되는데 명령어 형태로 작성
- 옵션
```
--interval: 헬스 체크 간격을 설정하는 것으로 기본값이 30s
--timeout: 타임 아웃 값으로 응답이 오는데 기다리는 시간으로 기본값이 30s
--retries: 재시도 횟수로 기본값을 3

HEALTHCHECK --interval=1m --timeout=3s --retries=5 CMD curl -f http://localhost || exit 1
```
- 3번 요청을 하면 중지되는 컨테이너를 실행
- 이미지는 diamol/ch08-numbers-api
- 포트는 80번 포트 연결
- `docker run -d -p 8080:80 --name numbersapi diamol/ch08-numbers-api`
- 요청을 4번 전송
  - curl http://localhost:8080/rng
  - 3번은 랜덤한 숫자를 리턴하고 4번째는 에러 코드를 전송
  - 컨테이너에서는 확인할 수 없음: `docker ps`
  - 웹서버인데 HEALTHCHECK 기능이 없어서 처음 구동될 때 에러가 없으면 계속 구동
- HEALTHCHECK 기능을 추가해서 이미지를 생성하면 이미지가 HEALTHCHECK를 통과하지 못하면 UNHEALTH 상태가 되도록 할 수 있음
  - 웹 서비스를 만들 때 HEALTHCHECK가 가능한 요청에 대한 응답을 만들어서 HEALTHCHECK 명령에 이용하는 것 권장
  - **public cloud는 디폴트로 welcome 요청을 보내도록 만들어져 있음**
- Dockerfile.v2는 이전 Dockerfile에 `HEALTHCHECK CMD curl --fail http://localhost/health`이 추가되었음
  - 30초마다 헬스체크를 하고 3번 연속 실패하면 unhealth 상태로 만듬
    - 새로운 이미지 빌드
      - `docker image build -t numbers-api:v2 -f ./numbers-api/Dockerfile.v2 .`
    - 컨테이너를 생성
      - `docker run -d -p 8081:80 --name numbersapi1 numbers-api:v2`
    - 컨테이너 확인 `docker ps`
    - 90초 후 컨테이너를 확인 - unhealthy 상태

### 의존성이 존재하는 경우
- 의존성이 존재하는 경우 애플리케이션을 실행하는 명령어를 의존성 명령 다음에 && 와 함께 기재하면 의존성 명령어가 실행되지 않으면 서비스가 실행되지 않음
- 다른 API에서 데이터를 가져오는 구문이 포함된 웹 애플리케이션을 생성
  - 컨테이너 실행
  ```
  docker run -d -p 8082:80 --name numbers-web diamol/ch08-numbers-web
  ```

## Docker Swarm
### 3개의 머신 생성하기
- VM을 중지하고 [파일] 메뉴에서 복제를 선책
  - 이름을 설정
  - 완전한 복제를 선택
  - MAC 주소 정책에서 모든 네트워크 어댑터의 새 MAC 주소 설정
- 2번 복제를 수행
- 첫번째 머신을 실행하고 IP 와 Hostname을 설정
  - ifconfig 명령으로 NIC를 확인: 10.0.2.15 할당된 NIC를 찾습니다.
    - enp0s3
  - 네트워크 기능 중지
    - `sudo systemctl stop NetworkManager`
  - IP할당
    - `sudo ifconfig enp0s3 192.168.56.100`
  - 네트워크 기능 활성화
    - `sudo systemctl start NetworkManager`
  - IP 확인
    - `ifconfig`
  - 호스트이름 변경
    - `sudo hostnamectl set-hostname swarm-manager`
    - `cat /etc/hostname`
    - `sudo reboot`
- 두번째 머신을 실행하고 IP 와 Hostname을 설정
  - netplan을 이용해서 IP를 수정(기본적으로 dhcp가 설정되어 있음) `sudo nano /etc/netplan/00-installer-config.yaml`
```yaml
network:
ethernets:
 enp0s3:
addresses: [192.168.56.101/24]
routes:
    - to: default
      via: 192.168.56.1
nameservers:
 address: [8.8.8.8]
version: 2
```
  - 변경 내용 적용: `sudo netplan apply`
  - hostname 변경: `sudo hostnamectl set-hostname swarm-worker1`
  - 재부팅: `sudo reboot now`

- 세번째 머신을 실행하고 IP 와 Hostname을 설정
  - netplan을 이용해서 IP를 수정(기본적으로 dhcp가 설정되어 있음) `sudo nano /etc/netplan/00-installer-config.yaml`
```yaml
network:
ethernets:
 enp0s3:
addresses: [192.168.56.102/24]
routes:
    - to: default
      via: 192.168.56.1
nameservers:
 address: [8.8.8.8]
version: 2
```
  - 변경 내용 적용: `sudo netplan apply`
  - hostname 변경: `sudo hostnamectl set-hostname swarm-worker2`
  - 재부팅: `sudo reboot now`