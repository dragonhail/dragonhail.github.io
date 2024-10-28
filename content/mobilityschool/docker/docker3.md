---
title: Docker Volume
weight: 3
---
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
```
- 데이터의 지속성 유지를 위해서 볼륨 생성
  docker volume create mysql-data-vol

- 볼륨과 연결해서 mysql:5.7 이미지를 컨테이너로 생성
  docker run -it -d --name=mysql-server -e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=dockertest -v mysql-data-vol:/var/lib/mysql mysql:5.7

- 데이터 수정
  쉘에 접속: docker exec -it mysql-server /bin/bash

데이터베이스 사용을 위해서 mysql에 접속: mysql -uroot -p

데이터베이스 확인: show databases;

데이터베이스 사용 설정: use dockertest;

테이블 생성: create table mytab(c1 int, c2 char);

데이터 추가: insert into mytab values(1, ‘a’);

데이터 확인: select * from mytab;

- 컨테이너를 삭제
  docker stop mysql-server

  docker rm mysql-server

- 이전에 만든 볼류 과 연결해서 컨테이너를 재생성해서 이전 데이터가 유지되는지 확인
  docker run -it -d --name=mysql-server -e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=dockertest -v mysql-data-vol:/var/lib/mysql mysql:5.7
```
- nginx 이미지의 컨테이너는 /var/log/nginx 디렉터리를 볼륨에 연결
  - 볼륨으로 사용할 디렉터리를 생성하면서 연결<br>`docker run -d -p 8011:80 -v /home/dh/nginx-log:/var/log/nginx nginx:1.19`

### linux의 awk
- 파일로부터 레코드를 선택하고 선택된 레코드에 포함된 값을 조작하거나 데이터화하는 것을 목적으로 사용하는 프로그램
- 텍스트 파일의 전체 내용을 출력할 수 있고 특정 필드만 출력하는 것이 가능
- 아파치 웹 서비스의 access.log는 하나의 레코드처럼 기록
- 줄단위로 전체 출력
  - `awk {print} nginx-log/access.log`
- 특정 필드만 출력
  - `awk '{print $1, $4}' nginx-log/access.log`
- 연산
  - `awk '{sum += $10} END {print sum}' nginx-log/access.log`

### 컨테이너 간 데이터 공유를 위한 데이터 컨테이너 생성
- 다른 컨테이너와 볼륨을 공유할 수 있는데 이 경우에는 `--volumes-from 컨테이너이름`을 이용하면 됨
- 컨테이너 간 데이터 공유를 위한 컨테이너
  - 데이터 공유를 위한 도커 컨테이너 생성: 실행하지 않음<br>`docker create -v /data-volume --name=datavol ubuntu:18.04`
  - 위의 컨테이너 와 데이터를 공유하는 컨테이너 생성<br>`docker run -it --volume-from datavol ubuntu:18.04`
  - volume 위치: /var/lib/docker/volumes

### 볼륨을 유용하게 사용
- httpd(apache 웹 서버) 의 경우 welcome 파일의 위치는 /usr/local/apache2/htdocs 디렉토리에 위치하고 있음
- docker cp를 이용한 메인 화면 수정
  - 컨테이너 실행<br>`docker run --name apache1 -d -p 8101:80 httpd`
  - 컨테이너 내부의 파일을 호스트 컴퓨터로 복사<br> `docker cp apache1:/usr/local/apache2/htdocs/index.html index.html`
  - index.html 파일을 호스트 컴퓨터에서 수정(수정할 때 마다 작업을 수행)<br>`docker cp index.html apache1:/usr/local/apache2/htdocs/index.html`
- 바인드 마운트를 이용한 welcome 파일 수정
  - 컨테이너 생성
  `docker run --name apache2 -v /home/dh/apache:/usr/local/apache2 htdocs -d -p 8102:80 httpd`
  - `curl http://localhost:8102`로 확인
  - apache 디렉터리에 index.html 파일을 생성
  `sudo nano index.html`

### 자원 사용에 대한 제약
- 볼륨에 대한 자원 사용 확인
  - 리눅스에서 자원 사용량을 확인: `df - h`
  - 컨테이너 내부에서도 동일한 명령을 수행
  `docker exec -it apache2 /bin/bash`
  - 현재는 컨테이너의 자원 사용 가능 사이즈와 호스트 컴퓨터의 자원 사용 가능 사이즈가 동일
- 리눅스 자원 모니터링
  - `top`: 리눅스 전체의 자원 소비량 및 개별 액티브 프로세스의 자원 사용량
  - `htop`: top 보다 향상된 자원 사용량 제공, 별도로 설치해야 함
  - `sar`(system active report): 다양한 옵션을 이용해서 시스템 전반의 사용량에 대한 세부적인 모니터링을 제공하는데 쉘 스크립트에 포함해서 사용하는데 sysstat 를 별도로 설치해야 함<br>`sar 2 10`(2초 마다 10번 수집)
  - `iostat`, `df`: 디스크 성능 지표를 수집, iostat는 sar 사용법이 동일
  - `vmstat`, `free`: 메모리 성능 측정<br>vmstat는 sar 사용법이 동일하고 `free -mt` 명령으로 마지막 사용량을 MB 단위로 표시
  - `dstat`: 시스템 전반의 자원 사용량에 대한 모니터링 제공, `dstat`를 별도 설치
  - `iptraf-ng`: 유입되는 네트워크 인터페이스 별 패킷량, 프로토콜 등을 통해서 네트워크 트래픽 모니터링, , iptraf-ng를 별도로 설치
