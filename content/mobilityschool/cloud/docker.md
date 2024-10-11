---
title: Docker
weight: 6
---
## Docker
### Container & 가상화
- 가상화
  - 서버, 스토리지, 네트워크 및 물리적 시스템에 대한 가상 표현을 생성하는데 사용할 수 있는 기술
  - 가상 소프트웨어는 물리적 하드웨어 기능을 모방하여 하나의 물리적인 컴퓨터에서 여러 가상 시스템을 동시에 실행
  - 가상화를 사용해서 하드웨어 리소스를 효율적으로 사용할 수 있어서 투자 대비 이익을 더 많이 얻을 수 있고 클라우드 컴퓨팅 서비스를 지원해서 조직의 인프라를 더욱 효율적으로 관리할 수 있음
- 가상화 이점
  - 효율적인 리소스 사용
  - 자동화된 IT 자원 관리
  - 신속한 재해 복구(가상화가 아니더라도 최근에는 운영체제나 소프트웨어가 이 기능을 지원)
- 가상화 서비스
  - 서버 가상화
  - 스토리지 가상화(하나의 스토리지를 나누어 쓰기도 하고 여러 개의 스토리지를 하나처럼 가상화하기도 함)
  - 네트워크 가상화
    - 물리적 환경의 데이터 라우팅에서 라우팅 관리를 인수해서 트래픽 라우팅을 제어(SDN - Software Defined Network)
    - 애플리케이션 트래픽보다 영상 통화 트래픽을 우선적으로 처리하도록 시스템을 프로그래밍해서 사용
  - 데이터 가상화
  - 애플리케이션 가상화
  - 데스크탑 가상화(DaaS)

### 가상화 방식
- 호스트 운영체제 가상화
```
게스트 OS
가상화 소프트웨어
호스트 운영체제
하드웨어
```
- 물리적 하드웨어 위에 설치된 Host OS 위에 가상화 소프트웨어와 가상 머신에 설치된 OS를 움직이는 방식
- 가상의 하드웨어를 Emulating 하기 때문에 호스트 운영체제에 크게 제약 사항이 없음
- OS위에 OS가 얹히는 방식이기 때문에 오버헤드가 클 수 있음
- 대표적인 가상화 소프트웨어
  - VMWare Workstation
  - Virtual Box
  - Virtual PC
  - UTM
- 하이퍼바이저 가상화
  - Host OS 없이 하드웨어에 하이퍼바이저를 설치해서 사용하는 가상화 방식
  - 하나의 컴퓨터에서 여러 가상 머신을 관리하는 소프트웨어 구성요소로 각 가상머신이 할당된 리소스를 얻고 다른 가상 머신의 작동을 방해하지 않도록 하는 방식
  - 호스트 OS와 별도로 개별 시스템처럼 동작하기 때문에 오버헤드가 존재하지 않음
  - 호스트 OS가 없기 때문에 관리를 위한 컴퓨터나 콘솔이 필요
  - 서버 가상화의 대부분은 이 방법을 사용
  - 하드웨어와 운영체제 사이에서 물리적 시스템의 하드웨어에 직접 설치되는 경우를 베어메탈 하이퍼바이저라고 함
  - 구현 방식
    - 전 가상화: 하드웨어 전체를 완전히 가상화하는 방식으로 VMWare의 ESX Server나 MS의 Hyper V가 대표적인 전가상화 소프트웨어
    ```
    HOST OS 게스트OS 게스트OS
                    |
    하이퍼바이저->하드웨어 제어를 위한 컨트롤러
        |
    하드웨어
    ```
    - 반 가상화: 하드웨어 제어를 위한 컨트롤러 대신에 운영체제와 직접 대화를 할 수 있는 인터페이스를 제공하는 방식으로 전가상화보다 성능이 우수
    - Xen, KVM이 대표적인 반가상화 소프트웨어
    ```
    HOST OS 게스트OS 게스트OS
        |
    하이퍼바이저
        |
    하드웨어
    ```
