---
title: 쿠버네티스
weight: 1
---
## 컨테이너 오케스트레이션
- 여러 개의 컨테이너를 관리하는 애플리케이션이나 프레임워크

### 장점
- 고가용성: 여러 개의 애플리케이션 인스턴스를 실행해서 실패한 애플리케이션의 인스턴스를 자동으로 새로운 인스턴스로 교체 가능
- 규모 확장성

### 오케스트레이션 도구
- Docker Swarm: 도커 컨테이너 엔진을 개발한 팀에서 개발을 했는데 설정이나 실행은 쉽지만 유연성이 부족
- Apache Methos
  - 데이터 센터와 클라우드 환경 모두에서 컴퓨팅, 메모리 및 스토리지를 관리하는 Low Level 오케스트레이션 도구
  - 기본적으로 컨테이너를 관리하지는 않지만 메소스 위에서 실행되는 마라톤은 완전하게 확장된 컨테이너 오케스트레이션 도구가 되고 쿠버네티스도 메소스 위에서 실행 가능
- 쿠버네티스
  - k8s라는 이름으로도 불리는 구글이 개발한 오픈소스 컨테이너 오케스트레이션 도구
  - 오픈소스가 된 후에는 엔터프라이즈 환경에서 컨테이너를 실행하고 오케스트레이션 하는 사실상의 표준
  - 매우 큰 커뮤니티를 가진 성숙도 높은 제품
  - 아파치 메소스보다 조작이 간단하고 도커 스웜보다는 유연성이 뛰어남

## 쿠버네티스 아키텍쳐
### 노드 유형
- 노드는 VM, 베어메탈 호스트, 라즈베리 파이 등 다양
  - Master Node: Control Plane Application 실행을 담당
  - Worker Node: 쿠버네티스에 배포되는 애플리케이션 실행을 담당
### Control Plane
- 마스터 노드에서 실행되는 애플리케이션과 서비스의 집합
- 고도로 특화된 서비스
  - kube-apiserver: 쿠버네티스에 전송된 커맨드를 처리
  - kube-scheduler: 워크 로드를 배치할 노드를 결정하며 이는 상당히 복잡한 과정일 때도 있음
  - kube-controller-manager: 클러스터와 클러스터에서 실행 중인 애플리케이션이 원하는 설정대로 구성되도록 고수준의 제어 루프를 제공
  - etcd: 클러스터 설정을 포함하는 분산 키-값 저장소
- 이 컴포넌트들은 모든 마스터 노드에서 실행되는 시스템 서비스 형태로 되어 있는데 클러스터 전체를 수동으로 실행 및 생성할 수 있지만 클러스터 생성 라이브러리나 클라우드 벤더가 제공하는 서비스(EKS)를 사용하면 프로덕션 환경에서 자동으로 시작할 수 있음
### 쿠버네티스 API Server
- 일반적으로 443 포트를 사용해 HTTPS 요청을 받는 컴포넌트
- 쿠버네티스 API 서버에 구성 요청시 etcd에서 현재 클러스터 설정 정보를 확인하고 필요 시 변경함
- 쿠버네티스 API는 RESTful API
- 쿼리 경로로 API 버전과 각 쿠버네티스 리소스 유형에 대한 엔드포인트로 구성
- 쿠버네티스를 확장할 때 API 그룹을 기반으로 동적 엔드포인트 세트를 가진 API를 이용해서 커스텀 리소스를 동일한 수준의 RESTful API로 노출할 수 있음
``` {filename="쿠버네티스 동작 과정"}
                                                         ___________클러스터_____________________________________
사용자 인증 -> 명령어 인증 -> API Server ->-------------->| kubelet -> Docker, containerd가 pod 생성              |
    |                           ||                      | kubelet  -> 스케쥴러(파드가 생성 워커 파드를 찾아서 조정)|
파드 생성 알림, etcd 저장<--------|                       |------------------------------------------------------|
(실제로 생성됐는지는 확인해야)    etcd  
```
### etcd
- API Server는 파드를 만든다는 사실을 etcd에 알리고 사용자가에게 파드가 생성되었음을 알리는 기능을 수행
- etcd는 클러스터의 상태를 저장하는 컴포넌트
- key-value 형태로 저장

