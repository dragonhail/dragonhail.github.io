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
- 하나의 메모리 공간을 공유하는 컨테이너(ch07/sleep)
```
파드 생성
kubectl apply -f sleep-with-file-reader.yaml

볼륨에 파일 기록: HOSTNAME이라는 환경 변수에 저장된 내용을 data-rw/hostname.txt 파일에 기록

 kubectl exec deploy/sleep  -c sleep -- sh -c 'echo ${HOSTNAME} > data-rw/hostname.txt'

자신이 저장한 파일 읽기
 kubectl exec deploy/sleep  -c sleep -- cat data-rw/hostname.txt

- 파드 내에 컨테이너 간의 통신
파드 생성
kubectl apply -f sleep-with-server.yaml

sleep 이라는 컨테이너에서 server라는 컨테이너의 8080 포트에 요청을 전송
kubectl exec deploy/sleep -c sleep -- wget -q -O - localhost:8080

확인: server 컨테이너의 로그 확인
kubectl logs -l app=sleep -c server
```
### 초기화 컨테이너
- 사이드카 패턴의 컨테이너들은 컨테이너가 만들어지는 순서가 없음
순차적으로 실행되어야 하는 컨테이너들을 사이트카 패턴으로 만들게 되면 필요한 컨테이너가 구동되지 않았음에도 컨테이너가 생성되서 오류를 발생시킬 수 있습니다.
- 이런 경우에는 초기화 컨테이너를 이용해서 먼저 생성되어야 하는 컨테이너를 초기화 컨테이너로 설정하고 이 컨테이너가 정상적으로 생성된 경우에 다음 컨테이너를 생성하도록 해줄 수 있음
- 초기화 컨테이너는 여러 개 설정할 수 있고 작성한 순서대로 동작
- 리눅스 명령어의 `;`는 사이드카, `&&`는 초기화 컨테이너와 유사
- 초기화 컨테이너 설정
  - 일반 컨테이너는 spec 안에 containers 라는 속성으로 설정하는데 초기화 컨테이너는 initContainers 라는 속성으로 설정하며 배열 형태이며 순차적으로 생성성
