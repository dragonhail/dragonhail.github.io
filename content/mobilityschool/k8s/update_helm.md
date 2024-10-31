---
title: 업데이트 & Helm
weight: 10
---

0. 삭제명령
- `kubectl delete 객체종류 객체id`
- 모든 객체 삭제: `kubectl delete all --all`
- `--all` 대신에 `-n 네임스페이스(레이블)`을 설정할 수 있음
- pod의 경우는 종료되었지만 삭제가 되지 않는 경우가 있는데 이 경우는 강제 삭제를 이용
  - `kubectl delete pod 파드ID --grace-period=0 --force`
  - `kubectl delete pod --all --force`

## 업데이트
### 전략
- 롤링 업데이트
  - 정해진 비율만큼의 파드만 점진적으로 배포해 나가는 방식
  - 최대로 생성할 수 있는 파드의 개수와 최대로 삭제할 수 있는 파드의 개수를 옵션으로 설정해서 업데이트

- 블루/그린 업데이트
  - 버전을 구성해서 이전 버전과 새로운 버전에 해당하는 파드를 전부 생성해두고 트래픽을 새로운 버전으로 전달하는 방식

- 카나리 업데이트
  - 새로운 버전의 일부만 배포해두고 트래픽도 일부만 새로운 버전으로 전달하고 배포에 문제가 없으면 점진적으로 새로운 버전을 배포하거나 트래픽을 전환하는 방식

### 쿠버네티스의 롤링 업데이트
- 디플로이먼트의 정의를 수정해서 배포를 하게되면 롤링 업데이트를 수행
  - 레플리카의 수만 조정한 경우에는 업데이트(rollout)가 발생하지 않음
  - rollout: 배포를 변경하는 것
- 이미지를 변경한다던가 다른 설정을 변경을 해서 배포를 하면 쿠버네티스는 새로운 버전의 파드를 만들고 이 새로운 버전의 파드가 replica 개수만큼 생성이 되면 이전 버전의 replica를 0으로 만들어서 업데이트를 완료함

- 레플리카의 개수를 늘린 경우
  - 프롬프트를 ch09로 이동
  - vweb 디렉터리에 있는 deployment와 service를 배포
    - `kubectl apply -f vweb/`
  -  vweb/update/vweb-v1-scale.yaml 파일을 이용해서 deploy를 재배포: 이 경우는 replicas만 수정한 경우
     -  이런 경우는 파드를 확인해보면 기존 파드가 그대로 존재하고 파드의 개수를 늘리거나 줄이는 방식
     -  이 경우는 롤아웃 된 것이 아님
- 변경된 내역을 확인
  - `kubectl rollout history 객체종류/객체이름`
    ```bash
    ubuntu@ip-172-31-10-135:~/kiamol/ch09$ kubectl rollout history deploy/vweb
    deployment.apps/vweb
    REVISION  CHANGE-CAUSE
    1         <none>
    ```
    - 수정은 했지만 실제 변경이 이루어진 것이 없기 때문에 배포에 사용한 것만 등록
- 이미지를 변경해서 배포
  - nano vweb/vweb-v1.yaml 명령을 수행해서 이미지를 수정
  - `image: kiamol/ch09-vweb:v2`
  - 다시 배포
    - `kubectl apply -f vweb/`
  - 파드 확인: 기존의 3개 파드가 모두 중지되고 새로운 파드 2개가 생성됨
    ```bash
    ubuntu@ip-172-31-10-135:~/kiamol/ch09$ kubectl get pod
    NAME                    READY   STATUS        RESTARTS   AGE
    vweb-7d7496cdc6-2r46m   1/1     Terminating   0          17m
    vweb-7d7496cdc6-dcfgs   1/1     Terminating   0          29m
    vweb-7d7496cdc6-p2rfv   1/1     Terminating   0          29m
    vweb-7dd797bdcc-sgvz2   1/1     Running       0          7m
    vweb-7dd797bdcc-sxjg4   1/1     Running       0          7m5s
    ```
  - 변경된 내역 확인: REVISION이 새로 추가됨
    ```bash
    ubuntu@ip-172-31-10-135:~/kiamol/ch09$ kubectl rollout history deploy/vweb
    deployment.apps/vweb
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```