### 스케쥴러
- 파드를 위치시킬 적당한 워커 노드를 확인하는 컴포넌트
- 워커 노드가 확인되면 API Server에게 알리고 그러면 etcd에 파드가 생성될 것이라고 저장

### kubelet
- API Server로부터 생성될 워커 노드에 있는 kubelet에게 파드 생성 정보를 전달하고 kubelet은 이 정보를 기반으로 파드를 생성
- 파드가 생성되면 kubelet은 API Server에 생성되었다는 사실을 알려주고 그 정보를 etcd에 업데이트(어떤 워커 노드에 어떤 파드가 생성되었는지 저장)

### Controller Manager
- 컴포넌트의 상태를 지속적으로 모니터링하는 동시에 실행 상태를 유지하는 역할을 수행
- 특정 노드 와 통신이 불가능하다고 판단되면 해당 노드에 할당된 파드를 제거하고 다른 워커 노드에서 파드를 생성해 서비스가 계속되도록 함

### Proxy
- 클러스터의 모든 노드에서 실행되는 네트워크 프록시
- 노드에 대한 네트워크 규칙을 관리
- 클러스터 내부와 외부에 대한 통신을 담당

### Container Runtime
- 컨테이너 런타임은 컨테이너 실행을 담당
- 여러 종류의 런타임을 지원하는데 최신 버전에서는 도커는 지원 중단되고 컨테이너디와 크라이오 등을 사용<br>

- etcd, API Server, Controller Manager, Scheduler는 Master Node에 존재하고 Proxy, Kubelet, Container Runtime은 워커 노드에 존재
```
|======================Cluster=====================================|
||==master node========|            |-----worker node---------|    |
||etcd <- API server<--|------------|----Proxy(통신)           |    |
||           ^      <--|------------|-----kubelet(명령실행)    |    |
||           |         |            |       |                 |     |
||Controller Manager / |            |  Container Runtime      |     |
||       스케쥴러       |            |                         |     |
||=====================|            |-------------------------|     |
|===================================================================|
```
## 쿠버네티스 컨트롤러
```
Deployment -> Replica Set ->
Daemon Set ---------------->
Stateful Set -------------->   POD
Cron Job -> Job ----------->
Application Controller ---->
```
- 파드를 관리하는 역할을 수행하는 객체

### Deployment
- 쿠버네티스에서 상태가 없는 애플리케이션을 배포할 때 사용하는 가장 기본적인 컨트롤러
- 레플리카 셋의 상위 개념이면서 파드를 배포할 때 사용
- 파드를 배포할 때는 Deployment나 서비스를 이용
- yaml 파일 기본 구조
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: 이름
 labels:
   app: 레이블 설정

spec:
 replicas: 레플리카 개수
 selector:
 
 metadata: 파드 실행 설정
```
### ReplicaSet
- 몇 개의 파드를 유지할지 결정하는 컨트롤러
- ReplicaSet과 ReplicaController는 다름
- ReplicaSet은 집합 기반으로 in, not in, exists 같은 연산자를 지원하지만 ReplicationController는 등호 기반이라서 =, !=를 지원
- ReplicaSet은 롤링 업데이트를 사용할려면 Deployment를 사용해야 하지만 ReplicationController는 롤링 업데이트 옵션을 지원

### Job
- 하나 이상의 파드를 지정하고 지정된 수의 파드가 성공적으로 실행되도록 해주는 컨트롤러
- 노드의 하드웨어 장애나 재부팅 등으로 파드가 비정삭적으로 작동하면 다른 노드에서 파드를 시작해 서비스가 지속되도록 함
- manifest
```yml
apiVersion: batch/v1
kind: job
spec:
metadata:
  name: 잡이름
