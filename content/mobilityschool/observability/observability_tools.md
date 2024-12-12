---
title: 관측 가능성 기반 기술
weight: 2
---
## 2.관측 가능성 기반 기술
### 1)트래픽 관리
- 단일 장애점

- Load Balancer

- 복원성 패턴
  - 재시도
  - 비율 제한
  - 벌크 헤드
  - 서킷 브레이커
  - 분산 메시지 시스템(Kafka, Rabbit MQ) 사용
  - 관측 가능성은 비즈니스 애플리케이션에 비해 대용량 트랜잭션을 처리하는 것이 일반적
  - 관측 가능성을 운영하면 메시지 유실이 빈번하고 시스템 증설의 필요성이 여러차례 부각될 것

- 가시성(VIsibility)
  - 다양한 장애를 극복할 수 있는 복원력을 제공
  - 네트워크 분야에서 사용되는 용어
  - 배경에서 분리된 가시 대상의 존재나 색의 차이를 잘 볼 수 있는 정도 또는 상태
  - 가시성의 대상은 주로 하드웨어 나 인프라
  - 벤더에서 제공하는 메트릭을 사용

- 서비스 메시
  - Istio와 넷플릭스의 open source software가 많이 사용되는데 OSS가 이스티오가 제공하지 않는 세밀한 조정도 수행
  - Istio에 대비한 open source software의 장점   
    클라이언트 측 부하 분산을 사용해서 오토스케일링을 구현   
    OSS는 Go와 파이썬을 사용해서 애플리케이션을 개발했기 때문에 다양한 플랫폼과 개발언어를 지원하지 못함

### 2)쿠버네티스 오토스케일링
- CPU 사용률이나 기타 메트릭을 체크하여 파드의 개수를 스케일링하는 HPA(Horizontal Pod Autoscaler) 기능을 제공
- 지정한 메트릭을 컨트롤러가 체크해서 부하에 따라 필요한 레플리카 수가 되도록 자동으로 파드를 늘리거나 줄이는 것
- HPA는 파드를 오토스케일링하고 다음 리소스에 파드 스케일링을 사용할 수 있음
  - Deployment
  - ReplicaSet
  - ReplicaController
  - StatefulSet
- 관측 가능성은 대용량 트래픽을 안정적으로 처리하고 비용을 절감해야 하므로 관측 가능성 시스템에도 쿠버네티스 오토스케일링을 적용해야 함
- 오토스케일을 하는 방법은 다양
  - AWS는 EC2 가상 머신을 기반으로 오토스케일링 기능을 제공
  - 가상 머신은 OS를 중심으로 네트워크, 스토리지가 모두 결합되므로 의존성이 높고 용량이 커서 오토스케일링에 필요한 복제과정에서 시간이 오래 걸리고 애플리케이션의 수평적인 확장 시 불필요한 부분까지 확장이 되므로 비용과 자원의 최적화 측면에서 효과적이지 못하고 수평적인 확장을 하는 것이 어렵기 때문에 컨테이너 사용을 권장
  - 오토스케일링은 파드를 수평적으로 증감시키고 유저의 트래픽을 증감된 파드로 고르게 분배하는 것이 중요
  - 측정된 메트릭을 기반으로 파드를 증가시키는 흐름   
    메트릭 측정을 위해 메트릭 서버를 사용하지만 이 경우 CPU 와 메모리 메트릭만을 지원한다는 한계가 있어서 애플리케이션의 메트릭 수집/관리를 위한 프로메테우스 어댑터, KEDA(kubernetes event driven autoscaling)등을 사용
  - 유저의 트래픽을 각 파드로 분산하는 흐름   
    측정된 메트릭에 따라 오케스케일링된 파드와 연계하고 유저에게 실제 서비스를 제공하는 오토스케일링 처리는 복잡   
    쿠버네티스 서비스 앞에 로드밸런서가 구축된 운영 환경을 구성
- 쿠버네티스에서의 오토스케일링
  - 선언적으로 표현된 요청(yaml 파일 이용)을 유지함으로써 수 많은 컨테이너로 구성된 애플리케이션의 오케스트레이션 과 관리를 자동화
  - 수동으로 또는 특정 규칙이 주어지면 완전히 자동화된 방식으로 컨테이너의 자원, 요청한 서비스 레플리카 수, 클러스터의 노드 수 등을 쉽게 변경할 수 있음
  - 쿠버네티스는 고정된 파드와 클러스터 설정을 유지할 수 있을 뿐 아니라 외부 로드와 용량 관련 이벤트의 모니터링을 통해 상태를 분석하고 원하는 성능에 맞게 자체 스케일을 할 수 있음
  - 쿠버네티스는 애플리케이션에 가장 적합한 설정을 찾기 위한 다양한 기능 과 기술을 제공   
    가장 간단한 방법이 HPA를 사용하는 것
