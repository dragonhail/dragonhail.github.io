---
title: Kubernetes의 Linux사용
weight: 7
---
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

## 처음부터 파드 만들기
### chroot를 사용해 격리 프로세스 생성
- 기본적인 컨테이너는 bash 쉘을 실행하는데 필요한 것 이외에 전혀 다른 것이 없는 디렉토리
- 과정
  - 실행할 프로그램과 프로그램이 실행되어야 할 파일 시스템의 위치를 결정
  - 프로세스가 실행할 환경을 만드는데 lib64 디렉토리에는 여러 리눅스 프로그램이 있는데 이것들이 Bash 와 같은 프로그램을 실행하기 하기 위해 필요하므로 이들을 새로운 루트로 불러와야 함
  - 실행하고 싶은 프로그램을 chroot 처리된 위치로 복사
- 명령어
```sh {filename="chmod0.sh"}
# 명령어 수행 위한 bin
sudo mkdir /home/namespace/box/bin
# bin 위한 라이브러리
sudo mkdir /home/namespace/box/lib
sudo mkdir /home/namespace/box/lib64
sudo mkdir /home/namespace/box/usr

sudo cp -r /lib/* /home/namespace/box/lib
sudo cp -r /lib64/* /home/namespace/box/lib64
sudo cp -r /usr/* /home/namespace/box/usr

sudo mkdir /home/namespace/box/proc
sudo mount -t proc proc /home/namespace/box/proc

sudo chroot /home/namespace/box /bin/bash
```
- sh 파일에 실행 권한 부여
  - `sudo chmod +x 파일경로`
- sh 파일 실행
  - ./chroot0.sh: 파일을 복사하다가 용량이 부족해서 에러가 발생

### 프로세스 데이터 제공
- 컨테이너는 클라우드나 호스트 시스템 같은 다른 곳에 있는 스토리지를 엑세스
- mount 명령을 수행하면 디바이스를 OS 의 루트 디렉토리 아래에 있는 모든 디렉토리에 노출하는 것이 가능
- `mount /tmp /home/namespace/box/data`
  ```bash
  # tmp 디렉토리 내용을 확인
  ls /tmp

  # 파일이 없으면 가상의 파일을 생성
  touch a

  # 디렉토리 생성
  mkdir /home/namespace/box/data

  # Mount
  sudo mount --bind /tmp/ /home/namespace/box/data

  # 확인: tmp 디렉토리의 내용이 아래 디렉토리에 복사
  ls /home/namespace/box/data
  ```
- kubernetes에서 hostpath를 이용한 볼륨 마운트 방식인데 이렇게 하는 것은 누구나 /tmp 내용을 조작하거나 읽을 수 있음
- 이러한 이유 때문에 운영환경에서는 hostPath 기능이 종종 비활성화됨

### unshare를 이용한 프로세스 보안
- chroot를 이용하면 격리된 프로세스를 만들 수 있음
- chroot를 사용하면 기본적으로 현재 쉘의 하위 프로세스로 만들어짐
- 현재 실행한 쉘에서 프로세스를 죽일 수 있음, 상위 프로세스에서는 하위 프로세스를 킬 할 수 있음
- 새로 실행되는 프로세스의 PID를 1로 해서 다른 곳에 이 프로세스를 킬 할 수 없도록 해야 함
```
unshare -p -f -mount-proc=/home/namespace/box/proc chroot /home/namespace/box /bin/bash
```
  - 이 명령을 수행하면 이 명령으로 생성된 프로세스는 자신의 PID가 1이라고 생각
  - 컨테이너 내부에서 exec로 실행한 프로세스들은 자신의 하위 프로세스로 간주

### 네트워크 네임스페이스 생성
- chroot 로 실행한 것 과 쿠버네티스의 pod 그리고 도커의 컨테이너의 차이점은 chroot 와 unshare 명령으로 프로세스를 생성하게 되면 모든 프로세스가 하나의 네트워크(eth0) 생성되는데 다른 네트워크를 갖도록 프로세스를 만들고자 할 때는  `unshare -p -f -n -mount-proc=/home/namespace/box/proc chroot /home/namespace/box /bin/bash` 명령으로 프로세스를 실행
- `-n` 옵션을 추가하면 eth0 하위에 훨씬 더 많은 동작중인 네트워크가 만들어짐
- 쿠버네티스 파드의 네트워크 인터페이지 확인
  - `ip a`