### 롤아웃 과 롤백
```
쿠버네티스 업데이트를 위해 사용한 이미지가 이미지이름과 태그가 똑같을 경우 쿠버네티스는 배포를 하지 않음, tag를 해쉬로 붙여서 업데이트 해야
- Image가 변경이 되면 업데이트
- replicas 변경: 업데이트X
- image이름을 그대로 업데이트 하려면 labels의 version 업데이트 수정
```
- 이미지의 태그를 변경해서 업데이트를 수행할 수 있고 replicas를 제외한 Pod의 내용을 변경해서 업데이트를 하는 것이 가능<br>
레이블이나 컨테이너의 이미지 부분을 제외한 부분을 변경하는 것은 다른 객체와의 연관성 문제 때문에 함부로 하는 것은 위험<br>
이 때 컨테이너에 영향을 주지 않고 내용을 변경할 수 있는 항목으로 version이 있음
- 버전만 변경해서 deployment를 변경
  ```
  - 배포
  kubectl apply -f vweb/update/vweb-v11.yaml

  - 버전만 변경했기 때문에 pod가 정의가 수정된 것이라서 변경이 발생하며 변경이 발생하면 하위 객체가 수정됨
  - Deployment의 경우 하위에 있는 replicaset이 수정되는데 이전 replicaset은 삭제되는 것이 아니고 pod의 개수가 0으로 수정되고 기존의 pod들은 terminated 됨
  - 변경된 내역 확인
  kubectl rollout history deploy/vweb

  - replicaset과 REVISION의 매칭 관계 확인
  kubectl get replicaset -l app=vweb -o=custom-columns=NAME:.metadata.name,REPLICAS:.status.replicas,REVISION:.metadata.annotations.deployment\.kubernetes\.io/revision

  - deployment 업데이트
  kubectl apply -f vweb/update/vweb-v2.yaml

  - 업데이트 된 내역 확인
  kubectl rollout history deploy/vweb

  - 롤백
  kubectl rollout undo 이름(Deployment, DaemonSet, StatefulSet) --to-revision=리비전번호
  kubectl rollout undo deploy/vweb --to-revision=2

  - 롤백을 수행하면 ReplicaSet은 재사용되고 Pod는 새로 만들어짐
  ```
- 환경에 관련된 내용을 별도의 파일에 만들어 두고 사용하는 ConfigMap의 경우 이 내용의 변경은 rollout이 아님<br>
변수의 값이 바뀌는 것은 Pod의 설정 내용을 바꾸는 것이 아니기 때문
- 기존 vweb이라는 deploy를 삭제

### 롤링 업데이트 설정
- Deployment는 2가지 업데이트 전략을 제공하는데 하나는 Rolling Update 이고 다른 하는 Recreate
- Rolling Update는 새로운 파드를 만들어서 성공하면 이전 버전을 제거해나가는 방식인데 업데이트해야 하는 파드의 내용이 많을 때 업데이트하는데 시간이 오래 걸림
- Recreate 전략은 기존의 파드를 제거하고 새로운 파드로 교체하는 방식으로 기존 Deployment의 replicas를 0으로 먼저 설정하고 파드를 생성해나가는 방식 
- Rolling Update는 새로운 파드를 만들고 제거해 나가므로 새로운 파드에 문제가 발생하면 이전 파드를 가지고 계속 서비스를 해 나갈 수 있지만 Recreate는 새로운 버전에 문제가 발생하면 서비스가 중단<br>
Recreate는 빠르게 업데이트가 가능하다는 장점이 있음<br>
간단한 애플리케이션이고 테스트가 충분히 이루어졌다면 Recreate 전략을 사용하는 것도 가능
- Recreate 전략을 설정하는 방법은 spec에서 strategy의 type 속성에 Recreate 만 사용하면 됨
  - Recreate는 기존파드를 먼저 다 죽이고 새 파드 생성, 중단 발생 가능
  - 롤링업데이트는 새파드 하나 생성 성공하면 기존 파드 하나 삭제 방식으로 업데이트, 두개 버전이 공존할 수 있음
