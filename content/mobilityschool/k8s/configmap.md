---
title: ConfigMap과 Secret
weight: 4
---
- Kubenetes 객체가 사용하기 위한 데이터를 저장하기 위한 객체
- 환경변수 와 같은 데이터는 대부분 실행 중에는 변하지 않고 실행하기 직전에 변할 수 있는 문자열이나 외부로 노출되어서는 안되는 데이터
- ConfigMap 이나 Secrets 은 스스로 어떤 기능을 하지는 않고 적은 양의 데이터를 저장하는 것이 목적

### ConfigMap
- 문자열 상수를 이용해서 만들 수 있고 파일의 내용을 읽어서 만들 수 있음
- 생성형식
  - `kubectl create configmap <map-name> <data-source> <arguments>`
- 문자열 상수(리터럴)를 가지고 생성
  - `kubectl create configmap [map name] --from-literal=[키]=[값]`
```bash
kubectl create configmap my-config --from-literal=JAVA_HOME=/usr/java

kubectl delete configmap my-config

kubectl create configmap my-config --from-literal=JAVA_HOME=/usr/java --from-literal=URL=http://localhost:8000

kubectl get configmap my-config

kubectl describe configmap my-config
```
- 파일에서 읽어서 만들기
  - 데이터를 저장하는 yaml 파일 생성하고 작성
  ```bash
  echo Hello Config >> configmap_test.html

  kubectl create configmap configmap-file --from-file configmap_test.html
  ```
- 다른 yaml 파일에서 가져다 사용할 때는 최상단에 작성
```yml
envFrom:
    - configMapRef:
      name: ConfigMap 이름
```
- 이제부터 ConfigMap 이름에 해당하는 데이터를 사용할 때 key만 입력하면 됨

### secret 만들기
- kubectl create secret generic 시크릿이름 --from-literal=키=값...

## Volume
- 볼륨은 데이터를 보관하는 장소
- 데이터를 저장소에 저장해두는 애플리케이션을 stateful 이라고 하고 데이터를 저장소에 저장하지 않는 애플리케이션을 stateless 라고 합니다.
- stateless 애플리케이션은 Deployment 나 ReplicaSet 또는 Pod 만 배포하면 되지만 Stateful 애플리케이션은 대부분 영구 저장소를 별도로 만들어야 하기 때문에 stateless 애플리케이션보다 구성이 복잡
- 파드 내에 만들어진 데이터는 파드가 종료되면 소멸되는데 이 때 파드가 종료되더라도 데이터를 보관하기 위한 개념이 볼륨
- 데이터 보관 방법
  - 파드 내에 위치: 파드 내에 데이터를 저장하면 파드가 종료될 때 데이터도 함께 사라짐, emptyDir 볼륨
  - 워커 노드 내에 위치: 파드가 종료되어도 데이터가 유지되지만 노드가 종료되면 데이터도 함께 소멸, hostPath 볼륨
  - 노드 외부에 위치: 파드 혹은 노드 와 무관하게 데이터가 유지

### emptyDir
- 파드 내에 위치
- 파드가 생성될 때 같이 생성되고 파드가 삭제될 때 같이 삭제되는 임시 볼륨
- spec에서 volumes 속성을 이용하고 하위 속성으로 emptyDir 속성에 {}를 설정하면 되고 이를 containers 속성의 volumeMounts 속성의 mountPath에 연결해주면 됩니다.
- 실습
  - 볼륨 마운트를 위한 yaml 파일을 생성: 이미지는 nginx를 이용
```yml {filename="emptydir"}
apiVersion: v1
kind: Pod
metadata:
  name: emptydata

spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-storage
      mountPath: /data/shared
  
  volumes:
  - name: shared-storage
    emptyDir: {}
```
  - pod 생성 및 확인
  ```bash
  kubectl apply -f emptydir.yml

  # 접속
  kubectl exec -it emptydata -- /bin/sh
  
  # 확인
  cd /data/shared
  ```

### hostPath
- 노드의 로컬 디스크를 파드에 마운트해서 사용하는 로컬 볼륨
- 같은 hostPath를 사용하는 다수의 파드끼리 데이터를 공유해 사용할 수 있다는 장점이 있음
- 파드가 삭제되더라도 hostPath에 있는 파일들은 삭제되지 않고 남아 있으므로 같은 hostPath를 사용하는 다른 파드는 해당 볼륨에 접근해서 파일을 사용할 수 있음
- 실습
  - hostpath를 이용하는 Pod 생성을 위한 yaml 파일을 작성: hostpath.yml
