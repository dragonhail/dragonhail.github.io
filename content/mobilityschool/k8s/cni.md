---
title: CNI
weight: 8
---
## 컨테이너 네트워크 서비스(CNI)

### 볼륨을 통한 데이터 공유

### SDN(Software-Defined Networking: 소프트웨어 정의 네트워크)
- 네트워크 리소스를 최적화하고 변화하는 비지니스 요구, 애플리케이션 및 트래픽에 신속하게 네트워크를 채택하는데 도움이 되는 네트워크 가상화 및 컨테이너화에 대한 접근 방식
- 네트워크 제어와 데이터 평면을 분리하여 소프트웨어 프로그래밍 가능 인프라를 만드는 방식

### 쿠버네티스에서 SDN이 필요한 이유
- 쿠버네티스 네트워킹에는 수백 개의 파드가 있고 그 중 일부가 같은 서비스에 대응하는 경우 파드가 이동해서 모든 트래픽이 항상 올바른 위치로 갈 수 있도록 클러스터에 들어오고 나가는 트래픽을 계속 라우팅 할 수 있어야 함
- 위의 목적을 달성하기 위한 쿠버네티스에서 제공하는 2가지 도구
  - Service Proxy: 정적 IP를 갖는 서비스 뒤에 파드가 로드 밸런싱되고 쿠버네티스 서비스 객체가 라우팅 되는 것을 보장해줌
  - CNI: 파드가 클러스터의 일정하면서도 쉽게 액세스 할 수 있는 네트워크 내부에서 계속해서 다시 재생될 수 있음을 보장해 줌
- 이를 위해서 ClusterIP 타입을 갖는 쿠버네티스 서비스 객체가 생성됨
  - ClusterIP 서비스는 쿠버네티스 서비스로 쿠버네티스 클러스터에서 라우팅이 가능하지만 클러스터 외부에서는 액세스 할 수 없음
  - ClusterIP 서비스는 다른 서비스의 상단에 만들어질 수 있는 기본적인 요소
  - 클러스터에서 애플리케이션이 직접 라우팅 할 필요없이 서로 액세스 할 수 있는 가장 간단한 방법은 파드의 IP주소를 활용하는 것
  - 서비스도 생성될 때 별도의 IP 주소를 가지고 생성되는데 이 때 각 서비스는 별도의 서브넷을 가지고 만들어짐<br>
  서비스에 파드를 등록하면 파드 레플리카들에 별도의 이름을 할당하고 그 이름에 IP를 매핑하는 DNS를 소유하게 됨<br>
  일종의 NAT 와 유사한 형태의 방식을 사용
- NodePort 서비스 와 ClusterIP 서비스
  - NodePort 서비스는 내부 파트 네트워크의 모든 포트를 외부에 노출시키는데 이를 이용해서 로드밸런서를 만들 수 있게 함
  - NodePort를 설정하게 되면 동일한 성격의 파드는 NodePort가 전부 개방됨
- 클러스터 파드 내부에서 실행되는 CoreDNS 컨테이너를 가리키는 NodePort 서비스 생성
```yml {filename="my-nodeport.yml"}
apiVersion: v1

kind: Service

metadata:
  annotation:
    prometheus.io/port: "9153"
    prometheus.io/scrape: "true"
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns-2
  namespace: kube-system

spec:
  ipFamlies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  
  selector:
    k8s-app: kube-dns

  sessionAffinity: None
  type: NodePort
```
  - 서비스 생성
  ```
  kubectl apply -f my-nodeport.yml
  ```
  - 서비스 확인
  ```
  kubectl get svc
  ```