- HPA Controller가 동작하는 방법
  - HPA 정의에 따라 스케일링된 파드에 대한 메트릭을 가져옴   
	직접 메트릭을 읽지 않고 집계된 메트릭을 제공하는 쿠버네티스 메트릭 API를 이용해서 메트릭을 읽음   
    파드 레벨의 자원 메트릭은 쿠버네티스 메트릭 API에서 가져오고 다른 모든 메트릭은 쿠버네티스 사용자정의 메트릭 API에서 가져옴   
    현재 메트릭과 목표하는 요청한 메트릭 값을 기반으로 레플리카 수를 계산   
    각 메트릭을 개별적으로 평가한 후 가장 큰 값을 제안 - 계산을 완료하면 요청한 임계값 이하이면서 요청한 레플리카 수 보다는 이상인 정수를 오토스케일링 한 값으로 최종 선택
  - HPA가 평가하는 메트릭 타입   
    표준 메트릭: CPU 와 같은 자원 사용량 메트릭을 나타내는 것으로 일반적으로 메트릭 서버에서 제공하는 가장 사용하기 쉬운 메트릭

    사용자 정의 메트릭: 클러스터 별로 각기 다른 클러스터 모니터링 설정이 필요한데 이는 custom.metrics.k8s.io API 경로의 Aggregated API 서버로 수행하고 프로메테우스 등의 다양한 메트릭 어댑터에서 제공
  - 권장 사항   
    파드 레벨의 오토스케일링과 노드 레벨의 오토스케일링(클러스터)을 함께 사용

    스케일 아웃(파드 증가) 보다는 스케일 인(파드 감소) 과정에서 장애가 발생할 가능성이 높음: 스케일 인을 위한 상세 매개변수를 설정하는 것이 중요

    커스텀 메트릭을 개발하고 프로메테우스를 통한 고도화된 오토스케일링을 추천하지만 복잡하게 연계된 마이크로서비스 기반의 애플리케이션을 사용 중이라면 주요한 리소스(CPU, Memory)에 기반한 오토스케일링을 시작하는 것을 권장

    노드 오토스케일러는 대부분 자동화 기능을 제공해주기 때문에 운영자 입장에서는 수작업이 필요없는 경우가 대부분이지만 drain, cordon, taint 명령 와 PDB, affinity 등 리소스 설정은 쿠버네티스 운영 유용하며 그라파나 관측 가능성 헬름(Helm) 차트를 기본적으로 메트릭 서버를 사용해서 CPU, 메모리 기반의 오토스케일링과 PDB를 사용하며 KEDA를 사용해서 요구 사항에 적합하게 고도화 할 수 있음