```yml {filename="hostpath.yml"}
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: localpath
      mountPath: /data/shared

  volumes:
  - name: localpath
    hostPath:
      path: /tmp
      type: Directory
```

```
# 파드 생성
kubectl apply -f hostpath.yml

# 파드 내부로 접속
kubectl exec -it hostpath -- /bin/bash

# 볼륨과 연결된 디렉토리로 이동
cd /data/shared

# 파일을 생성
echo Hello "LocalVOLUME" > test.txt

# 파일 확인
ls -al

# pod를 지우고 다시 만든 후 디렉토리를 확인하면 이전에 만들었던 test.txt 가 존재
# 이 방식도 node 가 삭제되면 데이터는 보존되지 않습니다.
```
### 영구 볼륨 과 영구 볼륨 클레임
- 필요성
  - 임시 볼륨이나 로컬 볼륨은 파드나 노드가 종료되면 데이터가 삭제
  - 데이터를 반 영구적으로 저장을 하고자 할 때는 파드나 노드 내부가 아니라 외부에 저장을 해야 함
- 영구 볼륨(Persistent Volume, PV)
  - 컨테이너, 파드, 노드의 종료와 무관하게 데이터를 영구적으로 보관할 수 있는 볼륨
  - 영구 볼륨은 시스템 관리자가 외부 저장소에서 볼륨을 생성한 후 쿠버네티스 클러스터에 연결하는 것
  - 개발자가 디플로이먼트로 파드를 생성할 때 볼륨을 정의하는데 이 때 볼륨 자체를 정의하는 것이 아니라 볼륨을 요청하는 볼륨 클레임을 지정하고 그 다음 볼륨 클레임에서 볼륨을 할당
  - 영구 볼륨을 요청하는 볼륨 클레임을 영구 볼륨 클레임(Persistent Volume Claim, PVC)이라고 함
  - 개발자가 요청한 영구 볼륨은 API Server에게 보내지고 API Server를 통해서 영구 볼륨이 할당되는 구조임
- 영구 볼륨 실습
  - 영구 볼륨(kind가 PersistentVolume)을 정의: pv.yml
```yml {filename="pv.yml"}
apiVersion: v1

kind: PersistentVolume

metadata:
  name: mysql-pv-volume
  labels:
    type: local

spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce # 노드의 접근 방법으로 ReadWriteOnce이면 하나의 노드에서 읽기쓰기, ReadOnlyMany이면 여러 노드에서 읽기 전용으로 사용하고 ReadWriteMany는 여러 노드에서 읽고 쓰기가 가능하고 ReadWriteOncePod는 하나의 파드에서만 읽고 쓰기가 가능
  hostPath:
    path: "mnt/data"
# 위에 ReclaimPolicy를 설정할 수 있는데 영구 볼륨 클레임이 삭제되었을 때 영구 볼륨을 어떻게 할 것인지 여부를 설정하는 것인데 retain을 설정하면 클레임이 삭제되도 영구 볼륨은 보존되고 delete를 설정하면 같이 삭제되고 recycle은 다른 노드에서 사용 가능한 상태로 설정

# 지금은 노드가 1개라서 hostPath를 설정했지만 nfs 서버를 이용한다면 path 속성에 디렉토리를 설정하고 server에 nfs 서버의 ip를 설정
```
  - pv.yml을 실행시켜서 영구 볼륨을 생성
  ```
  kubectl apply -f pv.yml
  ```
  - Persistent Volume Claim을 위한 yaml 파일 생성: pvc.yml
  ```yml {filename="pvc.yml"}
  apiVersion: v1

  kind: PersistentVolumeClaim

  metadata:
    name: mysql-pv-claim
  
  spec:
    storageClassName: manual # 영구 볼륨의 스토리지이름을 기재
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 20Gi
  ```
  - pvc를 생성
  ```bash
  kubectl apply -f pvc.yml
  ```
  - 영구 볼륨 클레임을 사용하는 Pod 생성을 위한 Deployment를 생성: pvc-deployment.yml
```yml {filename="pvc-deployment.yml"}
apiVersion: apps/v1

kind: Deployment

metadata:
  name: mysql

spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:8.0.29
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: var/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```
  - pod 배포
  ```
  kubectl apply -f pvc-deploment.yml
  ```
```
외부 스토리지(NFS, AWS EBS)<-->영구 볼륨(PV)<-ClassName->영구 볼륨 클레임(PVC)<-->API 서버(K8S)<-->Pod(DB)
```