---
title: Image/Container 관련 명령
weight: 2
---
## Image 관련 명령
### 컨테이너 삭제
모든 컨테이너 중지: docker stop $(docker ps -a -q)
모든 컨테이너 삭제: docker rm $(docker ps -a -q)

- 이미지 삭제
`docker rmi $(docker images -q)`

- 이미지 확인
`docker images`

### 2개의 이미지 다운로드
- debian 과 https 의 최신 버전을 다운로드
`docker pull debian`

`docker pull httpd:latest`

- 다운로드 받으면서 레이어의 개수를 확인(pull complete 이라는 단어가 들어가는 라인)

### 이미지가 다운로드 디렉토리 확인
- /var/lib/docker/image/overlay2/layerdb/sha256
- 이미지 저장소 확인(관리자 모드에서만 가능)
```
#sudo su -

#/var/lib/docker/image/overlay2/layerdb/sha256

#ls
```
- 이 디렉토리에 다운로드 받은 이미지의 레이어들이 저장됩니다.(이렇게 이미지 단위가 레이어 단위로 분할해서 사용하면 재사용성이 높아집니다.)
이 이미지들은 컨테이너가 만들어질 때 기반 이미지가 되며 동일한 레이어는 다시 다운로드 받지 않습니다.

- httpd 의 레이어 확인
```
#docker image inspect --format="{{ .RootFS.Layers }}" httpd

8d853c8add5d1e7b0aafc4b68a3d9fb8e7a0da27970c2acf831fe63be4a0cd2c
```
- 컨테이너가 만들어졌을 때 컨테이너의 최상위 경로는 레이어의 경로 안의 cached-id 파일의 정보를 확인하면 다른 해시 값을 제공하는데 이 해시 값이 컨테이의 최상위 경로입니다.

- 레이어 안으로 이동
```
# cd 8d853c8add5d1e7b0aafc4b68a3d9fb8e7a0da27970c2acf831fe63be4a0cd2c
```
- 레이어 내부 확인
```
#ls

cache-id  diff  size  tar-split.json.gz
```
- cache-id 확인
```
#cat cache-id
```

## 도커 이미지 태그 설정과 도커 로그인 로그아웃
### 태그 설정
- 도커 태그는 원본 이미지에 참조 이미지 이름을 붙이는 명령 형식
`docker tag 원본이미지[:태그] 참조이미지[:태그]`

- 태그 추가하기
  - 도커 이미지 확인: `docker images`

```
$ docker image tag 5daf6a4bfe74 debian-httpd:1.0
$ docker image tag httpd:latest debian-httpd:2.0
$ docker images

REPOSITORY     TAG       IMAGE ID       CREATED        SIZE
debian         latest    c7f9867d6721   2 weeks ago    117MB
debian-httpd   1.0       5daf6a4bfe74   2 months ago   148MB
debian-httpd   2.0       5daf6a4bfe74   2 months ago   148MB
httpd          latest    5daf6a4bfe74   2 months ago   148MB
```

- 새로운 버전을 최신 버전으로 업로드 하기 전에 이렇게 기존 최신 버전의 이름을 변경해서 저장해두어야 이전 버전 과 최신 버전을 같이 유지할 수 있습니다.

- 도커 허브에 업로드할 이미지는 본인아이디/이미지이름[:태그] 의 형태로 만들어야 합니다.
- 본인 아이디가 추가되어야만 레포지토리에 정확하게 업로드가 됩니다.
```
docker image tag httpd:latest ggnagpae1/httpd:3.0
```

- 아이디와 비밀번호를 이용한 로그인
```
docker login 명령은 브라우저에서 로그인을 시도

docker login -u 이름 을 입력하면 콘솔에서 로그인을 시도
```
- 도커 허브에 이미지 업로드: docker push 이미지이름


- 실습을 할 때 하나의 이미지를 push 해보고 동일한 이미지를 다시 한 번 push 해보고 자신의 이름이 포함되지 않은 이미지도 push

