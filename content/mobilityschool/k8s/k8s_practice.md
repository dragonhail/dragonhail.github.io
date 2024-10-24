---
title: 쿠버네티스 실습
weight: 2
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
  - Windows 홈 버전에서는 Hyper-V가 설치되지 않아서 설치가 안될 수 있는데 이 경우 아래 내용을 .bat 파일로 만들어서 관리자 권한으로 실행하면 Hyper-V 기능을 사용할 수 있게 됩니다. 그 이후 리눅스에서 minikube를 설치
  ```
  pushd "%~dp0"
  dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
  for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
  del hyper-v.txt
  Dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
  pause
  ```
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
- 컨테이너 실행
```
kubectl run 컨테이너이름 --image 이미지이름

kubectl run mynginx --image nginx
```
- 실행 중인 컨테이너(pod) 조회
  ```
  kubectl get pod
  ```
  - 상태
  ```
  Pending: 생성 명령 O 실행은 X
  ContainerCreating: 생성 중
  Running: 정상 실행 중
  Completed: 한 번 실행하고 완료된 상태
  Error: 에러
  CrashLoopBackOff: 지속적인 에러 상태로 인해 crash가 반복 중
  ```
  - 자세히보기
  ```
  kubectl get pod 파드이름 -o yaml

  kubectl get pod 파드이름 -o wide
  ```
  - event 기록까지 확인 가능한 자세히 보기:디버깅 할 때 사용
  ```
  kubectl describe pod 파드이름
  ```
- pod 로깅
  ```
  kubectl logs -f 파드이름
  ```
- pod에 명령을 수행
  ```
  kubectl exec 파드이름 -- 명령
  kubectl exec mynginx -- apt-get update
  ```
  - 파드 내부로 들어가고자 하는 경우
  ```
  kubectl exec -it 파드이름 -- bash
  ```
- 파드 와 호스트간 파일 복사
  ```
  kubectl cp 타겟 소스
  ```
  - 파드 안의 파일을 표현할 때 <파드이름>:>경로
  - 호스트의 /etc/password 파일을 mynginx 컨테이너의 /tmp/passwd로 복사하고자 하는 경우
  ```
  kubectl cp /etc/password mynginx:/tmp/passwd

  $ sudo touch /etc/password

  $ ls /etc/password
  /etc/password

  $ kubectl cp /etc/password mynginx:/tmp/passwd

  $ kubectl exec -it mynginx -- bash

  # ls /tmp
  passwd
  ```
- 파드 정보 수정
  ```
  kubectl edit pod 파드이름
  ```
- 파드 삭제
  ```
  kubectl delete pod 파드이름
  ```
- yaml 파일을 이용해서 파드를 생성
  ```
  kubectl apply -f 야믈파일경로
  ```
  - yaml 파일을 생성(mynginx.yml)
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
spec:
  containers:
    - name: mynginx
      image: nginx