- 오토스케일링 관련 Open Source
  - 지원하는 오픈 소스   
    Metric Server

    Prometheus Adapter

    KEDA
  - Metric Server   
    Pod -> kubelet(리소스 관리를 위한 노드 에이전트, Summary API는 사용 가능한 노드별 요약 정보를 탐색하고 수집할 수 있도록 kubelet이 /status 엔드포인트를 통해서 제공하는 API) -> Metric Server(리소스 메트릭을 수집하고 집계하는 역할, Metric API) -> API Server -> HPA

    메트릭 서버를 사용하기 위해서는 deployment를 만들 때 반드시 resources 안에 리소스 사용량을 명시해야 함
    ```yaml
    resources:
      limits:
        cpu: 100m
      requests:
        cpu: 200m
    ```
    cpu   
	측정된 평균 코어 사용량 형태로 보고되는데 1CPU는 클라우드 제공자의 경우 1vCPU/코어 에 해당하고 베어메탈 인텔 프로세서의 경우 1하이퍼 스레드에 해당

    메모리   
    바이트 단위로 측정된 working set 형태로 보고   
    이상적인 환경에서는 워킹셋은 해제할 수 없는 사용 중인 메모리 양   
    운영체제마다 계산 방법이 다름

    메트릭 서버   
    리소스 메트릭을 수집하고 쿠버네티스 API Server에게 HPA나 VPA(Vertical Pod AutoScaler)가 활용할 수 있는 정보를 제공   
    이 정보를 `kubectl top` 명령으로 확인   
    파드 헬스에 대한 캐시를 유지
  - Prometheus Adapter   
    Adapter에 설정을 추가: 어떤 네임스페이스의 파드를 측정할 것인지 설정

    HPA를 생성해서 Adapter에서 설정한 Metric을 이용해서 오토스케일링 하도록 해주어야 함
    
    초당 리퀘스트 수 또는 시간 등을 이용해서 파드를 오토스케일링 하는 것이 가능
  - KEDA   
    이벤트 소스(재시도, dead letter, check point 등)를 모니터링하고 해당 데이터를 HPA에 공급해서 리소스를 빠르게 확장하는 역할을 수행   
    HPA <- KEDA(Metric Adapter, Controller, Scaler) <- Kubernetes API Server

    KEDA는 이벤트 트리거 소스 와 연결 가능

    이벤트 트리거 소스로 Apache Kafka를 사용하는 것이 가능

    애플리케이션이 Kafka에게 메시지를 보내고 Kafka가 KEDA에게 이벤트를 전송해서 동작시킬 수 있음

    - 워크 플로   
      pending 메시지가 없으면 KEDA는 디플로이먼트를 0으로 확장   
      메시지가 도착하면 KEDA는 이것을 이벤트로 감지하고 디플로이먼트를 활성화 시킴   
      디플로이먼트가 실행되면 컨테이너 중 하나가 카프카에게 연결되고 메시지를 가져오기 시작   
      카프카 토픽에 더 많은 메시지가 도착하면 KEDA는 이 데이터를 HPA에 공급하여 수평 확장 시킬 수 있음

    메트릭 서버는 CPU 등 컴퓨팅 자원만을 지원하고 프로메테우스 어댑터는 프로메테우스만 지원하는데 반해서 KEDA는 redis, kafka, prometheus 등 다양한 자원의 메트릭을 직접 측정하고 오토스케일링 할 수 있음

    프로메테우스 어댑터는 프로메테우스 에코시스템과 쿠버네티스와 호환이 필요하기 때문에 KEDA 사용을 권장   
    Metric server < Prometheus Adapter < KEDA
- 메트릭 측정
  - 기존의 성능 관리에서는 초당 처리 개수 및 처리 지연 시간이 중요
  - 요청 수를 이용해서 애플리케이션을 자동으로 확장해야 하는지 판단하는 방법이 많이 사용되었음
  - 요청 시간도 중요한 값 중 하나인데 총 요청 시간만 중요한 것이 아니고 사용자 경험 측면에서도 판단을 해봐야 함
  - 동시에 처리할 수 있는 요청의 개수
- 메트릭 선정
  - 측정하고자 하는 메트릭을 식별했으면 부하 테스트를 거쳐 검증
  - `kubectl top pod` 나 `kubectl top node` 명령을 실행해서 스트레스 테스트를 진행하면서 메트릭 증감을 모니터링 해야 함
  - 지연시간을 측정한 경우   
    히스토그램을 만들어서 확인   
    시간에 따라서 처리되는 갯수가 현저히 줄어드는 경우   
        0.1초 이내에 60%가 처리되고   
        0.25 초 이내에 30% 정도 처리되고   
        0.5초 이내에 8% 정도 처리되고   
        1초 이내에 2% 정도 처리되는 경우   
        이 경우는 0.25 초 이내에 처리되는 경우가 많으므로 이런 경우는 오토스케일을 적용하지 않아도 됨, 2%만 느려진 것은 이상치에 해당되므로 원인 분석 필요(특별한 상황인지 순간적인 것인지 등)

        0.1초 이내에 30%가 처리되고   
        0.25 초 이내에 25% 정도 처리되고   
        0.5초 이내에 20% 정도 처리되고   
        1초 이내에 15% 정도 처리되는 경우   
        시간대 별로 처리되는 양이 차이가 적은 경우는 되도록이면 오토스케일링을 적용해서 작업을 특정 시간 내에 지연없이 처리되도록 해주어야 함(예측이 안되기 때문)

#### Replicas vs HPA
- replicas는 고정된 개수, 장애가 발생한 경우 조정
- HPA는 메트릭을 지정
  - CPU, Memory, Request 수를 이용해 파드의 개수 조정
  - 노드는 늘릴 수 없음
  - HPA가 replicas 보다 상위의 개념으로 봄

