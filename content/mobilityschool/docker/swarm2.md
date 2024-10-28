---
title: Docker Swarm 2
weight: 8
---

## Virtual Box를 이용한 3개의 리눅스 서버 만들기
### 1. 복제
- VM을 중지하고 [파일] 메뉴에서 [복제]를 선택
  - 이름을 설정
  - 완전한 복제를 선택
  - MAC 주소 정책에서 모든 네트워크 어댑터의 새 MAC 주소 설정
  - 설정->네트워크->네트워크 -> 어댑터에 브릿지 클릭 -> ssh 접속 가능
- 2번 복제를 수행

### 2. IP설정
- 먼저 자신의 호스트 컴퓨터에서 IP 와 서브넷 마스크 그리고 Gateway를 확인
- IP 가 192.168.201.216 / 255.255.252.0 / 192.168.200.1 입니다.
- GUI 환경에서는 상단의 맨 오른쪽 아이콘을 누르고 [Wired Connecting]을 선택하고 [Wired Settings]를 클릭하고 설정 아이콘을 클릭한 후  IPv4를 선택한 후 Manual 을 선택해서 IP 와 SubnetMask 그리고 Gateway를 설정

- 1번VM
  - IP: 192.168.201.100
  - Gateway: 192.168.201.1

- 2번 VM
  - IP: 192.168.201.101
  - Gateway: 192.168.201.1

- 3번 VM
  - IP: 192.168.201.102
  - Gateway: 192.168.201.1

### 3. hostname 변경
- `sudo hostnamectl set-hostname 호스트이름`
  - swarm-manager, swarm-worker1, swarm-worker2
- `cat /etc/hostname`

- `sudo reboot now`

### 4. DNS 설정
- swarm-manager의 /etc/hosts 파일에서 수정
  - 127.0.0.1     localhost 밑에 붙여넣기
    ```
    127.0.1.1       swarm-manager

    192.168.201.100 swarm-manager
    192.168.201.101 swarm-worker1
    192.168.201.102 swarm-worker2
    ```
    - `sudo reboot now`
- swarm-worker1의 /etc/hosts 파일에서 수정
  - 127.0.0.1     localhost 밑에 붙여넣기
  ```
  127.0.1.1       swarm-worker1

  192.168.201.100 swarm-manager
  192.168.201.101 swarm-worker1
  192.168.201.102 swarm-worker2
  ```
  - `sudo reboot now`
- swarm-worker2의 /etc/hosts 파일에서 수정
  - 127.0.0.1     localhost 밑에 붙여넣기
  ```
  127.0.1.1       swarm-worker2

  192.168.201.100 swarm-manager
  192.168.201.101 swarm-worker1
  192.168.201.102 swarm-worker2
  ```
  - `sudo reboot now`
- 각 컴퓨터에서 `ping 이름` 으로 명령 수행해도 응답이 옴

## 다중 호스트 기반의 도커 스웜 모드 클러스터
### 도커 스웜 모드
- 물리적 서버 클러스터를 통해서 컨테이너를 확장하기 위한 도커 고유의 플랫폼으로 여러 서버에 걸쳐 간단한 분산 워크로드를 구현할 수 있음
- 여러 서버 클러스터에 컨테이너화 된 애플리케이션 기반의 마이크로서비스를 배포해서 다양한 런타임 환경에서 애플리케이션의 효율성과 가용성을 유지하도록 개발
- 도커 스웜 모드는 동일한 컨테이너를 공유하는 여러 클러스터 내의 노드에서 애플리케이션을 원할하게 실행할 수 있도록 하는 도커 자체 컨테이너 오케스트레이션 도구
- 클러스터화된 각 서버의 도커 엔진을 통해 자원을 마치 하나의 서버처럼 풀링해서 스웜(여러 대의 서버를 묶어서 하나의 서버처럼 사용)을 형성
- 예전 버전에서는 별도의 스웜 엔진을 가지고 있었지만 지금은 도커 엔진으로 통합되어 기본 도커 오케스트레이션 도구 모델로 사용되기 시작

### 주요기능
- 도커 엔진과 통합된 다중 서버 클러스터 환경: 도커 엔진에 포함된 도커 스웜 모드를 통해 별도의 오케스트레이션 도구를 설치하지 않아도 컨테이너 애플리케이션 서비스를 배포하고 관리할 수 있음

#### 역할이 분리된 분산 설계
- 다중 서버를 클러스터에 합류시키면 모든 도커 스웜 모드의 노드는 각각 다른 역할을 수행하게 됩니다.
- Manager Node, Leader Node, Worker Node
- 단일 관리자 환경이라면 Manager Node 와 Worker Node로만 나누게 되지만 다중 매니저 모드로 구성하게 되면 매니저 중 하나를 Leader Node로 설정해서 수행
- Manager Node는 클러스터의 관리 역할로 컨테이너 스케쥴링 서비스 및 상태 유지 등을 제공하고 작업자 노드는 컨테이너를 실행하는 역할만 수행
- 쿠버네티스의 마스터 노드는 기본적으로 작업자 노드의 전체적인 관리만 수행하고 서비스 컨테이너는 수행하지 않지만(변경은 가능) 도커 스웜의 관리자 노드는 작업자 노드의 역할인 서비스 컨테이너도 수행할 수 있음
- 관리 역할을 수행하는 노드의 부하를 고려해서 각 역할은 분리해 사용하는 것을 권장
- 매니저 노드에 서비스 컨테이너를 수행하지 않도록 역할 분리를 수행하는 방법은 도커 서비스 생성 시 --constraint node.role!=manager 옵션을 사용하면 됨

