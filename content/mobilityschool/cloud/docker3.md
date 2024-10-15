## Docker 기본적인 명령어 사용
### 컨테이너와 이미지 모두 삭제
- 모든 컨테이너 중지: `docker stop $(docker ps -a -q)`
- 모든 컨테이너 삭제: `docker rm $(docker ps -a -q)`
- 모든 이미지 삭제: `docker rmi $(docker images -q)`

### httpd(apache web server) 이미지를 컨테이너로 생성
- 기본 정보
  - 이미지 이름:httpd
  - 포트번호: 80
- 명령어
  - `docker run --name apachewebserver -d -p 8001:80 httpd`

- 확인
  - 구동 중인 컨테이너 확인: `docker ps`
  - 웹 서버의 경우는 브라우저에서 localhost:8001 로 확인
  - 명령어로는 `curl localhost:8001`

### e 옵션
- 환경변수를 설정하는 옵션
- MySQL 5.7(Mac M1의 경우 --platform linux/amd64 를 추가) 버전 같은 경우는 관리자 비밀번호 와 초기 데이터베이스 생성 그리고 유저 와 유저 비밀번호 등을 설정할 수 있는 환경변수를 제공
- MySQL 5.7 의 환경변수
```
MYSQL_ROOT_PASSWORD: 관리자 비밀번호
MYSQL_DATABASE: 생성할 데이터베이스
MYSQL_USER: 생성할 유저
MYSQL_PASSWORD: 유저의 비밀번호
```
- MySQL5.7 버전의 컨테이너를 생성 `docker run --name mysql -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=wnddkd -e MYSQL_DATABASE=itstudy mysql:5.7`

### 호스트 컴퓨터 와 컨테이너 사이의 파일 복사
- 명령어
```
docker cp source경로 destination경로
컨테이너를 설정할 때 컨테이너이름:파일경로
파일 경로를 작성할 때 파일 이름을 생략하면 파일 이름은 동일하게 복사
```
- 설정 파일을 수정해서 사용해야 하는 경우 쉘에 접속해서 직접 수정할려고 하는 경우 번거롭기도 하고 vi 와 같은 텍스트 에디터가 존재하지 않는 경우도 있어서 외부에서 만든 후 복사하는 것이 편리

- httpd 이미지의 컨테이너에서 보여지는 welcome 파일의 경로는 /usr/local/apache2/htdocs/index.html

- 컨테이너에서 호스트 컴퓨터로 파일을 가져오기
`docker cp apachewebserver:/usr/local/apache2/htdocs/index.html ./index.html`

- 호스트 컴퓨터의 index.html 을 수정해서 컨테이너로 복사
`docker cp ./index.html apachewebserver:/usr/local/apache2/htdocs/index.html`

### 컨테이너 모니터링 도구
- 서비스 운영을 하면서 필요한 시스템 Metric(CPU/메모리 사용량, 네트워크 트래픽 등)을 모니터링 하면서 특이 사항이 있을 때 대응을 해야 함
- 컨테이너 환경에서는 기존의 모니터링 도구를 사용하는 것이 어려움
- 컨테이너를 모니터링하기 위해서 구글에서는 cAdvisor라는 도구를 제공
- 설치
```
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker:/var/lib/docker:ro -p 8080:8080 --detach=true --name=cadvisor gcr.io/cadvisor/cadvisor:v0.39.3
```
- 모니터링: 브라우저에서 localhost:8080

## Docker Volume
### 도커 볼륨이 필요한 이유
- 컨테이너는 프로세스
- 컨테이너는 기본적으로 메모리에 데이터를 저장하고 사용하며 파일에 저장하는 것도 가능합니다.
- 컨테이너는 삭제되면 소유하고 있던 모든 것들을 삭제합니다.
- 데이터의 영속성을 유지할 수가 없음
- 데이터의 영속성 때문에 볼륨을 사용

### 타입
- bind mount: 호스트 컴퓨터의 파일 시스템과 직접 연결하는 방식
- volume: 도커 엔진의 파일 시스템과 연결하는 방식
- tmpfs mount: 메모리에 일시적으로 저장하는 방식
- 컨테이너 간에도 설정 가능

