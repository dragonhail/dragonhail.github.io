---
title: AWS
weight: 9
---
## Amazon EC2(Elastic Computing Cloud)
- Amazon Web Service에서 컴퓨터 용량을 제공하는 서비스
- **EC2는 Managed Service**(관리 주체가 Public Cloud 사업자가 제공하는 서비스 - 백업과 업데이트를 사업자가 수행)가 **아님**
- 서버 및 네트워크 운영은 AWS가 담당하지만 운영체제를 포함한 필요한 소프트웨어는 사용자가 직접 설치하고 운영
- 장점
  - 클릭 한 번으로 쉽게 생성
  - 다양한 하드웨어와 운영체제 조합이 가능
  - 생성과 삭제가 자유로움
  - 스케일 업 다운이 편리
- 적합하지 않은 경우
  - 단순히 서버 1대로 구성하고 변화가 거의 없는 경우
```
SaaS: 애플리케이션
PaaS: 개발환경 빌려줌
Iaas(H/W, OS, Network, Disk): 인프라 빌려줌

Managed Service: 업데이트나 백업을 Public Cloud에서 수행, SaaS
Managed Service가 아님: 관리 주체가 사용자, IaaS
```
### 주요 구성
- Instance: AWS에 존재하는 가상 서버
- AMI: 운영체제 이미지
- Key Pair: 외부에서 인스턴스에 접속할 때 인증을 위해 사용하는 키 인증서는 보관을 잘 해두는 것이 중요
- EBS: 스토리지
- 보안 그룹: 가상의 방화벽
  - Inbound: 인스턴스 외부에서 인스턴스 내부로 요청
  - Outbound: 인스턴스 내부에서 외부로 요청

### 사용 절차
- 기본 리전 설정
  - region: 지리적으로 떨어진 독립적인 위치를 의미
  - AZ(Availability Zone: 가용 영역): region 내에서 물리적으로 격리된 공간
- 인스턴스 시작을 누르고 인스턴스 옵션을 설정
  - 이름
  - 운영체제
  - 하드웨어
  - 키페어: 외부에서 접속할 때 사용할 키와 관련된 파일
  - 보안 그룹: 일정의 방화벽으로 기존의 내용을 선택할 수 있고 새로 만들 수도 있는데 기본적으로 외부에서 SSH로 접속할 수 있도록 SSH 만 개방되어 있음
  - 저장장치: 기본적으로 8G가 제공되는데 30G까지는 프리티어 계정에서 무료
- 인스턴스를 생성하면 Public IP 와 Private IP 그리고 Public IP 와 매핑되는 도메인 한 개가 제공
- 인스턴스에 접속
  - AWS에 로그인해서 접속
  - 외부에서 SSH를 이용해서 접속
  ```
  ssh -i pem파일경로 ubuntu@IP 또는 DNS
  ```
### 웹 서버로 사용
- 외부에서 80 번 포트로 접속
```
sudo apt-get update
sudo apt install apache2 -y
sudo service apache2 start # 아파치 시작

# 확인
ps aux | grep apache2
systemctl status apache2
```
- 다른 컴퓨터에서 EC2 공인 IP를 브라우저에 입력
  - apache2 웹 페이지가 보이면 보안 그룹에서 http를 인바운드에서 처리할 수 있도록 설정을 한 것
  - 이 경우 외부에서 접속이 안되면 보안 그룹에서 80번 포트를 외부에서 접속할 수 있도록 허용을 해야 함
- 보안 그룹 편집
  - 인스턴스 상세보기 화면에서 하단의 [보안] 탭을 클릭
  - 보안 그룹의 ID를 클릭하면 편집 화면으로 들어가서 편집

### MySQL을 설치해서 외부에서 접속이 가능하도록 설정
- 패키지 정보 업데이트
```
sudo apt update
```
- MySQL 설치: 패키지 이름 - mysql-server
```
sudo apt install -y mysql-server
```
- 설치 확인
```
mysql --version
```
- MYSQL에 접속: 초기 비밀번호는 없거나 계정 이름
```
sudo mysql -u root -p
```
- 관리자 비밀번호를 수정
  - `use mysql`
  - `alter user 'root'@'localhost' identified with mysql_native_password by '비밀번호'`
  - `flush privileges;`