### 로그인 방법
- 아이디 와 비밀번호를 이용해서 로그인
- `docker login -u [아이디]` 를 입력한 후 비밀번호를 입력

- /home/계정/.docker/config.json 파일에 저장된다고 경고를 발생시키는데 파일을 열어보면 암호화 알고리즘이 아닌 base64 인코딩을 통한 암호 키값 저장을 합니다. 이 경우는 base64 디코딩을 하게 되면 암호가 노출이 됩니다.

- 토큰을 이용한 로그인
  - 도커허브(hub.docker.com)에 접속해서 로그인
  - 본인 계정을 클릭하고 [Account setting]를 클릭
  - [security]에서 [personal access token]을 클릭
  - [Generate New Token]을 눌러서 토큰을 발급받고 복사
  - 발급받은 토큰을 파일에 저장(토큰을 발급 받을 때만 내용을 보여주고 이후에는 보여주지 않음)
  - `vi .access_token` 명령을 수행해서 토큰 저장
  - `$cat .access_token | docker login --username dragonhailstone --password-stdin`

## 도커 이미지를 파일로 관리
### 파일로 관리하는 이유
- 도커 허브로부터 이미지를 내려받아 내부망으로 이전하는 경우: 예전에 폐쇄망에서 이용
- 신규 애플리케이션 서비스를 위해서 Dockerfile로 새롭게 생성한 이미지를 저장 및 배포하는 경우
- 컨테이너를 commit하여 생성한 이미지를 저장 및 배포하는 경우

### 명령
- 저장: `docker image save 이미지이름 > 파일명`
- 읽기: `docker image load < 파일명`
- mysql:5.7 다운로드(M1의 경우 다운로드가 안될 수 있는데 이 경우는 --platform linux/amd64 나 arm64를 이미지 이름 앞에 추가해주어야 합니다.)
`docker pull mysql:5.7`
- 다운로드 받은 mysql:5.7을 test-mysql57.tar로 저장 `docker image save mysql:5.7 > test-mysql57.tar`

- 압축된 파일 확인: `tar tvf test-mysql57.tar`

- 이미지를 불러오기 위해서 기존 이미지를 삭제: `docker image rm mysql:5.7`

- 이미지 확인: `docker images`

- 이미지 불러오기: `docker image load < test-mysql57.tar`

- 이미지 확인: `docker images`

- 이름을 변경해서 불러올 때는 docker import 를 사용<br>
- `cat test-mysql57.tar | docker import - mysql57:1.0`

- 용량을 줄이고자 하면 gzip 옵션을 이용<br>
`docker image save mysql:5.7 | gzip > test-mysql57gzip.tar.gz`

- 스크립트 변수 방식을 이용해서 도커의 전체 이미지를 하나로 묶는 것도 가능(docker image ls -q 명령은 이미지의 ID를 전부 출력)<br>
`docker image save -o all_image.tar $(docker image ls -q)`

## 이미지 삭제
### 명령어
```
docker image rm [옵션] {이미지이름:태그 | 이미지ID}
docker rmi [옵션] {이미지이름:태그 | 이미지ID}
```
- 이미지는 컨테이너가 사용중이면 삭제되지 않음

### 삭제 테스트
- ubuntu:14.04 버전을 다운로드
```
docker pull ubuntu:14.04
```
- 위의 이미지를 삭제
```
docker rm ubuntu:14.04
```
- 이미지 전부 삭제: `docker rmi $(docker images -q)`
- 특정 이미지 이름을 가진 것만 제외하고 삭제: `docker rmi $(docker images | grep -v 이름)`
- 별명을 이용: 상태가 exited인 container를 찾아서 삭제
```
$ alias cexrm='docker rm $(docker ps --filter 'status=exited' -a -q)'
$ source .bashrc
$ alias
```
- 사용 중이 아닌 모든 이미지 제거: `docker image prune -a`
- 사용중이 아닌 48시간 이전 이미지 제거: `docker image prune -a -f --filter 'until=48h'`