### 생성
- `docker volume create 이름`
```
생성
docker volume create my-appvol-1

조회
docker volume ls

상세보기
docker volume inspect myappvol-1
```
### 볼륨 설정
```
=>--mount 옵션을 이용해서 source 와 target을 설정
docker run --mount source=볼륨이름, target=연결할디렉토리 이미지이름

docker run -d --name vol-test1 --mount source=my-appvol-1,target=/app ubuntu:20.04

=>-v 옵션을 이용해서 직접 매핑
docker run -d --name vol-test2 -v my-appvol-1:/var/log ubuntu:20.04

=>없는 볼륨이름을 이용하면 자동 생성
docker run -d --name vol-test3 -v my-appvol-2:/var/log ubuntu:20.04
```

### 볼륨 삭제
- 볼륨 삭제
  - `docker volume rm 볼륨이름`
- 컨테이너에 연결되어 있으면 삭제 안됨
- docker volume rm my-appvol-1 명령을 수행하면 연결된 컨테이너가 있다가 에러 메시지가 출력됨
- 현재 만들어진 2개의 볼륨 삭제
  - 컨테이너를 삭제
  ```
  docker stop vol-test1 vol-test2 vol-test3
  docker rm vol-test1 vol-test2 vol-test3
  ```
  - 볼륨을 삭제
    - `docker volume rm my-appvol-1 my-appvol-2`

### bind mount
- 호스트 컴퓨터의 디렉터리와 컨테이너의 디렉터리를 직접 매핑
- 방법
  - `--mount` 이용
    - `--mount type=bind,source=호스트컴퓨터디렉터리,target=컨테이너디렉터리`
  - `-v 이용`
    - `-v 호스트컴퓨터디렉토리:컨테이너디렉토리`
- 사용자가 파일 또는 디렉토리를 생성하면 해당 호스트 파일 시스템의 소유자 권한으로 만들어지고 존재하지 않는 경우 자동 생성되는데 이 때 생성되는 디렉토리를 루트 사용자 소유가 됨
- 컨테이너 실행 시 지정하여 사용하고 컨테이너 제거 시 바인드 마운트는 해제되지만 호스트 디렉토리는 유지가 됩니다.

- 실습
```
- 실습을 위한 디렉토리를 생성
mkdir /home/adam/target

- mount 옵션으로 centos:8 의 /var/log 디렉토리 와 연결
docker run -dit --name bind-test1 --mount type=bind,source=/home/dh/target,target=/var/log centos:8

- v 옵션으로 centos:8 의 /var/log 디렉토리 와 연결
docker run -dit --name bind-test2 -v /home/dh/target:/var/log centos:8

- 없는 디렉터리와 연결
docker run -dit --name bind-test3 -v /home/dh/target1:/var/log centos:8

- 없는 디렉토리에 권한을 부여해서 연결
docker run -dit --name bind-test4 -v /home/adam/target_ro:/app1:ro -v /home/dh/target_rw:/app2:rw centos:8
```

### tmpfs mount
- 임시적으로 연결
- 컨테이너가 중지되면 마운트가 제거되고 내부에 기록된 파일도 유지되지 않음
- 중요한 파일을 임시로 저장하기 위해서 사용
- 컨테이너 실행 시 지정하여 사용하고 컨테이너 해제 시 자동 해제
- 실습
```
- mount 옵션을 이용해서 tmpfs 연결을 수행하는 httpd:2의 /var/www/html 파일을 임시로 연결

docker run -dit --name tmpfs-test1 --mount type=tmpfs,destination=/var/www/html httpd:2

docker run -dit --name tmpfs-test2 --tmpfs /var/www/html httpd:2
```

### 활용
- 데이터베이스의 데이터 지속성 유지
  - 데이터의 지속성 유지를 위해서 볼륨 생성<br>
    `docker volume create mysql-data-vol`
  - 볼륨과 연결해서 mysql:5.7 이미지를 컨테이너로 생성<br>
    `docker run -itd --name=mysql-server -e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=dockertest -v mysql-data-vol:/var/lib/mysql mysql:5.7`
  - 데이터 수정<br>
    쉘에 접속: `docker exec -it mysql-server /bin/bash` <br>
    데이터베이스 사용을 위해서 mysql에 접속: `mysql -uroot -p`
  - 컨테이너를 삭제
  - 이전에 만든 볼류 과 연결해서 컨테이너를 재생성해서 이전 데이터가 유지되는지 확인