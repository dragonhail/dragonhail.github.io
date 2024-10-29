---
title: Consumer
weight: 1
---
### Consumer Group


### 파티션 할당 알고리즘
- 키가 없으면 라운드 로빈
- 키가 있으면 키의 해쉬 값에 해당하는 파티션에 할당

### 중복 메시지 처리
- Consumed Offset (Current Offset): 컨슈머가 메시지를 어디까지 읽었는가를 나타낸다. 해당 오프셋을 통해 컨슈머가 읽어야 할 다음의 메시지 위치를 식별할 수 있다. 해당 오프셋은 컨슈머가 poll( )을 받을 때마다 자동으로 업데이트 된다. 해당 오프셋은 각각의 컨슈머가 관리
- Committed Offset : 컨슈머가 메시지를 읽고 카프카에게 ‘여기까지의 오프셋을 처리했다’ 는 것을 알리는 Offset Commit 을 통해 업데이트되는 오프셋이다. 컨슈머의 프로세스가 실패하고 다시 시작되면 컨슈머가 다시 메시지를 읽게 될 시작점이 되는 오프셋이기도 하다. 해당 오프셋은 __consumer_offsets 라고 하는 카프카의 내부 토픽에서 관리

### 트랜잭셔널 메시징(Transactional Messaging)
- 서비스 로직의 실행과 그 이후의 이벤트 발행을 원자적으로(atomically) 함께 실행하는 것
- 2가지
  - 트랜잭셔널 아웃박스 패턴 (Transactional Outbox Pattern)
  - 변경 데이터 캡쳐 (Change Data Capture)

### 리밸런싱 문제
- 한 컨슈머에서 다른 컨슈머로 소유권이 이전