## 컨테이너 관련 명령어
### 컨테이너
- 이미지는 읽기 전용의 불변 값으로 만들어 짐
- 도커 엔진은 이미지를 이용해서 컨테이너를 생성할 수 있는데 이 때 읽고 쓰기가 가능한 레이어를 추가해서 컨테이너를 생성
- 컨테이너 관련 명령도 dockerd 데몬이 제공하는 docker CLI API를 통해 제공
- 서비스 실행과 운영과 관련된 명령어를 제공
- 컨테이너는 프로세스
  - 도커 이미지를 기반으로 만들어지는 Snapshot
  - Snapshot은 도커 이미지 레이어(불변의 유니언 파일 시스템 - UFS)를 복제한 것이고 그 위에 읽고 쓰기가 가능한 컨테이너 레이어를 결합하면 컨테이너가 되는데 이 컨테이너는 이것들만 가지고 실행됨
  - 컴퓨터 애플레케이션의 동작은 process를 통해 이루어지는데 컨테이너는 격리된 공간에서 프로세스가 동작하는 기술입니다
  - 리눅스 운영체제를 부팅하면 PID 1번인 init(systemd) 프로세스가 동작하고 이 프로세스는 나머지 모든 프로세스의 부모 프로세스가 됨
  - 현재 호스트에서 실행 중인 프로세스 ID `echo $$`
- 현재 호스트 컴퓨터인 리눅스가 1번 프로세스를 소유하고 있는데 docker 로 수행한 컨테이너도 1번 프로세스를 소유?
- `docker run` 수행 시 PID 네임스페이스 커널 기능을 이용해서 시스템의 1번 프로세스를 공유하고 그 하위로 도커의 컨테이너들을 격리시킴
격리된 프로세스를 루트로 변경하는 chroot 커널 기능을 통해서 독립된 1번 PID를 갖게됨
- 격리된 컨테이너들은 컨테이너 동작 시 필요한 자원해 대한 할당을 받아야 하는데 이 기능은 cgroups를 가지고 수행함
- 도커 컨테이너를 이해하기 위해서는 컨테이너에 제공되는 커널 기술을 알아야 함, chroot와 cgroups의 동작 원리를 이해

### 컨테이너 실행
- 수동으로 컨테이너를 제어
  컨테이너 생성: `docker create 명령`
  컨테이너 시작: `docker start 명령`
  실행 중인 컨테이너 조회: `docker ps`
  모든 컨테이너 조회: `docker ps -a`
```
#컨테이너 생성
$docker create -it --name container-test1 ubuntu:14.04

#실행 중인 컨테이너 조회: 컨테이너가 안 보임
$docker ps

#모든 컨테이너 조회
$docker ps -a

#컨테이너 동작
$docker start container-test1

#컨테이너에 접속 - attach
$docker attach container-test1

#attach로 컨테이너에 접속한 후 컨테이너에서 빠져 나올때는 exit 명령을 사용하는데 이렇게 되면 컨테이너가 종료됨

#컨테이너 삭제
$docker rm container-test1
```

- docker run: `[docker image pull] + docker create + docker start + [command]`
  - 이미지가 없으면 다운로드
  - 컨테이너 생성한 후 실행
  - 이전 명령을 동일하게 수행
