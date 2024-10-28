---
title: CI/CD
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