## Spring Boot Application을 NodePort를 이용해서 모든 Node에서 포트를 개방해서 서비스 할 수 있도록 하기
### 1. 애플리케이션 생성
- Intellij에서 Spring Boot Application 생성(Lombok 과 Web, devtools 에 대한 의존성만 추가)
- 기본 패키지 안에 요청을 처리하는 클래스를 추가하고 작성(FrontController)
```java {filename="FrontController.java"}
package com.gmail.dragonhailstone.apiserver;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
public class FrontController {
    @GetMapping("/")
    public Map<String, Object> index() {
        Map<String, Object> data = new HashMap<>();
        data.put("result", "success");
        
        List<Map> list = new ArrayList<>();
        Map<String, String> map1 = new HashMap<>();
        map1.put("id", "itstudy");
        map1.put("name", "아담");
        list.add(map1);
        
        data.put("list", list);
        
        return data;
    }
}
```
- 실행 한 후 브라우저에서 8080 포트로 접속해서 에러가 없는지 확인
### 2. Dockerfile 파일 만들어서 이미지 생성
- 프로젝트의 루트 디렉토리에 Dockerfile을 생성하고 작성
```dockerfile
FROM amazoncorretto:17
CMD ["./mvnw", "clean", "package"]
ARG JAR_FILE=target/*.jar
COPY ./build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```
```
=>이미지 생성
docker build -f Dockerfile -t apiserver:0.0.1 .

=>이미지 확인
docker images

=>컨테이너로 실행
docker run -d --name apiserver -p 80:8080 apiserver:0.0.1
```
### 3.Github이용해서 DockerHub에 배포
- git hub에 레포지토리 생성: apiserver
=>git hub에 push
```
git init

git add .

git commit -m "init"

한번도 사용하지 않아서 이메일 과 이름을 등록하라고 메시지 출력
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

브랜치 확인(현재 git hub는 main이 기본 브랜치)
git branch
브랜치 변경
git checkout -b main

원격 저장소 등록
git remote add 저장소이름 url
git remote add origin https://github.com/itstudy001/apiserver.git

push: windows 나 mac에서는 브라우저에서 로그인하고 linux의 경우는 콘솔에서 로그인하라는 메시지가 출력됨
git push origin main
```
- push가 발생할 때 Dockerhub에 이미지 배포
  - Docker Hub에 Repository를 생성: apiserver
  - Docker Hub에서 토큰 발급
  - 프로젝트 안에 .github/workflows/upload.yml
```yml {filename="upload.yml"}
name: Java CI with Gradle

on:
  push:
    branches: ["main"]

jobs:
  build-docker-image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      # JDK 17 설치
      - name: Set Up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run chmod to make gradlew executable
        run: chmod +x ./gradlew

      # 빌드
      - name: Build andGradle
        uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
        with:
          arguments: clean bootJar

      # 도커 허브 로그인
      - name: Login To DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 이미지 빌드 및 업로드
      - name: Image Build and Release to Dockerhub
        env:
          NAME: ${{ secrets.DOCKERHUB_USERNAME }}
          REPO: ${{ secrets.DOCKERHUB_REPOSITORY }}

        run: |
          docker build -t $REPO .
          docker tag $REPO:latest $NAME/$REPO:latest
          docker push $NAME/$REPO:latest
```
```
push
git add .
git commit -m 메시지
git push 저장소이름 브랜치이름
```
  - 배포가 제대로 되었는지 도커 허브에서 확인
  - 이미지가 제대로 동작하는지 확인

### 4. 쿠버네티스 Deployment 이용해서 파드 생성
- pod를 생성할 yml 파일을 생성
```yml {filename="springbootdeployment.yml"}
apiVersion: apps/v1

kind: Deployment

metadata:
  name: devops-spring-deployment

spec:
  selector:
    matchLabels:
      app: devops-spring-app
  
  replicas: 2
  template:
    metadata:
      labels:
        app: devops-spring-app
    spec:
      containers:
      - name: core
        image: dragonhailstone/apiserver
        imagePullPolicy: Always # 컨테이너 재실행 시 image pull
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          requests:
            cpu: 500m
            memory: 1000Mi
```
### 5.쿠버네티스 Service 이용해서 NodePort 설정
- nodeport를 이용할 서비스 파일을 생성: service.yml
```yml {filename="service.yml"}
apiVersion: v1
kind: Service
metadata:
  name: devops-spring-service
spec:
  type: NodePort
  ports:
  - port: 80 # Service 포트, 노드포트는 자동할당
    protocol: TCP
    targetPort: 8080 # Pod 포트
  selector:
    app: devops-spring-app
```
- 서비스 생성 `kubectl apply -f service.yml`
- 결과 확인
```
dh@swarm-manager:~/1029$ kubectl get svc -o wide
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
devops-spring-service   NodePort    10.96.134.15   <none>        80:31901/TCP   2m46s   app=devops-spring-app
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP        28h     <none>

Cluster안에서 NodePort의 Cluster-IP(PrivateIP)의 31901포트로 들어갈 수 있음
```
- endpoint 확인
```
dh@swarm-manager:~/1029$ kubectl get ep
NAME                    ENDPOINTS                         AGE
devops-spring-service   10.244.1.2:8080,10.244.2.2:8080   2m56s
kubernetes              172.18.0.2:6443                   28h

엔드포인트 IP(Pod의 IP) 2개를 볼 수 있음
노드포트로 들어가게 되면 두개 파드의 8080포드 중 하나로 들어가게 됨
```
- 서비스 와 동일한 이름의 endpoint가 있으면 service에 요청을 하면 endpoint에 요청을 한다는 의미
