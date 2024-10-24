---
title: Deployment와 Service
weight: 3
---
## Deployment 와 Service 사용
### Deployment
- 파드 배포에 관한 객체
- 단순히 파드를 배포하는 것 뿐 아니라 몇 개의 파드를 실행할 지 결정하는 것도 가능
- Deployment를 이용해서 pod를 배포
  - nginx-deploy.yml 파일을 생성하고 작성
```yml {filename="nginx-deploy.yml"}
#API 버전
apiVersion: apps/v1

#객체 종류
kind: Deployment


#객체에 대한 정보를 생성
#labels 가 중요
# Deployment의 Pod은 lable 기반으로 생성, 바꾸면 Pod 전부 다시 생성됨, Deployment 간의 label 이름은 달라야 함
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 2    #2개의 파드 생성
  selector:      #Deployment 가 관리할 파드를 선택
    matchLabels:
      app: nginx
  template:      #이 정보를 가지고 pod를 생성
    metadata:
      labels:
        app: nginx

    spec:        #컨테이너 정보
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
  - yaml 파일의 정보를 가지고 파드를 생성
    - `kubectl apply -f nginx-deploy.yml`

  - 모든 파드 삭제
    - `kubectl delete pods --all`

  - 파드 확인
    - `kubectl get pods`
  - **Deployment로 배포한 replicas 에 설정한 파드의 개수를 유지할려고 하기 때문에 파드를 삭제하면 다시 생성해서 파드의 개수를 유지합니다. Pod로 만들면 삭제해도 복구가 되지 않습니다.**
- lables
  - 레이블이안 오브젝트에 키-값 쌍으로 리소스를 식별하고 속성을 지정하는데 사용을 하는데 일종의 카테고리라고 할 수 있습니다.
  - 아래 처럼 사용
  ```
  release: v1, v2
  environment: dev, production
  tier: frontend, backend
  app: webapp, middleware
  ```
  - 다른 객체와 연결을 할 때 label이 매핑이 되어야 합니다.
  - 모든 파드 이름 과 레이블 확인
    - `kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels`
  - Deployment 나 ReplicaSet은 파드를 직접적인 관계를 맺지 않고 자신의 레이블과 일치하는 파드가 개수만 맞게 존재하면 됩니다. 수동으로 레이블을 변경하게 되면 그 레이블에 맞게 파드가 생성되어야 된다고 판단
  - 레이블 변경
    - `kubectl label pods -l app=nginx --overwrite app=nginx-1`
  - 파드 확인: 파드가 2개 추가 된 것을 확인할 수 있습니다.
  ```bash
  dh@swarm-manager:~/1024$ kubectl get pod
  NAME                           READY   STATUS              RESTARTS   AGE
  nginx-deploy-75fd5cdd7-45gcg   0/1     ContainerCreating   0          8s
  nginx-deploy-75fd5cdd7-569fb   0/1     ImagePullBackOff    0          10m
  nginx-deploy-75fd5cdd7-rdgcw   0/1     ImagePullBackOff    0          10m
  nginx-deploy-75fd5cdd7-rx28s   0/1     ContainerCreating   0          7s
  dh@swarm-manager:~/1024$ kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels
  NAME                           LABELS
  nginx-deploy-75fd5cdd7-45gcg   map[app:nginx pod-template-hash:75fd5cdd7]
  nginx-deploy-75fd5cdd7-569fb   map[app:nginx-1 pod-template-hash:75fd5cdd7]
  nginx-deploy-75fd5cdd7-rdgcw   map[app:nginx-1 pod-template-hash:75fd5cdd7]
  nginx-deploy-75fd5cdd7-rx28s   map[app:nginx pod-template-hash:75fd5cdd7]
  ```
  - **Deployment 나 ReplicaSet은 pod를 직접 관리하지 않기 때문에 레이블이 변경되면 변경된 레이블 아래 pod가 존재해야 한다고 판단**
  - 레이블 변경 및 파드 확인
  ```bash
  dh@swarm-manager:~/1024$ kubectl label pods -l app=nginx-1 --overwrite app=nginx
  pod/nginx-deploy-75fd5cdd7-569fb labeled
  pod/nginx-deploy-75fd5cdd7-rdgcw labeled
  ```
  - 새로 만들어진 파드 2개가 소멸됨
  ```bash
  dh@swarm-manager:~/1024$ kubectl get pods 
  NAME                           READY   STATUS             RESTARTS   AGE
  nginx-deploy-75fd5cdd7-569fb   0/1     ImagePullBackOff   0          15m
  nginx-deploy-75fd5cdd7-rdgcw   0/1     ImagePullBackOff   0          15m
  ```
  - **맨 처음 nginx가 Deployment로 레이블을 생성해서 2개의 파드를 만들고 nginx-1로 labels를 수정하면 이 경우는 Deployment가 새로 만들어지는 것이 아니고 labels만 추가가 되서 Deployment 1개 와 새로운 labels 1개가 존재해서 총 4개의 파드가 존재하지만 변경한 labels를 nginx로 변경을 하면 labels에 nginx-1이 사라지기 때문에 다시 2개의 파드만 있으면 됨**
- Deployment가 자신의 내부 객체를 직접 관리하지 않고 이름을 통해서 연결되는 구조를 느슨한 결합이라고 함


### Service
- 기본적으로 파드는 같은 노드에 떠 있는 파드끼리만 통신이 가능
- 다수의 노드에 떠 있는 파드 간의 통신이나 외부와의 통신을 위해서는 CNI 플러그인이 필요한데 이 때 파드에 위치한 서비스를 외부에서 접속할려면 서비스를 이용해야 함

- 외부에서 파드에 접근하기 위해서는 포트포워딩 하는 방법이 있음
- `kubectl port-forward 파드이름 외부포트:내부포트`
  - nginx-deploy-54b9c68f67-7xxbd 파드의 80번 포트를 호스트의 8000번 포트로 연결하도록 port-forwarding을 수행
  ```bash
  kubectl port-forward nginx-deploy-54b9c68f67-7xxbd 8000:80
  ```
- 하나의 노드 안에 있는 파드끼리는 통신이 가능
- 파드 와 파드끼리 IP를 안다면 통신이 가능
  - 파드는 고정된 위치에 있는 것이 아니고 소멸되었다가 다른 곳에서 만들어지는 동적인 개념
  - 파드 간의 통신: IP 이용
  - 파드 IP 정보 확인 `kubectl describe 파드이름`
  - 파드 삭제: `kubectl delete pod 파드이름`
  - 파드 이름 확인 `kubectl get pod`
  - 파드 IP 정보 확인: `kubectl describe 파드이름`
  - 파드의 IP가 변경된 것을 확인
- 외부에서 서비스 이름으로 접근할 수 있도록 포트를 외부로 개방하기
  - 서비스를 생성하기 위한 nginx-svc.yml 파일을 생성하고 작성
```yml {filename="nginx-svc.yml"}
apiVersion: v1