- Rolling Update의 스케일링 속도를 위한 옵션
  - maxUnavailable<br>
  업데이트 하는 동안 사용할 수 없는 파드의 개수<br>
  기존 레플리카셋에서 동시에 종료되는 파드의 개수<br>
  정수로 설정해도 되고 백분율로 설정해도 됨
  - maxSurge<br>
  업데이트 할 때 만들어지는 새로운 버전의 최대 개수
  - 이 설정을 하고 업데이트를 수행하게 되면 기존 개수 + maxSurge만큼 **증가한 후** 기존의 파드를 제거해 나가게 됨
  - 이를 적절히 이용하면 생성 후 삭제, 삭제 후 생성, 동시 생성 및 삭제 등 가능
- 업데이트 예시
  - replica: 3
  - maxSurge를 1로 설정하고 maxUnavailable을 0으로 설정하는 것이 default 값인데 3+1의 상태로 만들고 새로운 버전이 정상적으로 동작을 하면 하나를 제거해 나가는 방식(생성 후 삭제)
  - maxSurge를 0으로 maxUnavailable를 1로 하면 1개가 제거되고 1개를 생성하는 구조(삭제 후 생성)
  - maxSurge를 1로 하고 maxUnavailable를 1로 설정하면 1개가 제거되고 2개를 생성하는 구조
- 업데이트 속도 조절 옵션: 파드가 생성이 되고 일정 시간이 지난 후에 교체하기 위해서
  - minReadySeconds<br>
  신규 파드의 상태가 안정적인지 확인할 수 있는 시간 여유를 두는 옵션인데 기본값이 9
  - progressDeadlineSeconds<br>
  이 시간동안 파드가 만들어지지 않으면 생성 실패로 간주하는 것으로 이 시간의 기본값은 600
- Deployment를 만들 때 minReadySeconds는 반드시 설정하라고 권장

### DaemonSet 과 StatefulSet의 업데이트 전략
- 종류
  - RollingUpdate: 기존의 Deployment 와 동일, 업데이트가 바로 수행
  - OnDelete: 삭제하면 업데이트가 수행되는 것, 사용자가 원하는 시점에 업데이트가 수행

## Helm
- 쿠버네티스는 파드를 생성하는 Deployment, ReplicaSet, Pod, StatefulSet, DaemonSet, Job, CronJob 만으로 구성할 수 있지만 Service나 ConfigMap 그리고 Secret, Claim 등과 함께 구성되는 경우가 많음
- 이렇게 여러 개의 객체를 별도로 관리하게 되면 관리가 어려움
- 다양한 리소스를 각각 관리하지 않고 하나의 패키지로 관리하는 도구가 Helm
- 리눅스에서 yum, apt나 Python의 pip, Node의 npm, yarn 등과 유사한 개념
- 많은 프레임워크 개발 업체들이 쿠버네티스 환경에서 Helm을 사용해 애플리케이션을 설치하도록 Helm 파일을 제공

### 헬름의 주요 구성 요소
- 헬름 차트
  - 애플리케이션 설치에 사용되는 네트워크, 스토리지, 보안과 관련된 여러 쿠버네티스 리소스를 묶어놓은 패키지
  - 애플리케이션을 설치할 때 쿠버네티스에 하나의 디렉토리에 관련 파일을 묶어놓고 디렉토리를 실행하듯이 헬름 차트를 이용해서 한 번에 설치하게 됨
  - 디렉터리 구성<br>
  Chart.yaml: 차트에 대한 정보가 담긴 파일<br>
  LICENSE: 라이선스 정보가 담긴 텍스트 파일<br>
  README.md: 설명<br>
  values.yaml: 차트의 기본 템플릿 변수 파일<br>
  charts/: 차트에 종속된 차트들을 포함하는 디렉터리<br>
  crds/: 커스텀 자원<br>
  templates/: 유효한 쿠버네티스 매니페스트 파일<br>
  templates/NOTES.txt: 차트 사용법<br>