#### 서비스 확장과 원하는 상태 조정
- 도커 스웜 모드에서 서비스 생성 시 안정적인 서비스를 위해 중복(복제 - replica)된 서비스 배포를 할 수 있고 초기 구성 후 스웜 관리자를 통해 애플리케이션 가용성에 영향을 주지 않고도(멈출 필요가 없음) 서비스 확장 및 축소를 수행
- 서비스가 배포되면 매니저 노드를 통해서 지속적으로 모니터링을 수행
- 사용자가 요청한 상태와 다르게 서비스 장애(노드 장애 및 서비스 실패)가 생길 경우 장애가 발생한 서비스를 대체할 복제본을 자동으로 생성해서 사용자의 요구를 지속하는데 이를 요구 상태 관리(desire state management)라함
- 오케스트레이션 도구들의 대부분의 목적은 요구 상태 관리

#### 서비스 스케쥴링
- 클러스터 내의 노드에 작업(task) 단위의 서비스 컨테이너를 배포하는 작업
- 선택 전략
  - 모든 작업자 노드에 균등하게 할당하는 spread 전략(loadbalancing의 round robin)
  - 작업자 노드의 자원 사용량을 고려하여 할당하는 binpack 전략
  - 임의의 노드에 할당하는 random 전략
- `swarm manage --strategy` 옵션을 이용해서 변경 가능
- 도커 스웜 모드는 고가용성 분산 알고리즘을 사용하는데 이는 생성되는 서비스의 복제본을 분산 배포하기 위해서 현재 복제본이 가장 적은 작업자 노드 중에서 이미 스케줄링된 다른 서비스 컨테이너가 가장 적은 작업자 노드를 우선 선택

#### 로드 밸런싱
- 도커 스웜 모드를 초기화하면 자동으로 생성되는 네트워크 드라이버 하나가 **인그레스 네트워크(ingress network)** 인데 인그레스 네트워크를 통해 서비스의 노드간 로드 밸런싱 과 외부에 서비스를 노출
- 도커 스웜 모드는 서비스 컨테이너에 Published Port를 자동으로 할당하거나 수동으로 노출할 포트를 구성할 수 있고 서비스 컨테이너가 포트를 오픈하면 동시에 모든 노드에서 동일한 포트가 오픈되기 때문에 클러스터에 합류되어 있는 어떤 노드에 요청을 전달해도 실행 중인 서비스 컨테이너에 자동으로 전달
- 스웜 모드의 라우팅 메시에 모두 참여하기 때문에 이것이 가능
- `init` 명령을 통해 확장된 노드 인식

#### 서비스 검색 기능
- 도커 스웜 모드는 서비스 검색을 위한 자체 DNS 서버를 통해서 서비스 검색 기능을 제공
- 도커 스웜 모드의 매니저 노드는 클러스터에 합류된 모든 노드의 서비스에 고유한 DNS 이름을 할당하고 할당된 DNS 이름은 도커 스웜 모드에 내장된 DNS 서버를 통해 스웜 모드에서 실행 중인 모든 컨테이너를 조회하는 것이 가능한데 이를 service discovery

#### 롤링 업데이트
- 노드 단위로 점진적인 업데이트를 수행하는 것
- 롤링 업데이트는 각 작업자 노드에서 실행 중인 서비스 컨테이너를 노드 단위의 지연적 업데이트를 수행하는 것
- 업데이트 지연 시간을 설정해서 하나의 작업자 노드에서 기존 컨테이너를 중지하고 새로운 서비스 컨테이너를 생성하고 성공하면 지연 시간만큼 기다린 후 다른 작업자 노드에서 동일한 작업을 수행
- 새 버전의 서비스 컨테이너를 하나씩 늘려가면서 이전 버전의 서비스 컨테이너를 줄여나가는 방식
- 업데이트 실패 시 중지 및 롤백 기능도 제공
- 다른 방법
  - 모든 업데이트 노드가 생성될 때까지 기다렸다가 한꺼번에 바꾸기 - 실패 시 전부 다운될 위험 
  - 새로운 버전 그대로 두기, A/B테스트 사용 예) 3개 + 업데이트3개

### 도커 스웜 모드 초기 연결 구성
#### 도커 스웜 모드 초기화(manager 에서 작업)
- Swarm 모드 활성화 여부를 확인
  - `docker info | grep Swarm`
  - 현재는 inactive 상태
- 초기화 명령: `docker swarm init --advertise-addr 매니저IP`
  - `docker swarm init --advertise-addr 192.168.201.100`
  - 노드가 참여할 때 사용하는 토큰 값이 출력됨
  - Swarm 모드가 되면 2377번 포트와 7946번(작업자 노드 간의 통신을 위한 포트) 포트가 개방됨
  - 인그레스는 4789번 포트 사용
  - `sudo netstat -nlp | grep dockerd`