### cgroup을 이용한 CPU 조정
- container를 만들 때 아래처럼 작성하게 되면 메모리 사용량이 제한됨
```yml
resources:
  limits:
    memory: "200Mi"
  requests:
   memory: "100Mi"
```
- 리소스 제한을 두게 되면 리눅스에서는 /sys/fs/cgroup/리소스이름/chroot로 실행한 이름/여기에 제한된 자원에 대한 내용이 설정
```bash
sudo ls /sys/fs/cgroup/
sudo mkdir /sys/fs/cgroup/box
sudo ls /sys/fs/cgroup/box
```

### 실제 파드
- 앞의 명령어들로 pod와 유사한 프로세스를 만들 수 있지만 현실적으로 위의 기능만 가지고 pod를 직접 생성하는 것은 어려움
- 마이크로서비스는 다른 많은 서비스와 통신해야 할 수도 있고 해당 서비스에 새로운 인증서를 마운트해야 하기도 하고 DNS를 이용해서 다른 서비스를 탐색도 해야함
- 대규모 환경에서 마이크로서비스를 관리하기 위한 쿠버네티스 모델의 최대 장점은 내부 DNS를 사용한다는 것
- 모든 쿠버네티스 컨테이너는 통신을 위해 다음 사항이 필요
  - 클러스터 내부나 파드에서 파드사이의 연결을 위해 직접 라우팅 된 트래픽
  - 른 파드나 인터넷에 액세스하기 위한 라우팅된 트래픽
  - 정적 IP 주소를 사용하는 서비스 뒤에서 엔드포인트 역할을 위한 로드 밸런싱된 트래픽
- 위와 같은 작업을 허용하려면 쿠버네티스의 다른 부분에 게시되어야 하는 파드의 메타데이터가 필요: API Server 가 하는 일
- 시간이 지나면서 상태가 업데이트되고 채워지도록 이들의 상태를 계속해서 모니터링을 해야함: kubelet이 하는 일
- 파드는 컨테이너 명령과 도커 이미지보다 많을 것을 설정해야함
- 파드는 상태가 게시되는 방법에 따라 label 과 spec을 갖게 됨
- 이 label을 가지고 IP 주소 와 DNS 규칙이 최신 상태로 유지되는 것을 보장
- 파드의 레이블이 잘 정의된 상태와 재시작 로직을 가지며 클러스터 내 내에서 IP 주소의 도달 가능성을 보장
- iptables를 이용한 kube-proxy의 쿠버네티스 서비스 구현 방법
  - 서비스: 이 IP에 액세스하면 여러 엔드포인트 중 서비스가 가능한 하나로 자동으로 전달시켜주는 객체
  - 실제 이러한 네트워킹 규칙은 iptables 프로그램을 사용해 저수준 네트워크 라우팅을 수행해주는 kube-proxy에 의해 완전하게 구현
  - 파드를 만들면 아래와 구문이 수행
  ```
  iptables -A INPUT -s 파드의IP -j DROP
  # 들어오는 모든 트래픽을 폐기
  ```
  - 쿠버네티스에서는 위의 명령문 이외의 기능을 포함
    - 서비스 엔드포인트로 트래픽을 받아들이는 능력
    - 자체 엔드포인트에서 외부로 트래픽을 보내는 능력
    - 진행 중인 TCP 연결을 추적하는 능력
  - 파드를 생성할 때는 실제로 실행되고 라우팅이 가능한 소프트웨어 정의 네트워크(SDN)이 필요하기 때문에 파드를 재사용하지 않음
  - 사용 가능한 모든 호스트 출력: `iptables-save` 
- kube-dns
  - 특징
    - 어떤 쿠버네티스 클러스터에서도 사용 가능
    - 특별한 권한을 가지지 않으며 호스트 네트워크가 아닌 일반적인 파드 네트워크를 사용
    - 트래픽을 DNs 포트 표준으로 널리 알려진 53번 포트로 전송
  - 기본적으로 어떤 트래픽도 받을 수 없음
  - 라우팅 규칙 확인: `ip route`
  - 서비스를 만들면 서비스에 해당하는 트래픽을 전송하면 End Point로 전송하기 위해서 라우팅 테이블과 DNS에 파드의 실제 위치와 매핑시켜 둠