spec:
  template:
    metadata:
      name: 하나의 템플릿 이름
    spec:
      containers:
        - name: 컨테이너이름
          image: 이미지이름
          command: [명령어]
      restartPolicy: 정책
```

### CronJob
- 잡의 일종으로 특정 시간에 특정 파드를 실행시키는 것 과 같이 지정한 일정에 따라서 잡을 실행시킬 때 사용
- 주로 애플리케이션 프로그램의 실행이나 데이터베이스의 경우 백업 등의 작업을 설정
- manifest
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: 잡이름
spec:
  schedule: "* * * * *" # linux의 cron 작업과 동일
  jobTemplate:
    spec:
      template:
      metadata:
        name: 하나의 템플릿 이름
      spec:
        containers:
          - name: 컨테이너이름
            image: 이미지이름
            args:
              - /bin/sh
              - -c
              - date; echo Hello this is Cron test
        restartPolicy: 정책
```

### DaemonSet
- Deployment 처럼 파드를 생성하고 관리
- Deployment는 파드의 개수와 배포 전략을 설정하지만 데몬 셋은 특정 노드 또는 모든 노드에 파드를 배포하고 관리
- 노드마다 배치되어야 하는 성능 수집 및 로그 수집 같은 작업에 사용
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: 이름
  labels:
    app: 레이블 # 데몬셋을 식별할 수 있는 레이블을 설정
spec:
  selector:
    matchLabels:
      app: 앱의 레이블
  template:
    metadata:
      labels:
        app: 앱의 레이블 # 파드의 레이블
    spec:
      tolerations:
      - key: node-role/master
        effect: NoSchedule
```
- taint와 tolerations
  - 쿠버네티스 클러스터를 운영하다 보면 특정 워커 노드에는 특정 성격의 파드만 배포하고자 하는 경우가 있음
  - GPU가 설치된 파드에는 GPU가 필요한 서비스만 배포하고자 하는 경우가 대표적
  - 테인트가 설정된 노드에는 일반적으로 사용되는 파드는 배포될 수 없으나 톨러레이션을 적용하면 배포할 수 있음
  - 테인트 설정 `kubectl taint node [NODE_NAME] [KEY]=[VALUE]:[EFFECT]`
  - EFFECT에는 3가지 옵션이 있음
    - NoSchedule: 톨러레이션이 완전치 일치하는 파드만 배포할 수 있음
    - NoExecute: 기존에 이미 배포된 파드를 다른 노드로 옮기고 새로운 파드는 배포하지 못하도록 하는 것
    - PreferNoSchedule: NoSchedule 과 유사하지만 지정된 노드에는 새로운 파드가 배포되지 않지만 리소스가 부족할 때는 배포할 수 있는 차이가 있음

## 쿠버네티스 서비스
```yml
K8S Cluster
└── Deployment
    └── ReplicaSet (2 replicas)
        ├── Pod (Node 1)
        ├── Pod (Node 2)
        └──     (Node 3)
```
- 파드는 쿠버네티스 클러스터 안에서 옮겨다니는 특성이 있음
- 각각의 파드는 별도의 IP를 할당받음
- 동적으로 변하는 파드에 고정된 방법으로 접근하기 위해서 사용하는 것이 service
- 서비스를 사용하면 파드가 클러스터 내의 어디에 떠 있든지 고정된 주소를 이용해서 접근할 수 있음
- 클러스터 외부에서도 접근할 수 있음
```
alias, symbolic link 쓰는 이유 두가지:
  1 짧게 쓰기 위해
  2 이름이 바뀔 수 있기 때문
    ex Linux의 systemd -> init.d 로 바뀜
       Linux의 bash
      Node의 Pod이 다른 Node로 옮겨가도 IP는 바뀌지만 IP가리키는 이름은 바뀌지 않음