- Container 가상화
  - 호스트 운영체제 위에 Container 관리 소프트웨어를 설치하고 논리적으로 Container를 나누어 사용
  - 애플리케이션 동작을 위한 라이브러리와 애플리케이션으로 구성된 Container를 이용하는데 각각 별개의 서버처럼 사용이 가능
  - 장점: 오버헤드가 적고 가볍고 빠르다는 장점이 있음
  - 단점: 다양한 OS를 사용할 수 없고(리눅스만 가능) 보안적으로 완전히 격리되지는 않음
  - 컨테이너 가상화 소프트웨어
    - OpenVZ
    - LXC
    - Linux VServer
    - Docker
    - Oracle Solaris Zones
  - 계층
    ```
    미들웨어
    컨테이너 관리 소프트웨어(가상화SW->OS 구조에 비해 단순)
    OS
    하드웨어
    ```
### 애플리케이션 배포 방식의 변하
- 전통적인 배포는 운영체제 위에 애플리케이션을 바로 설치하는 방식으로 배포
- 하이퍼바이저를 이용하는 방식으로 바뀌었다가 최근에는 컨테이너 방식으로 변경

### Container
- 각 애플리케이션에 운영체제가 아닌 의존성 요소만 포함시킨 것
- 애플리케이션 인터페이스는 호스트 운영체제가 직접 연결되며 게스트 운영체제 같은 추가 레이어가 없기 때문에 성능은 향상되고 리소스도 낭비되지 않으며 이미지 파일 크기도 작음
- 컨테이너는 호스트 운영체제의 프로세스 수준에서 격리가 되며 컨테이너간에는 기본적으로 의존성 요소를 공유하지 않음
- 컨테이너 방식은 Linux 네임스페이스와 컨트롤 그룹을 생성해서 모든 처리를 수행하는데 이 때문에 컨테이너의 보안이 Linux 커널 프로세스 격리를 기반으로 하게 되는데 이는 충분히 검증되었지만 가상 머신이 제공하는 전체 OS 기반 격리보다는 덜 안전하다고 알려져 있음

## Docker
- 컨테이너형 가상화 기술을 구현하기 위한 상주 애플리케이션과 이 애플리케이션을 조작하기 위한 명령형 도구로 구성되는 애플리케이션
- 같이 사용되는 프로그램과 데이터를 격리시키는 기능을 제공 - Container 가상화
- Docker를 사용하고자 하는 경우는 Docker 소프트웨어 본체인 Docker 엔진을 설치해야만 Container를 생성하고 구동시킬 수 있음
- Linux Container 구현체의 사실상(de-facto) 표준

### Docker는 Linux를 필요로 함
- Windows나 Mac OS에서도 Docker를 구동시킬 수는 있지만 이 경우 내부적으로 Linux가 사용되면 Container에서 동작시킬 프로그램도 Linux용 프로그램
- Docker가 Linux 운영체제에서 사용하는 것을 전제로 만들어 짐

### LXC(Linux Container)
- 운영체제 수준의 가상화 구현: 하나의 Linux 커널을 사용하는 제어 호스트에서 여러 개의 격리된 Linux 시스템(Container) 실행
- Linux 커널에서 가상화를 위해 사용하는 기능
  - Namespace: 프로세스를 독립시켜 주는 가사오하 기술 - Linux 커널 리소스의 분리
    - 운영환경에 대한 애플리케이션의 완전한 분리를 허용 프로세스 트리, 네트워킹, 사용자ID 및 파일 시스템 포함
  - cgroups: 자원을 제한하고 격리시키는 Linux 커널 기능
  - chroot(change root): 특정 디렉터리를 최상위 디렉터리인 root로 인식하게끔 설정하는 리눅스 명령

### 동작 원리
- Container 안에는 운영체제는 아니지만 운영체제와 유사한 것이 들어 있음
- Windows나 Mac에서는 Linux 운영체제를 끌어들여서 사용

### Image와 Container
- Image: Container를 만들어내는 설계도 역할을 수행하는 것으로 하나만 있으면 동일한 Container를 여러 개 생성할 수 있음, 프로그램이나 클래스의 역할
- Container: 실젤 사용이 되는 애플리케이션의 개념으로 인스턴스나 프로세스의 역할인데 이를 기반으로 이미지를 만들 수 있음
- Image는 주로 Docker Hub에서 가져오는데 Docker Hub는 공식적으로 운영되는 Docker Registry로 https://hub.docker.com
- 이미지의 종류는 운영체제 역할을 수행하는 것 그리고 소프트웨어 1개가 포함된 것 또는 여러 개 가 포함된 것으로 나눌 수 있음

