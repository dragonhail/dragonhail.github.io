---
title: Build & Registry
weight: 10
---
```
0. 컨테이너 와 이미지 모두 삭제
=>모든 컨테이너 중지: docker stop $(docker ps -a -q)
=>모든 컨테이너 삭제: docker rm $(docker ps -a -q)

=>모든 이미지 삭제: docker rmi $(docker images -q)
```

## 빌드 의존성 제거와 다단계 빌드
- 다단계 빌드는 FROM 명령을 이용해서 여러 단계의 빌드 과정을 만들고 다른 단계에 AS를 이용해서 이름을 부여해서 사용할 수 있도록 하는 것
- 다른 단계에서 생성된 결과 중 애플리케이션 구동에 필요한 특정 데이터만 가져올 수 있기 때문에 이미지를 경량화
- 다단계 빌드로 작성된 이미지는 모든 빌드 의존성이 하나의 환경에 포함되므로 빌드 의존성을 제거할 수 있음

### go 애플리케이션을 다단계 빌드로 이미지를 생성
- 디렉토리를 생성하고 그 디렉토리로 프롬프트를 이동
```mkdir goapp && cd $_```
- go 파일을 생성(goapp.go)
```go
package main

import(
"fmt"
"time"
)

func main(){
        for{
                fmt.Println("Hello World")
                time.Sleep(10 * time.Second)
        }
}
```
- golang 설치
  - 다운로드: wget https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
  - 압축해제: sudo tar -xzf go1.19.1.linux-amd64.tar.gz -C /usr/local
  - 압축해제된 go/bin 디렉토리를 PATH에 등록(명령어 또는 파일을 아무 곳에서나 사용할 수 있게 하기 위해서)
  - `sudo nano /etc/profile` 명령을 수행하고 아래 내용 추가
    - `export PATH=$PATH:/usr/local/go/bin`
  - 위의 내용을 추가한 후 저장하고 `source /etc/profile` 명령으로 변경 내용 적용
  - 버전 확인 `go version`
- 이전에 만든 go 파일 실행
  - 빌드: `go build 파일명`
  - 실행: ./파일명에서 확장자를 제거한 부분
  - node go C java는 빌드 과정이 있음, python은 없음
- Dockerfile을 생성해서 작성
```dockerfile
FROM golang:1.15-alpine3.12 AS gobuilder-stage

WORKDIR /usr/src/goapp

COPY goapp.go .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /usr/local/bin/gostart

FROM scratch AS runtime-stage

COPY --from=gobuilder-stage /usr/local/bin/gostart /usr/local/bin/gostart

CMD ["/usr/local/bin/gostart"]

```
- Dockerfile 빌드
  - docker build -t 이미지이름:태그 [-f Dockerfile 경로] .
- 이미지 확인
  - `docker images`
- 컨테이너로 실행
  - `docker run --name goapp-deploy goapp:1.0`

## Private Registry
- Docker Hub는 기본적으로 public으로 저장소가 만들어지기 때문에 주소만 알면 아무나 접근이 가능
- Docker Hub에서도 Private 저장소를 제공하는데 이 저장소는 1개만 무료이고 그 이후부터는 유료
- Docker Hub에서는 private registry를 위한 registry라는 이미지를 제공하는데 이 이미지를 컨테이너로 생성하고 이 컨테이너에 이미지를 저장하는 개념
- 단순 텍스트 방식만 지원하기 때문 웹 기반의 검색을 할려면 GUI 인터페이스를 제공하는 다른 컨테이너와 결합을 해야 함

### private registry를 만들고 이미지를 저장할 수 있는 서버 생성
- 이미지 조회
```docker search registry```
- private registry를 구성할 수 있는 이미지를 다운로드
```docker pull registry```
- 이미지를 업로드 할 수 있는 저장소 사용을 위한 설정을 /etc/init.d/docker 파일의 DOCKER_OPTS 항목에 추가
```sudo nano /etc/init.d/docker```
  - DOCKER_OPTS=--insecure-registry 컴퓨터의IP:포트번호
  - ```DOCKER_OPTS=--insecure-registry 10.0.2.15:5000```
- 도커를 재시작
```sudo service docker restart```
- registry를 컨테이너로 실행
```docker run --name local-registry -d -p 5000:5000 registry```
- 확인
```docker ps```

### private registry를 사용할 수 있는 클라이언트 작업
- private registry를 사용할 수 있도록 설정을 수정:  /etc/docker/daemon.json  없으면 생성
- sudo nano /etc/docker/daemon.json 명령으로 파일을 열고 추가
```{"insecure-registries":["10.0.2.15:5000"] }```
- 도커 재시작
```sudo service docker restart```
- 도커 정보 확인