kind: Service # Service를 생성하는 yaml 파일

metadata:
  name: nginx-svc # Service 이름
  labels:
    app: nginx

spec:
  type: NodePort # NodePort를 이용해서 외부에 공개
  ports:
  - port: 8080
    nodePort: 31472
    targetPort: 80
  selector:
    app: nginx # app:nginx를 갖는 파드와 연결
```
  - 서비스 생성 `kubectl apply -f nginx-svc.yml`
  - 서비스 생성 확인 `kubectl get svc`
- type
  - ClusterIP: 기본값으로 클러스터 내부의 다른 리소스들과 통신이 가능함. 이 때 IP 뿐 아니라 Service 이름으로도 통신이 가능
  - NodePort: 클러스터 외부에서 노드IP의 특정 포트로 들어오는 요청을 감지해서 해당 포트와 연결된 파드로 트래픽을 전달하는 것으로 이를 설정하면 ClusterIP도 자동으로 설정됨
  - LoadBalancer: LoadBalancer를 제공하는 Public Cloud와 연결할 때 사용하는데 NodePort와 ClusterIP는 자동 설정
  - ExternalName: selector 부분에 lables를 사용하지 않고 Domain Name을 직접 사용하고자 하는 경우 사용

### 롤백
- 디플로이먼트는 기본적으로 롤링 업데이트와 함께 롤백도 지원
- 롤백은 업데이트에 문제가 생겼을 때 이전 버전으로 바꿀 수 있는 기능
- 새로운 이미지로 변경은 `kubectl set image deployment` 명령을 사용
  - 기본 배포된 이미지를 nginx:1.16.1로 변경
    - `kubectl set image deployment.v1.apps/nginx-deploy nginx=nginx:1.16.1`
  - 확인 `kubectl describe deploy nginx-deploy` 
  - 존재하지 않는 버전으로 수정
    - `kubectl set image deployment/nginx-deploy nginx=nginx:1.200`
  - 현재 상황 확인
    - `kubectl rollout status deployment/nginx-deploy`
    - `kubectl get pods`
  - 기존의 2개의 파드는 유지가 되지만 업데이트하다가 이미지를 가져올 수 없어서 업데이트가 이루어지지 않고 ImageBackoff 상태
  - 이전 버전으로 희귀
    - `kubectl rollout undo deployment/nginx-deploy`
  - 업데이트 내역 확인
    - `kubectl rollout history deployment/nginx-deploy`
  - 특정 revision으로 롤백이 가능
    - `kubectl rollout undo 디플로이먼트 --to-revision=번호`

### ReplicaSet
- 일정한 개수의 동일한 파드가 항상 실행되도록 관리해주는 객체
- 필요한 이유는 서비스의 지속성 때문
- 노드의 하드웨어에서 발생항는 장애 등의 이유로 파드를 사용할 수 없을 때 다른 노드에서 다시 생성해서 사용자에게 중단없는 서비스를 제공할 수 있음
- yml 파일을 만들어서 실행
  - replicaset.yml 파일을 생성하고 작성
```yml {filename="replicaset.yml"}
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: 3-replicaset

spec:
  template:
    metadata:
      name: 3-replicaset
      labels:
        app: 3-replicaset
    spec:
      containers:
      - name: 3-replicaset
        image: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      app: 3-replicaset
```
  - 적용 `kubectl apply -f replicaset.yml`
  - 스케일 변경: `scale` 명령과 `--replicas=개수` 옵션을 이용
  - 삭제는 delete 명령인데 --cascade=orphan 옵션을 이용하면 pod는 유지되고 replicaset 만 없어짐
    - `kubectl delete -f replicaset.yml --cascade=orphan`