- 계정 생성
  - 데이터베이스 생성: `create database 데이터베이스 이름`
  - 유저 생성: `create user 유저이름@'%' identified by '비밀번호'`
  - 권한 부여: `grant all privileges on 데이터베이스이름.* to '유저이름'@'%';`
  - `flush privileges`
  - %에는 IP를 설정할 수 있는데 IP를 설정하면 설정한 곳에서만 접속이 가능함
  - 특정 애플리케이션에서만 DB에 접속하는 경우 설정하는데 최근에는 VPC를 이용해서 해결
```
mysql> create database itstudy;
Query OK, 1 row affected (0.02 sec)

mysql> create user dh@'%' identified by '0000';
Query OK, 0 rows affected (0.04 sec)

mysql> grant all privileges on itstudy.* to 'dh'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```
- MySQL은 외부 접속을 기본적으로 차단
  - 이 설정은 /etc/mysql/mysql.conf.d/mysqld.cnf 파일에 되어 있음
```cnf {filename="mysqld.cnf"}
bind-address  = 0.0.0.0
```
  - `sudo service mysql restart`로 설정 적용
  - dbeaver에서 edit connection클릭 후 EC2의 public IP, mysql user, pw 입력 후 driver properties 탭의 allowPublicKeyRetrieval 속성을 true로 설정

## EC2를 이용한 쿠버네티스 클러스터 구성
### 1. EC2 인스턴스 생성
- 외부에서 접속을 했을 때는 일정 시간 동안 입력이 없으면 자동으로 세션이 해제
- Master Node
  - CPU가 2개 이상: Core가 1개이면 설치는 성공을 하지만 마스터 노드로 실행하려고 할 때 에러가 발생
  - 포트 개방
    - API Server: 6443
    - etcd Server: 2379, 2380
    - kubelet API: 10250
    - kube scheduler: 10251
    - kube controller manager: 10252
    - Flannel CNI 플러그인에 대한 포트: 8285, 8472
- Worker Node
  - 하드웨어 제약 없음
  - 포트 개방
    - API Server: 6443
    - kube-proxy가 서비스를 로드밸런싱하기 위해서 사용하는 포트: 26443
    - Flannel CNI 플러그인에 대한 포트: 8285, 8472
    - NodePort 로 사용할 포트: 30000-32767

### 2. Ubuntu에 쿠버네티스 설치: https://hostnextra.com/learn/tutorials/how-to-install-kubernetes-k8s-on-ubuntu
```
패키지 정보 업데이트: sudo apt update && sudo apt upgrade -y
런타임 설치: sudo apt install -y docker.io
런타임 확인: sudo docker --version
Docker version 24.0.7, build 24.0.7-0ubuntu4.1

- 쿠버네티스 저장소 등록
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

- 쿠버네티스 클러스터 구성을 위한 패키지 설치
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

- 패키지의 용도
kubeadm: 클러스터 구성을 위한 패키지
kubelet: 노드 에이전트(Pod 생성, 실행, 모니터링)
kubectl: 명령 수행 도구(CLI)

- 버전 고정
sudo apt-mark hold kubelet kubeadm kubectl

- Memory Swap 비활성화: 가상 메모리 사용으로 인한 문제점 제거
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
- 확인: sudo free -m 과 sudo swapon -s

- 마스터 노드에서 클러스터 생성
  kubeadm을 초기화하고 파드의 네트워크 대역을 설정
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

- 아래 내용이 출력되는데 이 내용이 워커노드에서 마스터 노드에 연결하기 위한 코드
kubeadm join 172.31.10.135:6443 --token qtaw4i.n7eif2q1bnif1h28 \
        --discovery-token-ca-cert-hash sha256:018ed69a6a7c85b864dfceedc2ecfbfa586017e49b705788724caaf7bfe28785

- (Optional) Configure kubectl for a Non-Root User
sudo -i
mkdir -p /home/<your-username>/.kube
cp -i /etc/kubernetes/admin.conf /home/<your-username>/.kube/config
chown <your-username>:<your-username> /home/<your-username>/.kube/config
exit

- 쿠버네티스 환경 설정 파일 생성
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

- 노드 확인
kubectl get nodes

- 방화벽 중지
sudo systemctl stop ufw
sudo systemctl disable ufw
```
- Cloud 서비스는 기본적으로 포트에 대한 보안을 별도로 설정하도록 되어 있기 때문에 보안 그룹에 가서 인바운드 규칙에 추가를 해주어야 함
```
=> 토큰 확인
kubeadm token list

=>토큰 생성
kubeadm token create

=>해시값 확인
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt

=>포트번호 확인
kubectl cluster-info
```