- 헬름 레포지토리
  - 헬름 차트를 저장하고 있는 장소
  - 솔루션 업체들이 직접 헬름 레포지토리를 제공하는데 없는 경우에는 ArtifactHub에서 찾을 수 있음
  - 여러 원격 레포지토리를 등록해서 사용하는 것이 가능
- 헬름 템플릿
  - 별도의 파일에 값을 저장하고 다른 파일에서 사용하는 기능<br>
  values.yaml 파일에 키와 값 형태로 데이터를 저장하고 다른 파일에서 가져다가 사용
  - values.yaml
  ```yml {filename="values.yaml"}
  favoriteDrink: coffee
  ```
  - manifest.yaml
  ```yml {filename="manifest.yaml"}
  apiVersion: v1
  kind: ConfigMap
  metadata:
   name: {{ .Values.favoriteDrink }}
  ```
### 헬름 차트를 이용한 nginx 웹 서버를 설치
- 과정
  - 애플리케이션 설치에 사용되는 헬름 레포지토리를 등록
  - 헬름 차트를 다운로드
  - 애플리케이션을 설치
  - 필요한 경우 재배포
  - 헬름 차트를 삭제

### 헬름 설치
- 다운로드
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```
  - 스크립트 파일을 다운로드 받아서 bash 바로 명령문을 수행해서 /usr/local/bin/helm 에 설치
  - 삭제를 하고자 하면 설치된 디렉토리의 파일을 삭제하면 됨
- 버전 확인
  - `helm version`

### 주요 명령어
- `helm repo add`: 애플리케이션 설치에 사용되는 헬름 레포지토리를 로컬 환경에 추가
- `helm pull`: 헬름 차트 파일을 로컬 호스트로 내려 받음
- `cp 파일경로 파일경로`: 앞에 해당하는 파일을 뒤의 경로로 복사
- `helm install {helm 이름} -f 환경설정파일`
- `helm ls`: 설치된 헬름 차트 목록을 출력
- `helm get manifest`: 현재 실행 중인 헬름 차트의 전체 YAML 파일 목록 및 설정을 확인
- `helm upgrade`: 재배포
- `helm delete`: 헬름 차트 삭제

### nginx 설치
```
=>레포지토리 등록
helm repo add bitnami https://chars.bitnami.com/bitnami

=>로컬 레포지토리의 캐시를 업데이트
helm repo update

=>nginx 헬름 차트 검색
helm search repo nginx

=>nginx 헬름 차트 다운로드
helm pull bitnami/nginx

=>압축된 파일이 다운로드 되었으므로 압축을 해제
tar xvfz nginx-18.2.4.tgz

=>원본 파일 삭제
rm -rf nginx-18.2.4.tgz

=>버전 관리를 하고자 하는 경우 디렉토리 이름을 수정
mv nginx nginx-18.2.4

=>압축을 해제한 디렉토리
templates 디렉토리에 쿠버네티스 관련 파일이 만들어지고 values.yaml 파일에 환경 변수를 기재하는데 이 파일의 내용을 변경해서 배포를 합니다.

=>압축을 해제한 디렉토리에서 실행
helm install nginx -f values.yaml .

=>만들어진 helm 삭제
helm delete nginx

=>설정을 변경해서 설치
values.yaml 파일을 복사
cp values.yaml my-values.yaml

my-values.yaml 에서 replicaCount를 3으로 수정