- 위의 도구들을 이용해서 서버 자원을 모니터링을 해서 예방적 차원의 관리 작업 과 효율성을 확인할 수 있음
- 컨테이너를 사용하는 docker run 이나 docker create 명령과 함께 자원 할당 제어를 사용하지 않는다면 생성되는 컨테이는 호스트 운영체제의 모든 자원을 자유롭게 사용하고 과도한 자원 사용도 가능
- 클라우드 네이티브 환경에서는 컨테이너를 사용하는 목적이 소규모의 애플리케이션 서비스인데 이 작은 애플리케이션 서비스가 잘못된 설정으로 시스템에 부하를 유발한다면 다른 컨테이너의 동작 간섭 뿐 아니라 호스트 운영체제 전반이 영향을 받게 됨
- 도커에서는 여러 런타임 제약 옵션을 제공하며 컨테이너 생성 후에도 docker update 명령을 이용해서 변경이 가능
- 리소스 런타임 제약은 리눅스 커널이 제공하는 cgroup 기능을 통해 가능
```
Docker에서 중요한 3가지
namespace
chroot: 컨테이너가 1번 PID
cgroup: 자원 분배
```
- cgroup 확인
  - `grep crgoup /proc/mounts`
  - `docker info | grep Cgroup`

- nginx이미지에 메모리 1기가 할당한 컨테이너 생성
  - 할당
    - `docker run --name=nginx_mem_1g --memory=1g nginx`
  - 확인
    - `docker inspect nginx_mem_1g | grep \"Memory\"`

- CPU 제약
```
--cpus: 1개의 cpu 인 경우는 비율이 되고 여러 개의 cpu 인 경우는 개수
--cpus=0.2: 1개인 경우는 20%를 사용
--cpus=1.5: 여러 개 인 경우 1.5개를 사용
     
--cpu-period: 기간 제한 옵션으로 컨테이너의 CFS는 밀리세컨드 단위로 지정
 	--cpu-period=100000 의 형태로 지정
--cpu-quota: 시간 할당량
	--cpu-quota=50000 의 형태로 지정하는데 --cpu-period 와 같이 사용

--cpuset-cpus: 여러 개의 cpu 인 경우 특정 cpu의 코어 번호를 이용해서 할당
	--cpuset-cpus=”0,3”: 0번 과 3번 코어 이용
--cpuset-cpus=”0-2”: 0, 1, 2번 코어 사용

--cpu-shares: 공평한 스케쥴링을 원칙으로 하는데 가중치를 부여해서 사용하는 것이 가능
```
- cpu 제약 실습
```
docker run -d --name cpu_1024 --cpu-shares 1024 leecloudo/stress:1.0 stress --cpu 4

docker run -d --name cpu_512 --cpu-shares 512 leecloudo/stress:1.0 stress --cpu 4

ps -aux | grep stress | grep -v grep
```
  - 이런 테스트를 할 때는 무한 루프로 동작하는 애플리케이션을 만들어서 확인합니다.
  - 2개 이상 실행시키게 되면 일련번호 형태로 프로세스 ID가 만들어진 것이 하나의 컨테이너가 실행한 것입니다.
- 메모리 사용량 제한
  - 메모리
    - 물리적인 메모리와 스왑 메모리(가상 메모리)로 나누는데 물리적인 메모리는 실제 메모리 크기를 의미하고 스왑 메모리는 메모리가 부족할 때 하드디스크의 일정 부분을 빌려와서 사용하는 메모리
    - 보통의 경우 스왑 메모리를 물리적 메모리의 두배로 설정하는데 스왑 메모리를 사용하게 되면 속도는 느려지게 됨
  - 옵션
  ```
  memory 또는 m: 물리적인 메모리의 최대값이고 허용되는 최소값은 4m
  
  memory-swap: 스왑할 수 있는 메모리 양을 지정하는데 0을 주면 컨테이너 스왑 사용을 해제하는 것이고 -1이면 무제한
  
  kernel-memory: 커널 메모리 설정, 모든 cgroup이 필요한 메모리 크기가 장비의 메모리보다 클 때 사용
  ```
  - ubuntu:14.04 이미지를 이용하여 컨테이너를 생성하는데 메모리의 크기를 1기가로 제한
    - `docker run -it -d --name=ubuntu_1g --memory=1g ubuntu:14.04`
  - 애플리케이션이 사용하는 메모리 양보다 적게 설정되는 경우 오류가 발생: kafka를 사용할 때 이 오류가 많이 발생
  - 