### 3)관측 가능성 프로세스
- 관측 가능성 운영 프로세스
  - 시스템 장애가 없는 프로세스   
    메트릭 분석 -> 추적 분석 <-> 로그 분석
  - 장애 발생 시 운영 프로세스   
    장애 발생 -> 알림 전송 -> 추적 분석 -> 로그 분석
  - 최근에는 운영 부분에 AI를 도입하는 AIOps를 구현하는 경우가 있는데 이것은 불필요한 비용을 절감하고 운영자의 업무 효율성을 향상시키는 방향으로 검토하라고 함

### 4)수평 샤딩
- 그라파나 관측 가능성인 미미르, 템포, 로키는 모두 샤딩과 해싱 등의 기술을 기반으로 사용함
- 그라파나 관측 가능성에 사용하는 기술은 운영을 목적으로 하기 때문에 이해하는 것도 쉽지 않고 구성하는 것도 복잡
- 프로메테우스 기반으로 그라파나 관측 가능성은 개발
- 그라파나 관측 가능성은 읽기 와 쓰기가 분리되어 있음
- 쓰기에는 Ingester, Distributor 가 있으며 읽기에는 Querier, Query Frontend 가 있습니다.   
  각각의 요소를 개별적으로 샤딩 구성이 가능합니다.(CQRS)
- 데이터베이스의 수평적 확장을 샤딩이라고 하는데 이 방법은 시스템의 확장이나 성능 향상, 높은 안정성을 갖춘 클러스터를 구성하는 방법   
  최근에는 카프카, 레디스, 카산드라, MongoDB 등에서도 유사한 개념으로 사용을 함

#### CQRS
- 낮은 의존성
- 확장과 축소를 별개로

#### 파티셔닝과 샤딩
- 파티셔닝: 분할
- 샤딩: 수평적 확장(쿠버네티스에서의 replicas에 해당)

### 5)MicroService
- 마이크로서비스 개발 흐름   
  바이너리 -> 도커이미지 -> 쿠버네티스 파드
  - 언어나 프레임워크를 이용해서 개발한 소스를 Makefile로 빌드를 해서 바이너리를 생성하는데 운영 환경에서는 바이너리로 실행될 수 있고 도커 이미지, 쿠버네티스 파드 등 다양한 런타임 환경에 배포하고 운영할 수 있음
  - Dockerfile 등을 사용해서 이미지를 빌드하는데 도커 이미지가 여러 개라면 Docker Compose를 이용해서 도커 런타임에 배포
  - 도커 이미지를 참조해서 헬름 차트를 만들고 헬름 차트를 쿠버네티스에 배포
- CQRS
  - 대다수의 애플리케이션에서 읽기와 쓰기는 많이 다름
  - 쓰기는 정규화된 단일 엔트리에 영향을 주는데 반해 질의는 소스 범위에서 정규화되지 않은 데이터를 조회할 수 있음
  - **RDBMS에는 정규화시켜 삽입하지만 No SQL에는 조인된 데이터를 삽입해야 함(linking이나 embedding 통해, 질의 시 join 줄이기 위해)**
    - User테이블과 Board 테이블이 있으면 NoSQL에 Join한 테이블을 삽입
  - 작업들을 분리해서 사용하는 경우가 있는데 영속적인 트랜잭션 저장은 PostgreSQL을 이용하고 인덱스 조회 질의에서는 ElasticSearch를 이용하는 형태를 사용하는 것이 가능
  - 장점   
    독립적인 크기 조정: 디플로이먼트, HPA, VPA를 이용해서 수평 및 수직 확장 가능

    최적화된 데이터 스키마: 질의 쪽에는 쿼리에 최적화된 스키마를 사용하고 쓰기 쪽에는 업데이트에 최적화된 스키마를 사용

    관심사의 분리: 대부분의 복잡한 비즈니스 논리는 쓰기 모델로 이동하고 읽기 모델은 상대적으로 간단해 짐

    보안 강화: 데이터에서 올바른 도메인 엔티티만 쓰기를 수행할 수 있는지 쉽게 확인 가능

    단순한 쿼리: 읽기 데이터베이스에서 구체화된 뷰를 저장하여 쿼리를 실행할 때 애플리케이션에서 복잡한 조인이 발생하는 것을 방지
  - 마이크로서비스 기반으로 개발된 많은 오픈소스 아키텍쳐도 내부적으로 CQRS를 구현
  - 대용량 데이터를 처리할 때 디스크 IO에서 병목 현상이 발생하는 것을 방지하기 위해서 내부적으로 질의와 쓰기 프로세스를 분리하거나 레디스를 캐시로 사용해서 데이터를 관리하고 객체 스토리지에 장기간 데이터를 보관하며 병렬 처리 하는 것 등이 실제 사용되는 방식
  - 그라파나 관측 가능성이나 일라스틱서치도 동일하게 작동
  - 대용량 데이터를 다루고자 하는 경우는 튜닝을 하는 과정에서 읽기와 쓰기 비율을 조정하는 거의 무조건 발생함