## 파드 내 프로세스에서 cgroups 사용
### 프로세스와 스레드
- 쿠버네티스의 스케줄러의 PID를 확인
- `ps -ax | grep scheduler`

### 파드의 리소스 사용량 설정
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
    - image: nginx
      imagePullPolicy: Always
      name: nginx
      resources:
        requests:
          cpu: "1"
          memory: 1G
```

- 파드가 어디에 생성되었는지 확인 `kubectl get pods -o wide`
- 파드가 생성된 컴퓨터에 접속
  - docker ps 로 워커 노드의 컨테이너 ID를 확인
- 컨테이너에 접속 `docker exec -it 컨테이너ID /bin/sh`
- 자원 사용량 확인
  - `top`
- 사용량을 설정하지 않은 경우
  - yaml 파일을 수정
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
      resources:
        limits: # 실제로 사용할 수 있는 사이즈
          cpu: .1
        requests: # 이 정도 자원이 확보되어야 파드 정상 수행을 할 수 있다
          cpu: .1
      ports:
      - name: webapp-port
        containerPort: 80
        protocol: TCP
```
```
# pod 생성
kubectl apply -f pod.yml

# pod에 접속
kubectl exec -it core-k8s – /bin/sh

# 무한 작업 수행
dd if=/dev/zero of=/dev/null

# 다른 터미널에서 접속
# pod 가 생성된 노드를 확인
kubectl get pods -o wide

# 노드의 컨테이너ID 확인
docker ps

# 도커에 접속
docker exec -it 컨테이너ID /bin/bash

# 자원사용량 확인
top
```
- 리소스 사용량을 제한하지 않거나 제한을 하더라도 정수 단위로 설정하면 리소스 전체를 사용하는 일이 벌어질 수 있음
- pod 의 리소스 확인
```
kubectl get pods -o wide
```
### 쿠버네티스에서 스왑을 사용할 수 없는 이유
```
CPU <-> Memory(주기억) 실행 <-> Disk(보조기억) 저장
```
- 메모리 스왑을 이용하게 되면 유후 프로세스에서 메모리 할당이 갑자기 느려질 수 있음
- 성능이 심하게 가변적이 될 수 있음

### HugePages
- 쿠버네티스 초기에는 지원하지 않았던 개념
- 쿠버네티스가 초장기에는 웹 중심 기술로 발전했기 때문에 페이지의 크기를 4KB로 고정시켜 놓고 사용을 해도 아무런 문제가 없었음
- 최근에 쿠버네티를 데이터 센서 기술로 사용을 하기 시작하면서 페이지 크기를 조정할 필요가 생김 - Elastic Search 나 Cassandra 와 같은 데이터베이스를 쿠버네티스를 이용해서 배포하기 시작
- 리소스를 설정할 때
```yml
resources:
  limits:
    hugepages-2Mi: 100Mi
```
- 데이터 센터와 관련된 컨테이너를 만들 때는 매우 중요

### 리눅스 커널 모니터링
- 현실에서는 위와 같은 데이터를 수동으로 curating 하지 않음
- 일반적으로 시스템 매트릭과 전반적인 추세에 대해 컨테이너 시스템 레벨 OS 정보를 하나의 시계열 대시보드에 집계해 긴급 사항이 발생했을 때 문제의 시간 범위를 파악하고 다양한 관점에서 문제를 심층적으로 분석
- 클라우드 네이티브 애플리케이션 모니터링과 쿠버네티스 자체 모니터링을 위한 업계 표준인 프로메테우스를 많이 사용
- 프로메테우스를 사용했을 때의 이점
  - 클러스터 오버런을 발생시킬 수 있는 쿠버네티스가 볼 수 없는 은밀한 프로세스를 확인 가능
  - 클러스터가 OS와 상호작용하는 방식의 버그를 발견할 수 있는 커널 수준의 격리 도구를 사용해 쿠버네티스가 인식하는 리소스에 직접 매핑이 가능
  - kubelet 과 컨테이너 런타임을 통해 대규모 환경에서 컨테이너의 구현 방법을 자세하게 살펴볼수 있는 도구
- Metric(수량화 가능한 값) 유형
  - gauge: 특정 시간에 초당 얼마나 많은 요청을 수신하는지 표시
  - histogram: 다양한 유형의 이벤트를 시간 간격으로 표시
  - counter: 지속적으로 증가하는 이벤트의 개수