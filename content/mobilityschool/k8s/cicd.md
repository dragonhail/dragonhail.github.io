---
title: CI/CD & Kind
weight: 6
---
## 쿠버네티스 보안
### 쿠버네티스 강화 가이드
- 취약점이 있거나 구성이 잘못된 컨테이너 및 파드를 점검할 수 있어야 함
- 최소한의 권한으로 컨테이너와 파드를 실행해야 함
- 네트워크를 분리해 다른 시스템에 영향을 주지 않도록 해야 함
- 방화벽을 사용해 불필요한 네트워크 연결을 차단해야 함
  - 실제 쿠버네티스 설치에 대한 검색을 해보면 포트를 하나 하나 해제하는 것이 번거로워서 방화벽 자체를 무력화 시키고 작업하는 경우가 있음
- 강력한 인증 및 권한 부여를 사용해 사용자 및 관리자 접근을 제한해야 함
- 관리자가 모든 활동을 모니터링 하면서 잠재적/악의적 활동에 대응할 수 있도록 주기적으로 로그 감사를 해야 함
- 정기적으로 모든 쿠버네티스 설정을 검토하고 취약점 스캔 및 보안 패치가 적용되어 있는지 확인해야 함


### RBAC(Role-Based Access Control)
- 역할을 기반으로 쿠버네티스에 접속할 수 있는 권한을 관리
- user와 role 두 가지를 조합해서 사용자에게 권한을 부여
- Role의 예시
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 namespace: rbac
 name: role
rules:
    - apiGroups: [""] # 역할이 적용될 그룹
      resources: ["pods"] # 어떤 자원에 접근할 것인지
      verbs: ["get"] # 어떤 동작을 할 수 있는지
```
- 역할 바인딩
```
apiGroups: 사용자, 그룹, 서비스계정
resource: node, pod, service, deployment, configmap, secret
행위: Get, Create, Watch, Delete, Patch, List
```

## CI/CD
- CI(Continuous Integration)
  - 지속적 통합
  - 오류 수정 혹은 새로운 기능을 테스트하고 깃허브 같은 저장소에 배포하는 과정
  - 핵심
    - 빠른 개발과 배포
    - 자동화: 소스 코드를 수정했으면 빌드하고 테스트를 해야하는데 이 과정을 CI가 자동화함
- CD(Continuous Delivery 또는 Deployment)
  - 지속적 배포
  - 테스트까지 완료된 소스 코드를 개발 환경과 운영 환경에 배포하는 것
  - Continuous Delivery: 사람의 개입이 필요한 수동 배포
  - Continuous Deployment: 모든 과정이 자동화
- CI: 소스 코드 작성 -> 빌드 -> 테스트
- CD: CI -> 개발환경에 배포 -> 운영환경에 배포

### 필요성
- 프로그램 빌드의 자동화 그리고 자동화된 테스트
- CI Tool: Travis CI, Bamboo, Jenkins 등
- CD Tool: Argo CD

### 동작 과정
- 개발자 -> 소스코드를 수정해서 GitHub에 업로드를 하고 이 때 Web Hook(Git Action) Trigger를 이용해서 Jenkins에 보내면 Jenkins가 빌드를 수행하고 이 결과를 이미지로 만들어서 CI/CD Pipeline을 통해서 Docker Hub에 보내고 Docker Hub의 이미지를 쿠버네티스에 배포
- 운영자->매니페스트파일을 만들어서 GitOps Repository에 업로드를 하고 개발자가 소스 코드를 수정해서 Jenkins가 빌드한 도커 이미지 정보를 업데이트해서 GitOps Repository 내용과 ArgoCD에 작성한 내용이 다르면 쿠버네티스에 배포

## Jenkins
- 지속적 통합 도구
- 다수의 개발자가 개발한 소스 코드를 커밋/빌드하고 개발 환경에 배포하는 일 등을 자동화함
- 장점
  - 프로젝트 환경에서 오류 검출
  - 테스트를 자동으로 수행
  - 코딩 규약 준수 여부 체크
  - 소스 변경에 따른 성능 변화 감시
  - 개발자의 소스 통합 및 배포

### 설치
- 패키지 업데이트: `sudo apt update`
- 자바 실행 환경 설치: `sudo apt install -y openjdk-17-jre-headless`
- 리눅스에서 설치하려는 패키지가 기본 레포지토리에 없는 경우 `add-apt-repository` 명령어를 이용해서 레포지토리를 추가할 수 있는데 이 명령어를 사용하면 설치하려는 소프트웨어에 대한 접근 권한을 부여
  - 이런 유사한 역할을 수행해주는 것이 GPG 인데 GPG는 디지털 암호화 및 서명 서비스를 제공하는 OpenPGP
  - 특정 소프트웨어들은 패키지에 접근하기 위해서 GPG 키를 추가하기를 권장
```bash
wget -q -0 - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg

sudo sh -c ‘echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list’

sudo apt update

sudo apt install jenkins
```

## kind와 kubectl을 설치
### kind
- 쿠버네티스 클러스터를 간단하게 구성하기 위한 애플리케이션
- 런타임이 Docker
- 하나의 컴퓨터에서 별도의 가상 머신 없이도 여러 개의 노드를 구성할 수 있음

### kubectl
- 쿠버네티스 클러스터에 명령을 내리기 위한 도구

### 설치
- minikube의 클러스터가 동작 중이면 중지
```bash
minikube stop