```
- CRUD
  - Create(Insert)(DML)
  - Read(Select, DQL)
  - Update(DML)
  - Delete(DML)
- Query(질의)
- Command(명령)
  - Command Storage(RDBMS): 쓰기 작업
  - Query Storage(NO SQL): 읽기 작업
- MongoDB의 대안: Redis, ElasticSearch(읽기 특화 DB)
```
### 6)일관된 해시
#### 일반 해시
- N개의 캐시 서버가 있다고 가정할 때 이 서버에 부하를 균등하게 나누는 보편적 방법은 해시 함수를 사용하는 것
- 예를 들면 해시 함수로 mod 알고리즘을 사용   
  `서버인덱스 = 해시(키) % 서버의개수`
- 서버가 4대 있는 경우
```
키		해시		해시%4
키0 	401		1
키1		800		0
키2		242		2
키3		100		0
키4		310		2
키5		511		3
키6		86		2
키7		397		3

해시값 분산
서버		서버0		서버1		서버2		서버3
키		  키1		  키0		  키2		  키5
		    키3			        키4		  키7    
                        키6		
```
- 서버 풀의 크기가 고정되어 있고 데이터 분포가 균등할 때는 잘 동작합니다.
- 서버가 추가되거나 기존 서버가 삭제되면 문제가 발생   
  서버1에 장애가 발생해 동작이 중단되면 서버 풀의 크기가 3으로 변하게되는데 키에 대한 해시값은 변하지 않지만 나머지 연산을 적용해서 계산한 서버 인덱스 값은 달라지게 됨
- 서버에 장애가 발생하면 발생할 수 있는 현상   
  데이터가 복수 개로 복제되어 있거나 세컨더리에서 복제를 지원하는 등 장애 복구를 구성했다면 데이터 유실은 방지할 수 있지만 최악의 경우 데이터 유실이 발생할 수 있음

  해시 키를 재생성하고 적합한 로직에 따라 데이터를 재분배해야 합니다.

  서버1이 다운되면서 서버1에 저장된 데이터를 다른 서버 0,2,3 으로 분배

  노드 간 데이터가 분배되는 과정에서 성능 저하가 발생

#### 일관된 해시
- 해시 테이블의 크기가 조정될 때 평균적으로 오직 k/n 개의 키만 재배치하는 해시 기술   
  k는 키의 개수이고 n은 슬롯의 개수   
  전통 해시 기법의 해시 테이블은 슬롯의 수가 변경되면 거의 대부분 키를 재배치

#### 해시 공간과 해시 링
- 해시 함수로는 SHA-1을 사용하고 이 때 나올 수 있는 출력값의 범위는 X0, X1, X2, X3 등
- 해시 공간의 양쪽을 구부려서 접으면 해시 링이 만들어짐
- 해시 서버 와 키는 링 위의 어느 부분이든 배치가 가능
- 키는 오른쪽 방향으로 가장 가까운 곳에 배치
- 서버의 증설이나 감소의 경우 일부분의 키만 재배치
- 레디스, 카프카, 일라스틱서치 처럼 노드를 증설하거나 축소하는 경우가 빈번한 오픈소스에 사용

#### 일관성 해시라는 이름에서 일관성은 노드, 키, 데이터 등의 변화에 상관없이 영향을 최소화하고 외부에서 시스템을 바라볼 경우 안정적이라는 의미

### 7)관측 가능성 시각화
- 시각화의 장점은 대시보드를 보는 사용자에게 신속하게 의미를 전달할 수 있다는 점
- 파이, 바, 라인 차트는 단순한 값의 증감을 나타내기 때문에 관측 가능성의 복잡한 의미를 표현하기에 적합하지 않음   
  상관, 분포, 변동폭, 절차, 증감, 관계, 군집, 순서 등을 이해하는데 도움이 되는 차트를 사용하는 것이 좋습니다.
- 많은 시간이 걸리는 머신러닝 알고리즘 과 애플리케이션 개발보다는 대시보드를 사용하는 시각화를 도입하는 것이 좋은 방법
- 관측 가능성을 사용하는데 가장 기본적인 차트는 시계열 차트
  - 시계열 차트에 이그젬플러를 추가하여 추적 과 연계할 수 있음
  - 분포, 상관, 수치에 대한 정보를 제공하지 않기 때문에 히스토그램, 히트맵 등과 함께 사용해서 시스템의 상태를 보다 자세하게 표현하는 것이 좋음