agile은 같이 일하는 것, 서비스를 분할하는 것이 아닌
```
- 서비스의 종류
  - Cluster IP: 쿠버네티스 클러스터 내의 파드들은 기본적으로 외부에서 접근할 수 있는 IP를 할당받지 않지만 클러스터 내부에서는 파드들이 통신할 방법을 제공하는데 이것이 클러스터 IP 입니다. 클러스터 내의 모든 파드가 해당 클러스터 IP 주소로 접근할 수 있습니다.
  - NodePort: 서비스를 외부로 노출할 때 사용하는 것으로 노드포트로 서비스를 노출하기 위해 워커 노드의 IP 와 포트를 이용합니다. 워커 노드의 IP가 192.168.2.3 이고 30010 포트를 사용한다면 192.168.2.3:30010 포트로 외부에서 접근 가능
  - Load Balancer: 로드 밸런서는 주로 퍼블릭 클라우드에 존재하는 로드 밸런서에 연결하고자 할 때 사용하는데 이 경우는 Load Balancer의 외부 IP를 통해서 접근
```yml
Load Balancer
├── 서비스 1 (NodePort) (Public IP)
│   └── K8S Cluster 1 (Cluster IP)
│       └── Deployment
│           └── ReplicaSet (2 replicas)
│               ├── Pod (Node 1) (Private IP)
│               ├── Pod (Node 2) (Private IP)
│               └──     (Node 3)
└── 서비스 2 (NodePort) (Public IP)
    └── K8S Cluster 2 (Cluster IP)
        └── Deployment
            └── ReplicaSet (2 replicas)
                ├── Pod (Node 1) (Private IP)
                ├── Pod (Node 2) (Private IP)
                └──     (Node 3)
```
- manifest
```yml
apiVersion: v1
kind: Service
metadata:
  name: t-service
spec:
  selector:
    app: webserver #워커 노드에 떠 있는 컨테이너 중 webserver를 선택
  ports:
    - protocol: TCP
      port: 80 # 서비스에서 컨테이너 애플리케이션과 매핑 시킬 포트 번호
      targetPort: 8080 # 컨테이너에서 구동 중인 애플리케이션 포트번호
```

## 쿠버네티스 통신
### 쿠버네티스 통신의 특징
- 파드가 사용하는 네트워크와 호스트(노드)가 사용하는 네트워크는 다름
- 노드 내의 파드들은 가상의 네트워크를 사용하고 호스트는 물리 네트워크를 사용
- ipconfig 명령 시 실제 인터페이스는 무선LAN WIFI어댑터와 이더넷 어댑터 두개
  - 나머지는 가상 인터페이스
- 같은 노드에 있는 파드끼리만 통신이 가능
  - 같은 노드에 떠 있는 파드끼리는 통신이 가능하지만 다른 노드의 파드 또는 외부와의 통신은 불가능
- 다른 노드의 파드와 통신하려면 CNI 플러그인이 필요
  - CNI(Container Network Interface)는 컨테이너 간의 통신을 위한 네트워크 인터페이스
  - CNI Plugin은 컨테이너들의 네트워크 연결하거나 삭제하면 특히 삭제할 때는 할당된 자원을 제거
  - 쿠버네티스를 설치할 때 자동으로 구성되지 않으므로 별도로 설치해야 함

### 같은 파드에 포함된 컨테이너간 통신
- 파드: docker-compose와 유사 (1개 이상의 컨테이너로 구성)
- 같은 파드 내의 컨테이너가 통신은 직접(로컬호스트 통신) 통신이 가능
- 하나의 파드에는 하나의 가상 네트워크가 만들어지고 그 파드 내에 존재하는 컨테이너들은 같은 가상 네트워크를 사용
- 하나의 파드 내에 존재하는 컨테이너들은 모두 동일한 IP를 사용
- 하나의 파드 안에 있는 컨테이너들은 포트 번호를 이용해서 구분
- 하나의 파드가 만들어지면 가상의 네트워크가 만들어지고 브릿지가 존재하고 이 브릿지가 노드의 실제 NIC 와 연결됨
```
eth : Node
└── Bridge
    └── veth : Pod