압축을 해제한 디렉토리에서 실행
helm install nginx -f my-values.yaml .
```

### kiamol.net 레포지토리의 kiamol/vweb을 설치

- 레포지토리 등록
```
helm repo add kiamol https://kiamol.net
```

- 레포지토리 업데이트(로컬 레포지토리 캐시 업데이트)
```
helm repo update
```

- vweb 헬름 차트 검색
```
helm search repo vweb
```

- 환경 변수를 values.yaml 파일에 기록해두기도 하지만 기본값을 변경해서 설치하기도 합니다.

- 기본값 확인
```
helm show values kiamol/vweb
```

- 기본값을 수정해서 배포
```
helm install --set servicePort=8010 --set replicaCount=1 ch10-web kiamol/vweb --version 1.0.0
```

- 만들어진 디플로이먼트 와 레플리카셋을 확인
```
kubectl get deploy,replicaset
```
  - 기존의 헬름을 업데이트 할 때는 helm upgrade 명령으로 수행

- 헬름을 사용하는 경우는 replicas 만 변경해도 리비전이 만들어집니다.

- 로컬에 있는 헬름 차트를 설치할 때는 `install` 대신에 `lint` 명령 사용이 가능<br>
`install`은 로컬과 레포지토리 모두 설치가 가능하고 `lint`는 로컬에 있는 것만 검증 가능
- 로컬에 있는 web-ping 디렉터리가 헬름 차트가 될 수 있는지 확인
```
helm lint web-ping
```
- 설치는 동일

### 레포지토리 생성
- 원격 레포지토리
  - Google Cloud Storage
  - Git Hub Page
  - Harbor OCI
  - 일반 웹 서버: yaml파일과 tgz파일만 서빙할 수 있다면 가능
  - ChartMuseum 서버
  - AWS의 S3

- 로컬 레포지토리
  - ChartMuseum를 이용해서 구현
- ChartMuseum를 이용해서 구현하는데 public ip가 있다면 원격 레포지토리로 사용이 가능하고 public ip가 없다면 로컬 레포지토리가 됨
  - 헬름을 이용하기 위해서 저장소 추가
  ```
  helm repo add stable https://charts.helm.sh/stable
  ```
  - 업데이트
  ```
  helm repo update
  ```
  - 저장소를 만들 수 있는 ChartMuseum을 설치
  ```
  helm install --set service.type=LoadBalancer --set service.externalPort=8008 --set env.open.DISABLE_API=false repo stable/chartmuseum --version 2.14.2
  ```
  - URL 확인
  ```
  kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalalncer.ingress[0].*}:8008'
  ```
  - 등록
  ```
  helm repo add local $(kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalalncer.ingress[0].*}:8008')
  ```
  - 업로드<br>
    패키징: `helm package web-ping`

### 의존성 설정
- docker-compose를 사용하는 이유
  - 여러 개의 컨테이너가 의존 관계 형태로 묶여 있는 경우 순차적으로 컨테이너를 생성해야 함
  - 직접 컨테이너를 생성하게 되면 실수를 할 수 있고 자동화 할 수가 없음
- helm도 의존성 있는 차트를 하위로 설정을 해서 하위 차트가 전부 실행이 되면 상위 차트를 생성하도록 설정할 수 있음
- chart.yaml 파일에 dependencies 라는 속성을 이용해서 기재
```yaml
dependencies:
  - name: 이름
    version: 버전
    repository: 레포지토리이름
```

### 헬름의 업그레이드 와 롤백
- 헬름은 쿠버네티스 와 상관없이 업그레이드 와 롤백이 가능
  - 기존 헬름의 목록을 확인 `helm ls`

### 헬름을 사용해야 하는 경우
- 마이크로 서비스의 개수가 많이 늘어나면 배포와 관련된 별도의 팀을 두는 것이 일반적
- 마이크로 서비스를 개발하는 팀이 직접 배포를 수행하는 경우 이 마이크로 서비스와 연관된 서비스가 있으면 배포에 혼란이 올 수 있음<br>
같이 수정해서 배포해야 하는 경우 순서대로 배포하지 않으면 에러가 발생할 수도 있음<br>
이런 경우 헬름을 도입하면 여러 개의 YAML 파일을 하나로 합쳐서 적은 수의 YAML 파일로 배포가 가능
- 여러 개의 쿠버네티스 파드를 배포하다 보면 동일한 설정값을 사용하는 경우가 있는데 Helm을 이용하면 하나의 values.yaml 파일에 만들고 이를 재사용할 수 있음