- Heatmap은 다양한 애플리케이션 간의 상관 관계를 비교 및 이해하고 근본 원인 분석을 위해서 사용
  - 시계열 차트 와 유사하지만 이그젬플러를 제공하지 않음
  - 색의 명도를 이용해서 차이점을 구분
  - 시간별 히스토그램을 포함
  - 데이터셋의 변수가 서로 관련되어 있고 서로에 대해 어떻게 움직이는지 알려줍니다.
  - 상관 계수를 계산하고 결과를 히트맵으로 표현하면 데이터의 상관 관계를 쉽게 시각화하고 이해할 수 있습니다.
- 히스토그램
  - 버킷은 일반적인 시간 범위 내에서 애플리케이션의 요청을 관리
  - 개수 와 합계에 따른 분포도를 나타내는데 유용하지만 평균을 사용하기 때문에 세부적인 이상치가 출력되지 않고 무시되기도 하고 변동량을 보여주지 못함
  - CandleStick을 함께 사용하면 분포도와 변동량 그리고 폭을 동시에 이해할 수 있습니다. 
- flame graph
  - 프로파일을 시각화할 때 사용
  - 하위 수준의 시스템 정보를 가장 명확하게 출력
### 8)Key-Value Store
- 개요
  - 키-값 데이터베이스라고 불리우는 비관계형 데이터베이스(NoSQL)
  - 이 저장소에 저장하는 데이터는 고유 식별자를 키로 가져야 합니다.
  - 키는 일반 텍스트(S3에서 오브젝트에 대한 키)일 수 있고 해시값(AWS의 ID 나 리소스를 구별하기 위한 문자열)일 수 도 있습니다.
  - 성능 상의 이유로 키는 짧을 수록 좋음
  - 가장 널리 알려진 키-값 저장소는 Dynamo DB 와 Redis, MemCached
  - 그라파나에서는 레디스 보다는 MemCached를 권장
- 키값 저장소를 사용할 때 고려사항
  - 키값 쌍의 크기는 10KB 이하
  - 사이즈가 큰 데이터를 저장할 수 있어야 합니다.
  - 시스템은 장애에도 빨리 응답하는 높은 가용성을 제공해야 함
  - 트래픽 양에 따라 자동적으로 서버 증설, 삭제가 이루어지는 높은 규모의 확장성을 제공해야 합니다.
  - 데이터 일관성 수준은 조정이 가능해야 합니다.
  - 응답 지연 시간은 짧아야 합니다.

### 9)Object Storage
- 빅데이터는 대부분은 확장성이 높은 분산 스토리지에 저장
- 대량으로 파일을 저장하는 데이터베이스가 객체 스토리지
  - 파일을 읽고 쓰는 것은 네트워크를 거쳐서 실행
  - 여러 디스크에 복사되기 때문에 일부 하드웨어가 고장 나더라도 손실되지 않음
  - 데이터의 읽기 쓰기를 다수의 하드웨어 분산함으로써 데이터의 양이 늘어나도 성능이 떨어지는 일이 없도록 고안되어 있음
  - 객체 스토리지의 경우는 데이터의 양이 많을 때는 우수하지만 소량의 데이터에 대해서는 비효율적
  - 작은 파일을 읽고 쓰는 경우에 데이터 양에 비해 통신 오버헤드가 크기 때문
  - 블록 단위로 저장하는 이유
- AWS의 S3
  - 확장이 무제한적이고 데이터 복제가 기본적으로 이루어지기 때문에 높은 가용성을 보장
  - 기본적인 IOPS(초당 입출력 속도)는 SSD에 비해서 부족하지만 파티셔닝을 적용해서 애플리케이션에서 병렬 처리를 수행하면 월등하게 높은 성능을 보장
  - 단일 SSD는 단일 처리에는 높은 IOPS를 출력하지만 최대 IOPS는 SSD 하드웨어의 성능만큼만 가능하지만 분산 환경에 파티셔닝 되어 있는 객체 스토리지는 제약 없이 병렬로 입출력이 가능
- 시계열 데이터
  - 시간과 함께 생성되는 데이터
  - 시계열 데이터는 수시로 객체 스토리지에 기록을 하게 되는데 작은 파일이 대량으로 생성되기 때문에 시간이 지남에 따라 성능을 저하시키는 요인이 될 수 있음
  - 작은 데이터는 적당히 모아서 하나의 큰 파일로 만들어서 효율을 높여야 함
  - 파일의 크기를 무한정 크게 하는 것도 좋지 못함   
    파일의 크기가 너무 크면 네트워크 전송에 시간이 걸려서 예상치 못한 오류가 발생할 수 있음
