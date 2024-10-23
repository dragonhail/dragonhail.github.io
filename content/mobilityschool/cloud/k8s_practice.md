---
title: 쿠버네티스 실습
weight: 15
---
## 실습환경
### 쿠버네티스 전체 버전을 설치할 수 있는 환경
- 운영체제
  - 리눅스 환경
  - Ubuntu, Debian, Cent OS, Red Hat Enterprise, Fedora, 컨테이너 리눅스 까지 상관없음
- 하드웨어
  - CPU(Core) 가 2개 이상

### 쿠버네티스가 사용하는 포트
- Master Node 가 사용하는 포트
  - 6443: API Server
  - 2379, 2380: etcd Client API
  - 10250: kubelet API
  - 10251: Scheduler
  - 10252: Controller Manager
- Worker Node 가 사용하는 포트
  - 10250: kubelet API
  - 30000~32767: NodePort Service

### 설치 방법
- 전체 설치
  - 마스터 와 워커를 다른 컴퓨터에 설치
  - 가상머신을 이용해서 하나의 컴퓨터에서 마스터 와 워커 노드를 분리해서 설치
- 경량 버전 설치: 운영체제와 상관없이 설치 가능
  - k3s: 마스터와 워커 1개씩 설정, 제약이 많음 - linux에서 스크립트를 다운로드 받아 실행:  
`curl -sfL https://get.k3s.io | sh -`

  - minikube: 마스터와 워커 1개씩 설정 https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download
  - kind: 마스터 1개와 워커 여러개 가능한데 Docker를 런타임으로 사용
  - docker-desktop: 쿠버네티스 설치 옵션을 체크하면 사용 가능

- 설치 후 확인
```bash
sudo kubectl run mynginx --image nginx

sudo kubectl get pod
```

## 쿠버네티스 사용
### yaml 
- 쿠버네티스에서는 pod를 만들 때 명령어 만으로 생성할 수 있고 yaml 파일을 생성하는 것이 가능
- 작성 요령
  - 하나의 블록에 속하는 엔트리마다 -를 붙여야 합니다.
  - 키 값 매핑은 : 으로 합니다
  - 문서의 시작 과 끝에 — 를 삽입할 수 있습니다.
  - 키 와 값 사이에 공백이 있어야 합니다.
  - 주석은 #으로 시작
  - 들여쓰기는 tab 이 아니라 공백
- 검증: https://onlineyamltools.com/validate-yaml

### 파드를 생성하고 관리
- 파드는 쿠버네티스의 기본 배포 단위이면서 다수의 컨테이너를 포함
- 일반적으로는 하나의 파드에 하나의 컨테이너가 포함되지만 하나의 컨테이너에 2개 이상의 컨테이너를 포함시킬 수 있는데 이를 sidecar pattern 이라고 함
- 파드 생성
  - create 또는 apply 명령을 사용하는데 create는 새로운 리소스를 생성하는 것이고 apply는 create 와 replace 의 혼합
  - httpd 라는 이미지를 가지고 pod를 생성: yaml 파일 없이 생성
  ```
  kubectl run 파드이름 --image 이미지이름
  ```
  - pod 확인 `kubectl get pod`