minikube delete --all
```
- minikube 에 별명이 설정되어 있으면 삭제
```bash
unalias minikube
```
- Docker 는 미리 설치가 되어 있어야 함
- kubectl 설치
  - https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
  ```bash
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

  curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

  echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

  kubectl version --client
  ```
#### kind(kubernetes in docker) 설치
- 문서: https://kind.sigs.k8s.io/docs/user/quick-start/

- 명령어
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
- 확인
  - 기본 클러스터(Master 1개 와 Worker 1개) 생성
  - kind create cluster
  - 클러스터 삭제
  - kind delete cluster
  - 여러 개의 노드 생성
  - yaml 파일 작성
  ```yml {filename="kind-example-config.yml"}
	kind: Cluster
	apiVersion: kind.x-k8s.io/v1alpha4
	nodes:
	- role: control-plane
	- role: worker
	- role: worker
  ```
```bash
#클러스터 생성
kind create cluster --config yaml파일경로

#클러스터 확인
docker ps
```


## 쿠버네티스에서 리눅스 기본 요소 사용
### 파드 실행을 위한 전제 조건
- 쿠버네티스에서 사용하는 리눅스의 프로그램
  - swapoff: CPU 와 메모리 설정을 존중하는 방식으로 쿠버네티스를 실행하기 위한 전체 조건인 메모리 스와핑(실제 메모리가 부족할 때 디스크 공간을 이용해 부족한 메모리를 대체할 수 있는 기술 - 가상 메모리)을 비 활성화
  - iptables: 네트워크 프록시에 대한 핵심 요구사항, pod로 보내는 iptables 규칙을 생성
  - mount: 경로의 특정 위치에 리소스를 연결, 디바이스를 홈 디렉토리에 디렉토리로 노출
  - systemd: 모든 컨테이너를 관리하기 위해 실행되는 핵심 프로세스인 kubelet을 시작
  - nsenter: 네트워킹, 스토리지 또는 프로세스 측면에서 다양한 네임스페이스로 들어가 진행 상태를 확인할 수 있는 도구
  - unshare: 프로세스가 네트워크, 마운트 또는 PID 관점에서 격리되어 실행되는 하위 프로세스를 만들 수 있게 해주는 명령
  - ps: 실행 중인 프로그램을 나열하는 명령인데 kubelet 이 프로세스가 실행 중인지 아니면 종료되었는지를 계속 확인
- 파드를 생성
  - yml 파일을 생성
```yml {filename="pod.yml"}
apiVersion: v1

kind: Pod

metadata:
  name: core-k8s
  labels:
    role: just-an-example
    app: my-example-app
    organization: friends-of-manning
    creator: adam

spec:
  containers:
    - name: any-lod-name-will-do
      image: docker.io/busybox:latest
      command: ['sleep', '10000']
      ports:
      - name: webapp-port
        containerPort: 80
        protocol: TCP
```
  - 파드 생성
  ```bash
  kubectl create -f pod.yml
  ```
  - 파드가 리눅스 내부적으로 프로세스로 실행되는 것을 확인
  ```bash
  # 리눅스에서 현재 실행 중인 프로세스 개수 확인
  ps -ax | wc -l
  ```
### 의존성 탐색
- 파드는 다음 관점에서 일반적인 프로그램
  - 키보드 입력, 파일 나열 등을 사용할 수 있게 해주는 공유 라이브러리나 OS에 특화된 저수준 유틸리티를 사용
  - 네트워크 호출이나 수신이 될 수 있도록 TCP/IP 스택의 구현 과 함께 동작할 수 있는 클라이언트에 엑세스
  - 다른 프로그램이 자신의 메모리를 덮어쓰지 않도록 보장하기 위한 일정의 메모리 주소 공간이 필요
- 파드를 생성할 때 kubelet이 수행하는 동작
  - 프로그램이 실행되기 위한 격리된 홈(CPU, Memory, Namespace 제한을 갖는)을 생성
  - 홈이 이더넷과 연결이 되는지 확인
  - DNS를 확인하거나 스토리지에 엑세스 하기 위한 일부 기본 파일에 대한 엑세스 권한을 프로그램에게 부여
  - 프로그램에게 다른 파드로 이동해서 시작하는 것이 안전하다고 알려줌
  - 프로그램이 종료될 때 까지 대기
  - 프로그램이 종료되면 사용한 공간 과 자원을 정리
- kubelet은 리눅스의 시스템 관리자의 역할

```
사용자가 kubectl명령 -> k8s -> kubelet -> linux
kubeadm: 클러스터 만들어주는 기능
```
- 파드에 대한 상세 정보 출력: `kubectl get pods -o yaml`
- 원하는 정보를 출력하고자 할 때 jsonpath 옵션을 이용
```bash
# 파드 상태에 대한 쿼리
$ kubectl get pods -o=jsonpath='{.items[0].status.phase}'

# 파드의 IP 주소 확인
$ kubectl get pods -o=jsonpath='{.items[0].status.podIP}'
	
# 호스트 컴퓨터의 IP 주소 확인
$ kubectl get pods -o=jsonpath='{.items[0].status.hostIP}'
```
- 파드에 마운트 한 데이터 검사
  ```
  # 리눅스에서 수행:
  $ kubectl exec -it core-k8s -- sh

  # 컨테이너 내부 쉘에서 수행
  mount | grep resolv.conf
  /dev/sda2 on /etc/resolv.conf type ext4 (rw,relatime)
  ```
  - 실제 마운트 된 호스트 컴퓨터의 볼륨을 의미하며 이 내용은 실제로는 /var/lib/containerd 의 디렉토리에 위치한 내용