- 데이터의 양이 많아지거나 파일의 사이즈가 커지면 csv, json을 Parquet나 ORC 등으로 변환해서 파일의 크기를 줄이고 읽고 쓰는 속도를 향상시킬 수 있음

### 10)안정적 데이터 관리
- 데이터 파티션
  - 데이터가 많을 때 나누어서 저장
  - 데이터를 여러 서버에 고르게 분산할 수 있는가?
  - 노드의 추가나 삭제 시 데이터의 이동을 최소화할 수 있는가?
- 데이터 다중화
  - 높은 가용성 과 안정성을 확보하기 위해서 데이터를 N개 서버에 비동기적으로 다중화할 필요가 있습니다.
  - AWS의 S3는 객체 스토리지의 파일을 3개로 복제해서 고가용성을 보장
- 데이터 일관성
  - 여러 노드에 다중화된 데이터는 적절히 동기화가 이루어져야 함
  - 정족수 합의 프로토콜(Quorum Consensus)을 이용(예: 3개 중 2개 이상 전달되어야 전달 성공으로)
- 메시지의 순서 보장
  - AWS 의 SQS는 기본적으로 FIFO 와 순서를 보장하는데 카프카는 FIFO를 지원하지 않음
  - 자바 메시징 서비스에서 사용하는 토픽은 FIFO를 지원하고 순서를 보장하지만 카프카는 별도의 토픽을 사용하고 1:N 개 소비자의 관계이므로 순서를 보장하는 것이 쉽지 않음
  - 분산 트랜잭션 처리를 자동화하기 위해서는 2단계 커밋을 이용   
    애플리케이션과 데이터베이스에서 2단계 커밋을 처리하기 위해서 추가적인 리소스를 사용   
    스프링에서는 트랜잭션의 격리 수준을 다양한 형태로 제공을 함
- 장애 감지
  - 하나의 서버가 장애를 알린다고 해서 바로 장애처리를 하는 것이 아니고 보통은 두 대 이상의 서버에서 장애가 발생했다고 알려주면 그 때 보통 장애가 발생되었다고 간주
  - 네트워크의 모든 호스트를 연결해서 판단하면 좋지만 실제로는 가십 프로토콜을 이용해서 장애를 감지
![](master_worker.png)
  - 워커노드가 서로 연결되어 있어야 어디서 문제가 발생했는지 알기 쉬움, 마스터노드에 문제가 생길 경우 워커노드끼리 연결되어 있지 않으면 문제를 찾기 쉽지 않음
  - 하나의 길에 문제가 생기면 돌아갈 방법이 없음, 연결되어 있으면 돌아갈 수 있음
  - 어떤 노드에서 발생했는지 쉽게 파악할 수 있음
- 일시적 장애를 처리하는 방법
  - 장애를 감지한 시스템은 가용성을 보장하기 위해서 필요한 조치를 취해야 합니다.
  - 서버에 장애가 발생하면 장애가 발생한 서버로 가는 요청을 다른 서버가 잠시 맡아서 처리하고 그 동안 발생한 변경 사항은 해당 서버가 복구되었을 때 일괄적으로 반영하여 데이터 일관성을 보존합니다.

## 3.프로메테우스
### 1)프로메테우스 바이너리 구성
- 클라우드 네이티브 구현을 위한 핵심 소프트웨어는 쿠버네티스이며 이는 런타임 실행 환경에 해당
- 프로메테우스는 CNCF에서 다양한 프로젝트가 진행 중이며 많은 기능을 포함하고 있음
- 운영자를 위한 매트릭 모니터링, 개발자를 위한 Exporter, AutoScaling 설정, 시계열 데이터베이스, 서비스 모니터를 사용한 서비스 디스커버리, 알람과 업무 규칙 등
- 서비스 디스커버리, 커스텀 매트릭 생성, 오토스케일링 등은 프로메테우스가 제공