- 예제
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
    spec:
      initContainers:	#여기에 기재된 컨테이너를 먼저 생성
				#여기에 설정하는 컨테이너들은 순차적으로 생성됨
        - name: init-html
          image: kiamol/ch03-sleep
          command: ['sh', '-c', "echo '<!DOCTYPE html><html><body><h1>KIAMOL Ch07</h1></body></html>' > >
          volumeMounts:
            - name: data
              mountPath: /data
      containers:
        - name: sleep
          image: kiamol/ch03-sleep
        - name: server
          image: kiamol/ch03-sleep
          command: ['sh', '-c', 'while true; do echo -e "HTTP/1.1 200 OK\nContent-Type: text/html\nConte>
          ports:
            - containerPort: 8080
              name: http
          volumeMounts:
            - name: data
              mountPath: /data-ro
              readOnly: true
      volumes:
        - name: data
          emptyDir: {}
```

### 어댑터 컨테이너
- 애플리케이션이 어떤 내용을 스스로의 파일에 기록을 하게 되면 외부에서는 이 내용을 확인할 수 없음
- hostPath 와 같은 노드 컴퓨터의 특정 디렉토리와 볼륨 마운트를 하면 가능은 함
- 애플리케이션이 로그를 출력하면 그 출력한 로그를 읽어서 표준 출력으로 다시 연결하는 방식을 이용해서 로그를 외부로 출력할 수 있음
- 이렇게 자신의 어떤 로직을 수행하는 것이 아니고 다른 컨테이너가 수행한 작업의 내역을 가지고 다른 컨테이너에게 전송을 하거나 외부에 출력해주는 역할을 수행하는 컨테이너를 어댑터 컨테이너라고 함
- 헬스 체크나 성능 지표를 측정하는 애플리케이션을 이러한 방식으로 배포하는 경우가 있는데 최근에는 이를 위한 프로메테우스 나 ELK Stack 잘 만들어져 있어서 어댑터 컨테이너 형태보다는 별도의 애플리케이션을 주로 이용

## 데이터를 많이 다루는 애플리케이션
### Stateful Set
- stateful: 상태를 저장하는 것
- stateless: 상태를 저장하지 않는 것
- http나 https는 stateless
  - web에서 이전 상태를 확인하기 위해서는 별도의 기술이 필요
  - web에서는 이전 상태를 server에 저장해 둘 수 있고 client에 저장하기도 함
  - server에 저장하는 것이 session이고 session을 저장하는 기술로는 메모리에 저장하는 것 그리고 파일이나 데이터베이스에 저장하는 것이 있음
  - 메모리에 저장하는 경우 많은 클라이언트가 접속하는 경우 서버의 처리가 느려지기 때문에 속도가 빠른 In Memory Database를 사용하는 경우가 많음
  - 클라이언트에 저장하는 기술로는 Cookie, Local Storage, Web SQL 등이 있음
- pod는 기본적으로 stateless
- stateful 형태의 파드를 만드는 방법은 kind를 StatefulSet으로 설정하고 serviceName을 설정해야 함
- 파드를 stateless 형태(ReplicaSet, Deployment)로 replica를 3이라고 하면 3개의 파드를 동시에 생성할려고 함<br>
이 경우는 데이터를 저장할 필요가 없기 때문에 병렬로 생성해도 아무런 문제가 되지 않음<br>
데이터베이스를 이런식으로 배포하게 되면 3개의 데이터베이스의 불일치가 발생할 수 있음<br>
이름도 랜덤하게 만들어짐
- 파드를 stateful 형태(StatefulSet)로 replica를 3이라고 하면 3개의 파드를 동시에 생성하는 것이 아니고 하나의 파드를 만들고 이 파드를 만드는 데 성공하면 이 내용을 복사해서 다음 파드를 생성<br>
이렇게 만들면 파드의 이름도 순차적으로 부여
- 데이터베이스를 배포할 때 이 방식을 취하게 됨
  - 첫번째 파드는 읽고 쓰기가 가능하도록 하고 두번째 부터는 읽기만 가능하고 첫번째 파드의 데이터를 동기화하는 역할만 수행
- StatefulSet 사용
  - 파일 구성
  - ch08/todo-list/db/todo-db-secret.yaml: 환경 설정 파일로 데이터베이스 비밀번호로 사용할 값을 가지고 있음
```yml
apiVersion: v1
kind: Secret
metadata:
  name: todo-db-secret
  labels:
    kiamol: ch08
type: Opaque
stringData:
  POSTGRES_PASSWORD: "kiamol-2*2*"
```

  - ch08/todo-list/db/todo-db-service.yaml: 서비스 설정 파일로 데이터베이스 포트를 외부에서 사용할 수 있도록 개방하는 내용
```yml
apiVersion: v1
kind: Service
metadata:
  name: todo-db
  labels:
    kiamol: ch08
spec:
  ports:
    - port: 5432
      targetPort: 5432
      name: postgres
  selector:
    app: todo-db
  clusterIP: None
```
- ch08/todo-list/db/todo-db-service.yaml: Stateful 한 Pod 생성
```yml
apiVersion: apps/v1
kind: StatefulSet # StatefulSet을 생성: Pod가 순차적으로 생성
metadata:
  name: todo-db
  labels:
    kiamol: ch08
spec:
  selector:
    matchLabels:
      app: todo-db
  serviceName: todo-db
  replicas: 2
  template:
    metadata:
      labels:
        app: todo-db
    spec:
      containers:
        - name: db
          image: postgres:11.6-alpine
          env:
          - name: POSTGRES_PASSWORD_FILE
            value: /secrets/postgres_password
          volumeMounts:
            - name: secret
              mountPath: "/secrets"
      volumes:
        - name: secret
          secret:
            secretName: todo-db-secret
            defaultMode: 0400
            items:
            - key: POSTGRES_PASSWORD
              path: postgres_password
```
  - 파드 생성(ch08 디렉토리에서 실행)
  ```
  kubectl apply -f todo-list/db/
  ```
  - 파드 확인
  ```
  kubectl get pods -o wide
  ```
    - 파드의 이름이 -0, -1 이렇게 순차적으로 만들어 짐
    - 하나의 파드가 만들어지고 다음 파드가 만들어짐
    - 파드를 삭제하면 새로운 파드가 만들어질 때 이전 파드 이름으로 만들어짐
    - 웹의 forwarding과 유사(redirect는 이전값 기억x)
- deploy로 배포하면 pod을 삭제하면 pod이 다시 생성되면서 이름이 달라짐
```
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
pi-web-cfcbc4f49-fqlrr   1/1     Running   0          12m     10.244.1.7   ip-172-31-33-74   <none>           <none> <- 이 pod 삭제
pi-web-cfcbc4f49-mhh27   1/1     Running   0          12m     10.244.2.6   ip-172-31-38-88   <none>           <none>
todo-db-0                1/1     Running   0          8m1s    10.244.2.7   ip-172-31-38-88   <none>           <none>
todo-db-1                1/1     Running   0          7m52s   10.244.1.8   ip-172-31-33-74   <none>           <none>
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl delete pods pi-web-cfcbc4f49-fqlrr --grace-period=0 --force
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pi-web-cfcbc4f49-fqlrr" force deleted
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
pi-web-cfcbc4f49-mhh27   1/1     Running   0          15m
pi-web-cfcbc4f49-swkq6   1/1     Running   0          3s <- pod 이름이 달라짐
todo-db-0                1/1     Running   0          10m
todo-db-1                1/1     Running   0          10m
```
- StatefulSet으로 배포한 pods 삭제 시 똑같은 이름의 pod이 다시 생성
```
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
pi-web-cfcbc4f49-mhh27   1/1     Running   0          15m   10.244.2.6   ip-172-31-38-88   <none>           <none>
pi-web-cfcbc4f49-swkq6   1/1     Running   0          17s   10.244.1.9   ip-172-31-33-74   <none>           <none>
todo-db-0                1/1     Running   0          10m   10.244.2.7   ip-172-31-38-88   <none>           <none>
todo-db-1                1/1     Running   0          10m   10.244.1.8   ip-172-31-33-74   <none>           <none>
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl delete pods todo-db-0 --grace-period=0 --force
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "todo-db-0" force deleted
ubuntu@ip-172-31-10-135:~/kiamol/ch08$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
pi-web-cfcbc4f49-mhh27   1/1     Running   0          18m     10.244.2.6   ip-172-31-38-88   <none>           <none>
pi-web-cfcbc4f49-swkq6   1/1     Running   0          3m36s   10.244.1.9   ip-172-31-33-74   <none>           <none>
todo-db-0                1/1     Running   0          2s      10.244.2.8   ip-172-31-38-88   <none>           <none>
todo-db-1                1/1     Running   0          13m     10.244.1.8   ip-172-31-33-74   <none>           <none>
```
### 초기화 컨테이너 사용 가능
- 특정 스크립트 파일을 만들고 변수의 값을 0으로 초기화를 하고 값이 0일 때 특정 작업을 수행하고 1을 증가시키게 되면 이 스크립트는 첫번째 컨테이너가 만들어질 때 와 다른 컨테이너가 만들어질 때 다른 작업을 수행하도록 할 수 있음
- 데이터 동기화
  - 애플리케이션이 구동될 때 데이터를 복제, 작업 시간 별 로깅
  - 애플리케이션에 작업이 수행 할 때 양쪽에 작업 수행, 작업 시간을 같이 기록

### Job과 CronJob의 활용
- 데이터를 활용하는 애플리케이션의 데이터의 백업은 중요한 작업인데 이를 수동으로 하게되면 실수를 할 수 있음, 이러한 작업은 자동으로 수행되도록 해주는 것이 좋음
- 이렇게 특정 작업을 일정한 주기를 가지고 작업을 할 수 있도록 해주는 것이 CronJob 임
- Job은 한 번 수행하면 그대로 종료됨
  - Job은 배치 작업에 사용함
  - 모아서 한번에 많은 내용을 수행하고자 할 때 사용
- CronJob은 주기를 정해서 주기 단위로 작업을 계속 수행하기 위해서 생성
```
EC2 2개에 mysql 설치, 외부에서 접속해보기
db 복사
파이썬으로 실행해보기
크론잡 만들어보기
```