### 3. 워커 노드에서 마스터 노드에 연결
```
kubeadm join 172.31.10.135:6443 --token qtaw4i.n7eif2q1bnif1h28 \
        --discovery-token-ca-cert-hash sha256:018ed69a6a7c85b864dfceedc2ecfbfa586017e49b705788724caaf7bfe28785
```
- 교재 소스: https://github.com/gilbutITbook/kiamol

## 스케일링
- 성능을 높이거나 낮추는 작업
- Scale Up
  - 하드웨어의 성능을 높이는 것
  - 초기 단계의 서비스나 하드웨어 업그레이드 전략으로 충분히 성능 향상이 가능한 경우에 적합한 전략
  - 확장의 한계에 도달했을 때 추가적인 성능 향상이 어렵기 때문에 장기적으로 사용하는 서비스에는 부적합
  - 쿠버네티스의 매니페스트에서 resource의 제한을 가할 수 있는데 이 부분을 수정하면 scale up과 유사한 효과를 나타낼 수 있음
- Scale Out
  - 컴퓨터의 대수를 늘리는 전략
  - 서비스 성장에 따라 서버를 추가로 확장할 수 있음
  - 하나의 서버에 문제가 발생하더라도 다른 서버가 역할을 대신 수행해줌
  - 서버의 수가 증가하게 되면 네트워크가 복잡해지고 관리 비용이 증가
  - 쿠버네티스에서는 replica를 조정해서 동일한 서비스를 하는 pod의 개수를 늘리는 것

### replicaset
- pod를 여러 개 만들 수 있는 객체
- pod 보다는 상위의 개념인데 deployment 보다는 하위의 개념, 최근에는 replicaset을 잘 사용하지 않음
- replicaset 이나 deployment로 만들어진 pod는 삭제되거나 중지되면 바로 다른 pod가 만들어짐

### scale 명령
- pod가 구동중에 동적으로 replica를 변경해야 할 때 사용
- 재배포를 하게 되면 무효가 됨
```
kubectl scale --replicas=파드개수 유형/이름
```
- 실습
  - ch06으로 이동
  ```
  kubectl apply -f pi/web/ # 파일 경로 대신에 디렉토리 경로를 사용하게 되면 디렉토리 안의 모든 yaml 파일을 수행
  ```
  - 파드의 개수(kubectl get pod)를 확인: 2개
  - 파드의 개수를 3개로 증가시키기
  - `kubectl scale --replicas=3 deploy/pi-web`
  - 파드의 개수(kubectl get pods)를 확인: 3개
  - 다시 배포 `kubectl apply -f pi/web/`
  - 파드의 개수를 확인: 2개로 복귀

### DaemonSet
- replicas를 설정하지 않더라도 모든 워커 노드에 파드를 생성해주는 객체
- 모니터링 용도

### garbage collection
- 쿠버네티스는 가비지 컬렉션이 있어서 주기적으로 감시를 하다가 자신의 상위 객체가 존재하지 않으면 자동으로 제거가 됨

### 파드 강제 삭제
- `kubectl delete pods 파드이름 --grace-period=0 --force`

## 멀티 컨테이너 파드
- 하나의 파드에 여러 개의 컨테이너를 실행
- 이런 패턴을 사이드카 패턴이라고 함
- 애플리케이션 컨테이너와 애플리케이션 컨테이너가 필요로 하는 다른 컨테이너를 같이 배치
- 대표적인 경우가 wordpress 같은 애플리케이션은 반드시 mysql이나 mariadb가 있어야 하는데 이 경우 서로 다른 파드로 구성해도 되고 하나의 파드에 다른 컨테이너로 묶어도 됨

### 파드와 컨테이너 통신
- 파드는 하나의 가상 환경
- 파드는 하나의 리눅스 시스템이 존재하고 그 안에 네트워크 그리고 파일 시스템을 제공하는 하나의 파드에 속한 컨테이너들은 이를 공유
- 하나의 파드에 속한 모든 컨테이너는 동일한 IP를 가지게 됨
- 하나의 파드에 속한 모든 컨테이너는 동일한 IP를 가지기 때문에 통신을 할 때 localhost를 사용할 수 있음
- 하나의 파드에 속한 컨테이너들은 내부 볼륨(hostPath)을 공유할 수 있음
- 하나의 메모리 공간을 공유하는 컨테이너