```
### 단일 노드에서 파드간 통신
- 단일 노드에 떠 있는 파드들은 같은 네트워크 대역(veth0: 172.10.0.2/24, vetho: 172.10.0.3/24)을 가지므로 브릿지를 이용해서 통신을 수행할 수 있음

### 다수의 노드에서 파드 간 통신
- 각 노드에서는 노드 별로 가상의 네트워크를 생성하기 때문에 서로 다른 노드 간에 동일한 IP를 가질 수 있기 때문에 단순한 방식으로는 통신이 불가능함
- 오버레이 네트워크를 이용해서 통신(NAT)
- 오버레이 네트워크는 노드에서 사용하는 물리적인 네트워크 위에 가상의 네트워크를 구성하는 것
- 오버레이 네트워크를 사용하면 클러스터로 묶인 모든 노드에 떠 있는 파드 간의 통신이 가능
- 쿠버네티스에서는 기본적으로 kubenet이라는 기본적이고 간단한 네트워크 플러그인을 제공하는데 이 플러그인이 오버레이 네트워크를 구성함
```{filename="오버레이 네트워크"}
Kubernetes Cluster
├── Node 1: eth0(public IP)
│   ├── Pod A (172.16.0.1)
│   └── Overlay Network
│       └── 10.0.0.2/24
└── Node 2: eth0(public IP)
    ├── Pod B (172.16.0.1)
    └── Overlay Network
        └── 10.0.0.3/24
```
### 파드와 서비스 간의 통신
- 서비스의 특성
  - 서비스도 파드처럼 IP를 가짐
  - 파드와 서비스에서 사용하는 IP 대역은 다릅니다.
- Client 파드가 네트워크를 통해서 다른 노드의 Web Server 파드에 http 요청을 하는 경우
  - Client 파드는 서비스 이름으로 http 요청을 보냄
  - 클러스터 DNS 서버는 서비스이름에 해당하는 서비스 IP를 Client 파드에게 전달
  - 서비스 IP를 전달받은 Client 파드는 해당 IP를 이용해서 http 요청을 보내는데 이 때 클라이언트 워커 노드는 라우팅 테이블에 서비스 IP가 있는지 검색하고 검색 결과 IP가 없으면 해당 요청을 라우터/게이트웨이 로 전달
  - 라우터 나 게이트웨이는 서비스 IP를 찾지 못할 수 있는데 이 경우 linux에 있는 netfilter 와 iptables 기능을 이용해서 찾아옴
  - netfilter는 주소변환, 규칙 기반의 패킷 처리 엔진으로 서비스 IP를 특정 IP로 변경할 수 규칙을 저장하고 변환하는 역할을 함
  - 이런 이유로 netfilter를 프록시라고 함
  - 이 정보를 저장하고 있는 것이 iptables

### 외부 와 서비스 간의 통신
- NodePort: 노드 포트란 노드 IP에 포트를 붙여서 외부에 노출시키는 것
- Load Balancer: 로드 밸런서는 로드밸런서 IP를 이용해서 클러스터 외부에서 파드에 접근할 수 있도록 해주는 기능
- Ingress: 클러스터 외부에서 내부로 접근하는 요청을 어떻게 처리할 것인지에 관한 규칙의 모음이고 실제로 동작시키는 것은 Ingress Controller
- 서비스를 넓게 이야기 할 때는 위의 3개를 포함하는 것으로 이야기 하고 좁게 이야기 할 때는 클러스터 내부에서 다른 노드에 접근할 수 있도록 해주는 것만 서비스라고 하기도 함