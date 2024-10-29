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