```
                          |<-------------------------------Exporter
                          |                                    |
프로메테우스서버 -> 프로메테우스 어댑터 -> HPA -> Deployment -> Pod <- Kube Proxy
                        |                                                |
                        |                                            서비스모니터
                        |                                                |
                        |<-----------------------------------------프로메테우스 오퍼레이터
```
- Prometheus Operator
  - 쿠버네티스에서 프로메테우스를 구성할 때 자원 관리 와 프로비저닝을 수행
  - 쿠버네티스에서는 헬름이나 오퍼레이터로 프로메테우스를 설치할 수 있습니다.
  - 그라파나는 헬름 차트로 설치와 구성을 진행
  - 동적으로 증가하는 서비스 와 파드를 발견하는 역할
  - Service Monitor 와 Pod Monitor가 서비스 디스커버리 기능을 제공
  - 서비스 와 파드의 증감을 모니터링해서 증감 발생 시에 프로메테우스 구성을 파일을 업데이트

- Prometheus Exporter
  - 특정 메트릭을 수집해서 엔드포인트에 노출시키는 소프트웨어
  - 데이터베이스, 하드웨어, 메시지 시스템, 저장소 등 여러 시스템에 대한 Exporter, CollectD 등 기존의 서버 모니터링에 사용되는 에이전트 들과 통합해서 사용할 수 있는 Exporter 도 존재
  - 기본적으로 제공되는 Exporter 이외 도메인에 적합하게 개발된 마이크로서비스의 메트릭 측정을 하기 위해서 Customer Exporter를 개발할 수 있는 API(Application Programming Interface) 와 SDK(Software Development Kit)를 제공
- Prometheus Adapter
  - 커스텀 메트릭이 Exporter를 통해서 제공되면 이를 스크래핑해서 메트릭을 측정하고 부하가 증가하는 시스템에 대응하기 위해서 HPA를 통해서 파드를 오토스케일링

### 2)바이너리(설치 프로그램)를 이용한 설치
- 바이너리 파일을 다운로드: git clone https://github.com/itggangpae/prometheus.git

- 압축 해제   
`tar xvzf prometheus-2.28.1.linux-amd64.tar.gz`

- 명령 수행
  - 사용자 추가   
    `sudo useradd --no-create-home --shell /bin/false prometheus`<br>
  - 디렉터리 생성   
    `sudo mkdir /etc/prometheus`   
    `sudo mkdir /var/lib/prometheus`<br>
  - 생성한 디렉토리의 소유자를 변경   
    `sudo chown -R prometheus /etc/prometheus`   
    `sudo chown -R prometheus /var/lib/prometheus`<br>
  - 실행 관련된 파일을 아무 곳에서 실행할 수 있도록 복사
  ```
  sudo cp prometheus-2.28.1.linux-amd64/prometheus /usr/local/bin/

  sudo cp prometheus-2.28.1.linux-amd64/promtool /usr/local/bin/
  
  sudo cp -r prometheus-2.28.1.linux-amd64/consoles /etc/prometheus
  
  sudo cp -r prometheus-2.28.1.linux-amd64/console_libraries /etc/prometheus
  ```
- /etc/prometheus 디렉토리에 환경 설정 파일을 생성(prometheus.yaml)
```yaml {filename="prometheus.yaml"}
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
    - targets: ["localhost:9090"]


  - job_name: "node"
    static_configs:
    - targets: ["localhost:9100"]
```
- prometheus를 백그라운드에서 수행하기 위한 서비스 파일을 생성: /etc/systemd/system 디렉터리에 prometheus를 서비스로 실행하기 위한 prometheus.service 파일을 생성하고 작성
`sudo nano /etc/systemd/system/prometheus.service`
```{filename="prometheus.service"}
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
     --config.file /etc/prometheus/prometheus.yaml \
     --storage.tsdb.path /var/lib/prometheus \
     --web.console.templates=/etc/prometheus/consoles \
     --web.console.libraries=/etc/prometheus/console_libraries \
     --storage.tsdb.max-block-duration=1m \
     --storage.tsdb.min-block-duration=1m \
     --web.enable-lifecycle \
     --web.enable-admin-api \
     --log.level=info

[Install]
WantsBy=multi-user.target
```
- 실행
  - 새로운 service 파일을 추가했을 때 디렉토리를 다시 읽기 위한 명령   
    `sudo systemctl daemon-reload`
  - 서비스 시작   
    `sudo systemctl start prometheus`
  - IP주소:9090 으로 접속
- node exporter 파일 압축 해제
- 서비스 생성   
  `sudo nano /etc/systemd/system/node_exporter.service`
```
Description=Node-Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple

SyslogIdentifier=node-exporter
WorkingDirectory=/home/ubuntu/prometheus/node_exporter-1.3.1.linux-amd64

ExecStart=/home/ubuntu/prometheus/node_exporter-1.3.1.linux-amd64/node_exporter

[Install]
WantsBy=multi-user.target
```
- 실행
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```