### 추천하지 않는 경우
- Container는 운영체제의 동작을 완전히 구현하지는 못하기 때문에 좀 더 엄밀한 Linux 계열의 운영체제 동작이 요구되는 경우는 가상화 소프트웨어 사용을 권장

### 주요 구성 요소
- Docker Engine: Docker를 이용한 애플리케이션 실행 환경을 제공
- Docker Client: Docker Engine에 명령을 내릴 수 있는 CLI 도구
- Docker Host: Docker Engine을 설치한 컴퓨터
- Docker Hub: Docker Image를 공유하는 클라우드 서비스
- Docker Compose: 의존성있는 독립된 Container에 대한 구성 정보를 yaml코드로 작성하여 일원화된 애플리케이션 관리를 가능하게 하는 도구
- Docker Swarm: 여러 Docker 호스트를 클러스터로 구축해서 관리할 수 있는 도구
- Docker Registry: 이미지를 저장하고 내려받을 수 있는 레지스트리 구축을 위한 도구

### 설치
- Windows나 Mac에서는 도커 허브 사이트에 접속해서 Desktop을 다운로드 받아서 설치
- Ubuntu Linux에 설치
  - Docker에서 제공하는 공식 GPG(GNU Privacy Guard) key를 추가
```bash
sudo apt-get update

sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo apt-key fingerpring

Docker를 다운로드 받기 위한 repository 추가
echo \  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \ $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

최신 패키지로 업데이트: sudo apt update
설치된 저장소 확인: apt-cache policy docker-ce
도커 관련 패키지 설치: sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
설치 확인: docker version
서비스 시작: sudo service docker start
데몬 확인: sudo systemctl status docker

Docker 그룹에 현재 사용자 등록
sudo usermod -aG docker $(whoami)
sudo chmod 666 /var/run/docker.sock
sudo service docker restart
```
### 기본 명령어 사용
- image 다운로드: `docker pull hello-world`

- image 확인: `docker images`

- Container로 생성: `docker run hello-world`

- Container 확인: `docker ps -a`

### Docker 정보 확인
- `docker version`
- docker 구성 정보 확인: `docker info`
  - 커널 정보, 현재 컨테이너의 수, 이미지 수 등을 확인
  - 사용한 공간의 크기등도 조회가 가능
- 디스크 용량 확인: `docker system df`
- Docker 관련 이벤트 정보 확인: `docker system events`
  - 롤링(계속 쌓는 것이 아니고 일정 주기로 백업 또는 파일로 덮어쓰면서) 로그이고 한 번에 최대 1000개의 이벤트를 보유
  - 확인할 때는 2개의 터미널을 열어서 작업
  - 하나의 터미널에는 docker system events 명령을 수행한 후 다른 터미널에서 `docker run -itd -p 80:80 --name=webapp nginx` 명령과 `docker stop webapp` 명령을 수행
  - 필터 옵션
    -  이미지 관련 명령만 확인:`docker system events –-filter 'type=image'`
    - `docker run -itd -p 80:80 –name=webapp1 nginx`(이 명령은 이미지를 다운로드 받지 않기 때문에 로그가 기록되지 않음)
    - 상태 관련 로그만 확인: `docker system events --filter 'event=sotp'`
    - 특정 컨테이너 로그만 확인: `docker system events --filter 'container=컨테이너이름'`
    - webapp이라는 컨테이너의 중지 상태 로그만 확인: `docker system events --filter 'container=webapp' --filter 'event=stop'`
    - 지난 24시간 동안의 로그만 확인: `docker system events –-since 24h`
    - 로그를 JSON으로 출력: `docker system events --format '{{json .}}'`
    - 문자열을 JSON(데이터 전달을 위한 포맷) 형식으로 만든다는 것은 대부분 다른 곳으로 전송을 하거나 파일로 출력하기 위한 목적
- 컨테이너를 실행시키면서 터미널로 접속
  - `docker container run --interactive --tty diamol/base`
  - `docker container run -it diamol/base`
  - 터미널에서 명령 수행
```
#hostname

#date
```
- 현재 실행 중인 컨테이너 확인 
  - `docker container ls 였는데 지금은 docker ps`
- 프로세스 확인 
  - `docker container top 컨테이너ID` 또는 `docker top 컨테이너ID`
  - 컨테이너ID는 똑같은 게 없을 땐 앞몇자리만 적어도 됨