- `docker run -it --name container-test1 ubuntu:14.04 bash`
  - it옵션 과 bash를 붙이면 attach 와 동일한 기능을 수행
  - exit를 입력하면 bash 쉘에서 빠져나오는데 이 때 컨테이너도 중지
  - 컨테이너를 만들 때 쉘에 접속을 하면 쉘을 빠져나올 때 컨테이너가 중지
  - 컨테이너 삭제: `docker rm container-test1`
  - 옵션
  ```
  i, interactive: 대화식 모드로 열기
  t: 단말 디바이스 할당
  d, detach: 백그라운드에서 컨테이너를 실행하고 컨테이너ID를 등록
  name: 컨테이너 이름 설정
  rm: 컨테이너 종료 시 컨테이너를 삭제
  restart: 컨테이너 종료 시 적용할 재시작 정책 지정 (no | on-failure | on-failure:횟수 | always)
  env: 환경변수 설정
  v, volume: 호스트 컴퓨터의 경로와 컨테이너 경로의 공유 볼륨 지정
  h: 컨테이너의 호스트이름 지정, 지정하지 않으면 컨테이너ID가 호스트명으로 지정
  p, publish: 호스트 포트와 컨테이너 포트 매핑, 포트포워딩
  P: 컨테이너의 노출된 포트를 호스트의 임의의 포트에 게시
  link: 동일 호스트의 다른 컨테이너와 연결을 설정하는 것인데 연결이 되면 IP가 아닌 컨테이너 이름으로 통신이 가능
  ```
- MySQL 5.7을 이용한 SQL 테스트
  - MySQL 5.7 다운로드 `docker pull mysql:5.7`
  - 이미지 확인 `docker images | grep mysql`
  - 컨테이너를 실행하면서 쉘에 접속(운영체제는 쉘 이름만 입력하면 되지만 운영체제가 아닌 경우는 /bin/쉘이름) `docker run -it mysql:5.7 /bin/bash`
  - 설치 환경 확인 `cat /etc/os-release`
  - 빠져 나오기 `exit`
  - `docker run`은 컨테이너를 생성하면서 시작
  - 중지된 컨테이너를 재시작하는 것은 `docker start {컨테이너ID | 컨테이너이름}`
  - 컨테이너에 명령 수행 `docker exec [옵션] {컨테이너ID | 컨테이너이름} 명령어`
  - 컨테이너를 종료하지 않고 쉡에서 빠져나오기: CTRL + p + q
  - 내부 IP 확인: `docker inspect 컨테이너이름 | grep IPAddress`
  - bash에 접속: `docker exec -it 컨테이너이름 /bin/bash`

- nginx 컨테이너 생성
  - 이미지 다운로드: - `docker pull nginx:1.18`
  - 이미지 확인: `docker images`
  - nginx는 웹서버 애플리케이션이므로 계속 동작해야 함, 이런 경우에는 데몬으로 실행해야 해서 d옵션을 추가해서 실행해야 함
  - nginx는 웹 서버이므로 외부에서 접속을 할 수 있어야 함, 이런 경우는 포트포워딩 해야 함
    - 포트포워딩을 할 때는 p옵션 다음에 호스트 컴퓨터의 포트번호:컨테이너의 포트번호를 설정해 주어야 함
  - 컨테이너를 만들 때 컨테이너 이름을 설정하지 않으면 컨테이너ID를 가지고 작업을 계속 수행해야 함, 컨테이너 이름을 설정하는 것이 좋음
  - nginx를 외부에서 8001번 포트로 접속할 수 있도록 컨테이너를 생성  
    `docker run --name webserver -d -p 8001:80 nginx:1.18`  
    확인 할 때는 docker ps 명령으로 컨테이너가 생성되었는지 확인을 해보고 `curl http://localhost:8001`로 html 내용이 오는지 확인
- 리소스 사용량 확인: `docker stats 컨테이너 이름`
```
docker run --name webserver -d -p 8001:80 nginx:1.18

docker stats webserver - 종료하고자 하는 경우는 ctrl + z

브라우저나 다른 터미널에서 localhost:8001 에 접속하면서 사용량을 확인
```
- 컨테이너 내부의 실행 중인 프로세스 확인: `docker top 컨테이너이름`
- 컨테이너 내부의 로그 확인: `docker logs -f(f는 실시간이고 t를 사용하면 마지막 로그까지) 컨테이너이름`
- 호스트 컴퓨터에서 컨테이너로 파일 복사: `docker cp 호스트컴퓨터의파일경로 컨테이너이름:컨테이너내부파일경로`
  - nginx 의 첫 화면 파일을 확인
  ```
  docker exec -it webserver /bin/bash

  #cat /usr/share/nginx/html/index.html
  ```
  - 컨테이너 외부에서 index.html 파일을 작성
  ```
  $ vi index.html
  <h1>File Copy</h1>
  ```
  - 컨테이너 외부에서 컨테이너 내부로 파일을 복사
  ```
  docker cp index.html webserver:/usr/share/nginx/html/index.html
  ```
  - 웹 서버에 접속해보면 메인 화면이 변경된 것을 확인할 수 있음
  - 설정 파일을 컨테이너의 쉘에서 직접 수정할 수 있는 경우도 있지만 그렇지 못하는 경우가 많기 때문에 외부에 설정 파일을 만든 후 docker cp 명령을 이용해서 복사해서 사용