```
  - `kubectl apply -f mynginx.xml`
- yaml을 권장하는 이유
  - 로컬 시스템에 위치한 YAML 정의서 뿐 아니라 인터넷 상에 위치한 YAML 파일도 사용 가능
  - 멱등성을 보장

### Pod
- 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위
- 하나 이상의 컨테이너 그룹
- 스토리지 및 네트워크를 공유하고 함께 배치됨
- Pod는 직접 생성하지 않는 경우가 많고 대부분의 경우는 워크로드 리소스를 사용해서 생성
- 파드의 단계
  - pending: 파드가 쿠버네티스 클러스터에 승인되었지만 하나 이상의 컨테이너가 설정되지 않았고 실행할 준비가 되지 않음, 파드가 스케쥴되기 이전까지의 시간 뿐만 아니라 네트워크를 통한 컨테이너 이미지 다운로드 시간도 포함
  - running: 파드가 노드에 바인딩 되었고 모든 컨테이너가 생성되었으며 적어도 하나의 컨테이너가 아직 실행중이거나 시작 또는 재시작 상태에 있는 것
  - Succeedd: 파드에 있는 모든 컨테이너들이 성공적으로 종료되었고 재시작되지 않음
  - failed: 파드에 있는 모든 컨테이너가 종료되었고 적어도 하나 이상의 컨테이너가 실패로 종료된 경우인데 해당 컨테이너는 non-zero 상태로 빠져나왔거나(exit) 시스템에서 의해서 종료된 것(terminated)
  - unknown: 파드의 상태를 얻을 수 없는 상태로 대부분은 다른 노드와의 통신 오류
- 컨테이너 재시작 정책: restartPolicy
  - Always: 항상 재시작
  - OnFailure: 실패한 경우 재시작
  - Never: 재시작 하지 않음
- 동일한 노드에서 kubelet에 의한 컨테이너 재시작 정책인데 파드의 컨테이너가 문제가 발생해서 종료되는 경우 5분 동안 지수 백오프 지연(10초, 20초, 40초, 80초...)으로 컨테이너를 재시작

### 리소스를 이용한 파드 생성
- `kubectl create` 나 `kubectl apply` 명령으로 생성
  - httpd 이미지를 이용해서 deployment로 파드를 생성
  - `kubectl create deployment my-httpd --image=httpd --replicas=1 --port=80`
  - stateless와 stateful
    - stateless(상태가 없음): 사용자가 애플리케이션을 사용할 때 상태나 세션을 저장할 필요가 없는 애플리케이션에 사용
    - stateful(상태가 있음): 사용자가 애플리케이션을 사용할 때 상태나 세션을 별도의 데이터베이스에 저장해야 하는 애플리케이션에 사용
  - deployment 를 확인
    - `kubectl get deployment`
    - READY: 레플리카의 개수
    - UP-TO-DATE: 최신 상태로 업데이트 된 레플리카의 개수
    - AVAILABLE: 사용 가능한 레플리카의 개수
    - AGE: 파드가 실행하고 있는 지속 시간
  - 이미지 변경
    - `kubectl set image deployment/디풀로이먼트이름 컨테이너이름=이미지`

## 디플로이먼트와 서비스
### Deployment
- stateless한 애플리케이션을 배포할 때 사용
- 레플리카 셋의 상위 개념으로 파드의 개수를 유지할 뿐 아니라 배포 작업을 좀 더 세분화해서 관리할 수 있음
- Pod < ReplicaSet < Deployment
- **배포 전략**
  - **롤링 업데이트**: 새 버전의 애플리케이션을 배포할 때 새 버전의 애플리케이션은 하나씩 늘려가고 기존 버전의 애플리케이션은 하나씩 줄여나가는 방식
    - 쿠버네티스의 표준 배포 방식
    - 새로운 버전으로 배포된 파드에 문제가 발생하면 이전 버전의 파드로 서비스를 대체할 수 있어서 상당히 안정적이지만 업데이트가 느리게 이루어지는 단점이 있음
    - 옵션 maxSurge(업데이트 주에 만들 수 있는 파드의 최대 개수)
    - 옵션 maxUnavailable(업데이트 중에 사용할 수 없는 파드의 개수로 0보다 큰 정수를 지정할 수 있음)
  ```yml
  spec:
    replicas: 3
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
  ```
  - **recreate(재생성)**
    - 이전 버전의 파드를 모두 종료하고, 새 버전의 파드로 일괄 교체하는 방식
    - 빠르게 교체할 수는 있지만 새로운 버전의 파드에 문제가 발생하면 대처가 늦어질 수 있다는 단점이 있음
  ```yml
  spec:
    replicas: 3
    strategy:
      type: Recreate
  ```
  - **blue/green**
    - 애플리케이션의 이전 버전(블루) 과 새로운 버전(그린)이 동시에 운영되는 구조
    - 새로운 버전만 서비스 목적으로 사용 가능하고 그린은 테스트 목적
    - 새로운 버전에 문제가 있으면 바로 이전 버전으로 대체가 가능하지만 파드의 개수가 늘어남
    - 버전으로 구분
  ```yml
  spec:
    replicas: 3
    version: v1.0.0
  ```
  - **canary**
    - 블루/그린 과 비슷하지만 조금 더 진보적인 방식
    - 애플리케이션의 기능 테스트를 할 때 사용하는 방식
    - 두 개의 버전을 모두 배포하지만 새 버전에는 조금씩 트래픽을 증가(5% -> 10%... 50%)시키면서 새로운 버전의 기능을 테스트
    - 기능 테스트가 끝나고 문제가 없다고 판단되면 이전 버전은 모두 종료시키고 새 버전으로만 서비스
    - version을 이용해서 배포