- 컨테이너 로그 확인 
  - `docker container logs 컨테이너ID`

## Docker 활용
### DOcker Hub와 Docker Registry
- Docker Registry: Image를 배포하는 장소
- Docker Hub: Docker 제작사에서 운영하는 공식 Docker Registry
  - Apache나 MySQL, Ubuntu의 공식 Image가 전부 Docker Hub에 참여해서 Image를 배포하는데 run 명령을 수행했을 때 내려받는 이미지는 여기서 다운로드
- Repository는 Registry를 구성하는 단위

### Docker 컨테이너 기반의 애플리케이션 서비스 개발 흐름
- 개발 흐름
  - 애플리케이션 코드를 개발(특정 서비스 구동을 위한 애플리케이션 코드 및 웹 화면 구성을 위한 코드 개발) => 베이스 이미지를 기반으로 Dockerfile 작성(개발에 필요한 인프라 구성 요소를 Dockerfile에 작성하는데 Docker Hub를 통해 Base Image를 다운로드 하고 다양한 구동 명령어 와 작성한 애플리케이션 코드, 라이브러리, 여러 도구를 Dockerfile에 포함시키는 과정) => Dockerfile build를 해서 새로운 이미지를 생성(docker build 명령을 통해 작성한 Dockerfile을 실행하는데 단계 별로 실행되는 로그를 화면에서 확인하며 오류 발생도 확인)  => 생성된 이미지를 이용해서 컨테이너를 실행(docker images 명령으로 이미지를 확인하고 docker run 명령으로 컨테이너를 생성해서 실행) => 마이크로 서비스인 경우는 도커 컴포즈를 이용해서 다중 컨테이너 실행(마이크로 서비스로 개발하게 되면 여러 서비스 간의 실행 순서, 네트워크, 의존성 등을 통합 관리하고 이 때는 여러 개의 yml 파일을 사용할 수 도 있음) => 마이크로 서비스인 경우는 마이크로 서비스를 테스트 => 컨테이너 애플리케이션 테스트 => 로컬 및 원격 저장소에 이미지를 저장(다른 팀 간의 공유 및 지속적인 이미지 관리) => 깃허브 등을 통한 Dockerfile 관리(단순 저장 뿐 아니라 git hub action 등을 이용하면 자동화된 빌드 기능을 이용한 이미지 생성 가능) => 동일환경에서의 지속적 애플리케이션 개발 관리

- 컨테이너 동작에 필요한 모든 내용을 사전에 코드로 작성로 작성하며 Ansible, Chef, Puppet, Vargrant 와 같은 Infra Provisioning 도구 자동화하게 되면 필요할 때 마다 애플리케이션 및 서버 환경을 적은 비용으로 빠르게 개발, 배포, 확장할 수 있는데 이 개념이 IaC(코드형 인프라 - Infrastructure as Code)

- 개발자는 개발, 테스트, 배포 시마다 모든 인프라 구성 요소를 하나 하나 수동으로 체크하거나 맞출 필요가 없고 변경 불가능한 인프라 환경에서 언제든 동일한 상태에서의 개발이 가능해지며 버전 업이나 패치 등의 작업이 필요하면 기존 이미지를 변경하지 않고 해당 작업을 수행할 새로운 이미지를 생성해서 신규 인프라 서버로 사용하는 것이 가능

## 도커 이미지 명령
- 도커 컨테이너는 일반적으로 도커 허브에서 제공하는 이미지를 기반으로 실행되는데 도커 이미지는 도커의 핵심 기술이면 코드로 개발된 컨테이너 내부 환경 정보를 고스란히 복제해서 사용할 수 있음
- 도커 컨테이너로 사용할 도커 이미지는 `docker search` 명령을 통해서 조회하며 도커 허브 및 로컬 서버 및 데스크탑에 도커 이미지를 저장하기 위해서는 Dockerfile을 통해 새로운 이미지를 생성하거나 도커 허브로부터 내려받게 됨
- Dockerfile로 생성한 이미지는 도커 허브에 로그인을 통해서 자격 증명 후 업로드하고 공개 및 비공개로 설정할 수 있음

- `docker search`: 이미지 검색
  - 명령어 형식
    - `docker search [options] 검색키워드`
    - mysql 이미지 검색 `docker search mysql`
    - 개수 제한을 할 때는 `--limit 개수` 옵션을 이용