- 컨테이너 일시 중지: `docker pause 컨테이너이름`
- pause된 컨테이너 다시 시작: `docker unpause 컨테이너이름`
- 컨테이너 다시 시작: `docker restart 컨테이너이름`

### 파이썬 파일을 도커 컨테이너로 실행할 수 있도록 생성
- 파이썬의 기본 이미지는 python
- 현재 위치에 파이썬 파일을 1개 작성
- 리눅스에서 작업한느 것이 불편하면 윈도우에서 작업한 후 git 명령 등을 이용해서 복사해도 됨
- 파이썬 파일 작성(py_lotto.py)
```python
from random import shuffle
from time import sleep

gamenum = input('How many times?')

for i in range(int(gamenum)):
        balls = [x+1 for x in range(45)]
        ret = []
        for j in range(6):
                shuffle(balls)
                number = balls.pop()
                ret.append(number)
        ret.sort()
        print('lotto number[%d]:' %(i+1), end=' ')
        print(ret)
        sleep(1)
```
- 파일을 실행해서 에러가 없는 것을 확인 `python3 py_lotto.py`
- python 컨테이너 실행 `docker run --name=python_test -dit -p 8900:8900 python`
- 소스 파일을 컨테이너로 복사 `docker cp py_lotto.py python_test:/`
- 도커 외부에서 실행 `docker exec -it python_test python /py_lotto.py`

### nodejs 애플리케이션을 도커 컨테이너로 실행
- nodejs의 이미지 이름은 node
- nodejs에서 web 서버를 만드는 경우 포트는 직접 지정해야 함
- nodejs로 웹 서버를 만드는 코드를 작성: nodejs_test.js
```js
let http =  require('http')
let content = function(req, res){
        res.end('Go for it Korea~!!!')
        res.writeHead(200)
}
let web =  http.createServer(content)
web.listen(8002)
```
- 8002 번 포트를 외부에서 접속할 수 있도록 node 컨테이너를 생성 `docker run -dit --name=nodejs_test -p 8002:8002 node`

### 컨테이너를 이미지로 만들기
- `docker commit 컨테이너이름 이미지이름`
- `-a` 옵션을 이용해서 작성자를 추가해주기도 함
- 도커 허브에 있는 이미지를 다운로드 받아서 소스코드를 추가하고 만든 이미지를 골든이미지라고 함
- nginx에 welcome file을 변경한 이미지를 만들어서 업로드하고 다운로드해서 테스트
  - welcome file을 생성(index.html)
  - nginx의 최신 버전으로 컨테이너 실행 - 용도는 웹서버, 기본 포트번호는 80
    - `docker run --name=website -dit -p 8008:80 nginx`
  - 로컬의 파일을 컨테이너로 복사
    - `docker cp index.html website:/usr/share/nginx/html/index.html`
  - 확인 
    - `curl http://localhost:8008`
  - 컨테이너를 이미지로 생성 
    - `docker commit -a adam website website:1.0`
  - 이미지를 업로드 하기 위해서 태그 작업 수행
    - `docker tag website:1.0 dragonhailstone/website:1.0`
  - 이미지 업로드
    - `docker push dragonhailstone/website:1.0`