#### 도커 스웜 모드 작업자 노드 연결
- `docker swarm join --token 토큰값 매니저IP:2377`
- swarm-worker1로 이동
  - `docker swarm join --token SWMTKN-1-3pvhakwl5zhzc345vuno9y57gz1gacjg6qxkbqxc4fxqggh1gs-297k5lb7mpwityudsi7ljf26q 192.168.201.100:2377`
- 매니저에서 토큰 확인: `docker swarm join-token worker`
- 다중 매니저 노드를 구성하는 경우에는 매니저 노드 추가에 대한 조인 키도 조회가 가능: `docker swarm join-token manager`

#### 스웜 명령어
- 노드 확인: `docker node ls`
- 도커 정보를 확인해보면 Swarm도 확인하는 것이 가능: `docker info`
- 새로 만들어진 네트워크 확인: `docker network ls`
```
adam@swarm-manager:~$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
4810e3536dea   bridge            bridge    local
af6a7225f58d   docker_gwbridge   bridge    local
fb625edba68e   host              host      local
du2xxydflei5   ingress           overlay   swarm
b1a4049e1ff1   none              null      local
```
- 운영하는 도중 노드를 확장하고자 하는 경우는 새로운 토큰이 필요할 수 있는데 이 경우는 2개의 명령으로 토큰을 받는 것이 가능
  - `docker swarm join-token --rotate worker`
  - `docker swarm join-token -q worker`: 토큰값만 보여줌
- Error Response from daemon: rpc error ..... 접속에러가 나는 경우에는 방화벽 때문일 수 있음
```
sudo ufw disable

sudo ufw status

sudo systemctl stop firewalld.service
```
- 노드 연결 중지
  - worker: `docker swarm leave`
- 도커 스웜 시각화 도구
  - `docker service create --name=viz_swarm -p 7070:8080 --constraint=node.role==manager --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock dockersmaples/visualizer`
- 스웜 모드를 GUI로 관리하기 위한 도구
  - `docker run -it --rm --name swarmpit --volume /var/run/docker.sock:/var/run/docker.socket swarmpit/install`

### 서비스 배포
- `docker run` 대신에 `docker service create`를 이용하는데 `docker run` 명령과 거의 유사
- nginx, busybox, alpine:3

### 서비스 생성 실습
- swarm-manager 컴퓨터를 인터넷이 되도록 설정해서 nginx, busybox, alpine:3 이미지 다운로드
- swarm-manager 컴퓨터에서 docker swarm 모드 초기화
- alpine:3 버전을 이용한 서비스 생성
  - `docker service create --name swarm-start alpine:3 /bin/sh -c "while true; do echo'Docker Swarm mode Start'; sleep 3; done"`
- 서비스 확인: `docker service ls`
- 컨테이너 정보 확인: `docker service ps 컨테이너이름`
- 로그 확인: `docker service logs -f 컨테이너아이디`
- 서비스 중지: `docker service rm 컨테이너이름`

### docker-compose `--scale` 옵션 과 docker swarm 의 replica 와 차이점
- docker-compose 에서 scale 옵션은 하나의 노드(도커 엔진)에 여러 개의 컨테이너를 생성하는 것이고 docker swarm은 여러 도커 엔진에 여러 개의 컨테이너를 생성하는 것
- docker-compose에서 scale 옵션을 이용해서 하나의 이미지로 여러 개의 컨테이너로 만들 때 주의할 점은 포트를 외부로 노출한다면 포트 여러 개를 매핑해야 함

- swarm-manager에서 nginx 서비스를 2개 생성
  - `docker service create --name web-alb --constraint node.rol==worker --replicas 2 --8001:80 nginx`
  - 여러 개의 서비스를 만들고자 하는 경우는 replicas 옵션을 이용하는데 이 때 노드의 개수보다 더 많은 서비스 개수를 요청하면 특정 노드에 여러 개가 배치됨
- 서비스를 만들 때 mode를 global로 설정하면 모든 노드에 전부 생성
- global을 설정하면 노드가 추가되는 경우 자동으로 확장이 됨
- replicas를 이용해서 컨테이너를 생성하면 자동 복구 기능을 사용할 수 있음
- service update를 수행하면 롤링 업데이트를 수행
  - 서비스를 만들 때 --update-delay 옵션을 이용해서 하나가 업데이트가 되고 난 후 얼마나 대기 할 것인지 설정할 수 있음
  - 서비스를 만들 때 옵션을 지정하지 않으면 1개씩 롤링 업데이트가 수행되지만 `--update-parallelism`에 개수를 설정하면 개수만큼 롤링 업데이트가 발생함
  - 실패한 경우 rollback을 이용해서 이전으로 돌아갈 수 있음
  - 돌아갈 수 있는 경우는 도커 정보를 확인하면 됨
- docker-compose.yml 파일을 가지고 만든 컨테이너는 `docker stack deploy --compose 파일경로 서비스이름` 으로 배포함