### 도커 이미지 다운로드
  - 기본형식
    - `docker [image] pull [Options] 이미지이름[:TAG | @Image Digest]`
- debian 이미지 다운로드
  - `docker pull debian`
  - 태그를 붙이지 않으면 자동으로 latest 버전이 지정이 됨. 이미지 앞에 library는 도커 허브라 이미지를 저장하고 있는 네임스페이스
  - 세번째 값은 도커 허브에서 제공된 이미지의 분산 해시값으로 다운로드 한 이미지는 여러 계층으로 만들어지는데 그 중 핵심 정보를 바이너리 형태로 제공
  - 다이제스트 값은 원격 도커 레지스트리에서 관리하는 이미지의 고유 식별 값
  - 다운로드된 이미지 정보가 로컬에 저장되었음을 나타냄
  - docker.io는 도커 허브의 이미지 저장소 주소를 의미
- 옵션
  - `--all-tags`, `-a`: 저장소에 태그로 지정된 여러 이미지를 모두 다운로드
  - `--disable-content-trust`: 이미지를 다운로드 할 때 검증하지 않고 다운로드
  - `--platform`: 이미지의 플랫폼 지정(윈도우 도커에서 리눅스 이미지를 받아야 하는 경우 `--platform=linux`)
  - `--quiet`, `-q`: 이미지 다운로드 과정에서 화면에 나타나는 상세 출력값 숨김
- 이미지를 다운로드 받을 때 태그를 추가
  - `docker pull debian:latest`
- 저장소 추가해서 받기
  - `docker pull docker.io/library/debian:latest`
  - 도커 허브에서 이미지를 다운로드 받을 때는 저장소를 생략하는 것이 가능하지만 그 이외의 곳에서 받을 때는 저장소 생략이 안됨. 도커 허브는 index.docker.io가 앞에 붙음
- 도커 허브가 아닌 곳에서 다운로드 받기: 
  - https://gcr.io/google-samples/hello-app:1.0 이미지 다운로드 받기
  - 이 경우는 latest가 아니므로 태그를 입력해야 하고 도커 허브의 저장소가 아니므로 앞에 저장소 생략이 안됨
  - `docker pull gcr.io/google-samples/hello-app:1.0`

- 이미지 확인: `docker images` 또는 `docker image ls`
  - REPOSITORY: 이미지 이름
  - TAG: 버전 정보
  - IMAGE ID: 이미지 식별자로 원래는 64자인데 앞의 12글자만 출력
  - CREATED: Image 생성 후 경과한 시간
  - SIZE: 이미지의 크기

### 도커 이미지 세부 정보 조회
  - 명령어 형식
    - `docker image inspect [options] 이미지이름 [이미지 이름 나열..]`
    - 옵션은 `--format`이나 `-f`뒤에 JSON을 설정해서 JSON 형식으로 세부 정보를 출력하는데 이 때 정보 이름을 사용할 수 있음
  - 주요 정보
    - image ID: Id
    - 생성일: Created
    - 도커버전: DockerVersion
    - 이미지 다이제스트 정보: RootFS
    - 이미지 레이어 정보: GraphDriver
  - httpd라는 이미지를 검색해서 최신 버전을 다운로드
    - `docker search httpd`
    - `docker pull httpd:latest`
  - httpd의 상세 정보 확인
    - `docker image inspect httpd`
  - httpd 의 운영체제만 확인
    - `docker image inspect --format="{{.Os}}" httpd`
### 이미지를 구성하고 있는 레이어와 실행 정보를 확인하는 명령
- 기본 형식
  - `docker image history [옵션] 이미지 이름`
- 현재 이미지 구성을 위해 사용된 레이블 정보와 레이어의 수행 명령 그리고 크기를 조회할 수 있음
- 도커의 이미지는 여러 개의 레이어로 구성할 수 있음, 여러 개로 나누면 재사용성이 증가
- 이 명령으로 출력했을 때 사이즈를 가진 것이 레이어
- 일반적으로 레이어는 운영체제가 가장 적게 가지고 있고 이 운영체제 위에 플랫폼이 놓이고 플랫폼위에 애플리테이션이 위치하는 구조입니다.
```
                kafka
        httpd   httpd
Layer3  Layer3  Layer3
Layer2  Layer2  Layer2
Layer1  Layer1  Layer1
------  ------  ------
OS      Platform Application
```