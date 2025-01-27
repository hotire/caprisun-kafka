# 컨슈머 중요 개념
## 컨슈머와 컨슈머 그룹
* 메시지를 쓰는 속도와 처리하는 속도가 다를 수 있음
* 특히 쓰는 속도가 빠르다면 문제가 발생할 수 있음
* 다수의 컨슈머들이 같은 토픽의 메시지를 분담해서 읽을 수 있도록 위해 컨슈머 그룹을 사용
* 다수의 컨슈머가 같은 토픽을 소비하면서 해당 토픽의 다른 파티션을 분담할 수 있다.
* 각기 다른 애플리케이션마다 컨슈머 그룹을 생성할 수 잇다.

## 컨슈머 그룹과 리밸런싱
* 컨슈머 그룹에 컨슈머가 추가되거나 컨슈머에 문제가 생길 수 있음, 그럼 파티션 재할 당이일어남
* 한 컨슈머로부터 다른컨슈머로 파티션 소유권을 이전하는 것을 리밸런시잉라고 함
* 문제는 리밸런싱을 하는동안 컨슈머들은 메시지를 읽을 수 없음
* 컨슈머 그룹 전체가사용 불가 상태가 됨
* **그룹 조정자**로 지정된 브로커에게 컨슈머가 **하트비트**를 전송하여 파티션 소유권을 유지할 수 있음 
* **하트비트**는 컨슈머가 **폴링** 또는 메시지를 **커밋**할 때 자동 전송
* 하트비트 전송 세션 타임아웃이 지나거나 문제가 생긴다면 컨슈머가 중단된 것으로 간주하고 리밸런싱이 일어남
* 리밸런스는 토픽에 변경 사항이 생기거나 컨슈머 그룹에 새로운 컨슈머가 추가되거나 컨슈머 그룹에 속해 있던 컨슈머가 제외되는 경우에 발생한다.
* [Naver D2](https://d2.naver.com/helloworld/0974525)

# 컨슈머 생성하기
* KafkaProducer 인스턴스 생성과 유사
* 기본 속성으로는 bootstrap.server, key.deserializer, value.deserializer 가 필요
* deserializer 는 값의 역직렬화를 위해 필요
* group.id는 컨슈머 그룹을 나타냄

# 폴링 루프
* 컨슈머 api의 핵심은 서버로부터 연속적으로 많은 데이터를 읽기 위해 **폴링**하는 루프에 있음
* poll() 메서드의 매개변수는 데이터 도착하기를 기다리는 시간이다.
* 컨슈머 종료시 close()를 실행해야 한다. 그럼 곧바로 리밸런싱이 일어남
* 새로운 컨슈머에서 최초 poll() 호출시 그룹조정자를 찾고, 컨슈머 그룹에 추가시키며, 파티션 내역을 받는 작업이 일어남 

# 컨슈머 구성하기
* fetch.min.bytes : 브로커로부터 받기 원하는 데이터의 최소량 (default: 1bytes)
* fetch.max.wait.ms : 데이터가 모일 때까지 기다리는 시간 (default : 500ms)
* max.partition.fetch.bytes : 파티션당 반환하는 최대 바이트수 (default : 1MB)
* session.timeout.ms : 컨슈머가 브로커와 연결이 끊기는 시간, 하트비트 전송이 그룹조정자에게 해당 매개변수 일어나지 않으면 컨슈머가 종료된 것으로 간주 (default : 10000ms)
* heartbeat.interval.ms : 하트비트를 전송하는 시간 간격 (default : 3000ms)
* auto.offset.reset : 커밋되지 않은 오프셋이나 유효하지 않는 오프셋을 컨슈머가 요청할 때 어떤 레코드를 읽게 할지에 대한 설정 값, 유효한 오프셋이 없다고 판단하여 최근 레코드들을 선택하는 'latest', 맨 앞부터 다시 읽는 'earliest'가 있다.  (default : latest)
* enable.auto.commit : 컨슈머가 오프셋 커밋을 자듕으로 할 것인지 (default : true)
* partition.assignment.strategy : 파티션과 컨슈머의 할당 전략, Range와 RoundRobin이 기본적으로 존재하고 사용자가 구현 가능 (default : Range)
* client.id : 클라이언트로 부터 전송된 메시지 식별용 값
* max.poll.records : 한 번의 poll() 메서드 호출에서 반환되는 레코드의 최대 개수 (default : 500)
* receive.buffer.bytes, send.buffer.bytes : tcp 송수신 버퍼 크기
* max.poll.interval.ms : rebalance Timeout 이내에 JoinGroup 요청을 보내야 하는데 그 시간을 설정 , HeartBeat 스레드가 현재 시간과 마지막으로 poll 메서드가 호출된 시간의 차이를 계산하여 설정값 보다 큰 경우 컨슈머 그룹 탈퇴(default : 300000)

# 커밋과 오프셋
* 각 컨슈머는 파티션볼로 자신이 읽은 레코드의 현재 **위치(오프셋)** 를 추적 관리 가능
* 파티션 내부의 현재 위치를 변경하는 것을 **커밋** 이라고 한다.
* 기존 컨슈머가 비정상적으로 종료되거나 새로운 컨슈머가 그룹에 추가 된다면 리밸런싱을 유발한다.
* 리밸런싱 후 마지막으로 커밋된 오프셋과 현재 읽고 처리중인 메시지가 다를경우 메시지가 두번 처리 되거나, 누락될 수 있음

## 자동 커밋
* 일정 시간마다 마지막으로 호출된 poll()메서드에서 반환된 오프셋을 커밋한다.
* 다만 리밸런싱 후 모든 컨슈머는 마지막으로 커밋된 오프셋부터 레코드를 읽기 시작하므로 마지막 커밋 이후 읽었던 레코드는 두번 처리 된다.
* poll() 메서드에서 반환된 모든 메시지는 다시 poll()을 호출하기 전에 끝나도록 하는게 중요

## 현재의 오프셋 커밋하기
### commitSync()
* poll() 에서 반환된 가장 최근의 오프셋을 커밋
* 동기적으로 실행, 브로커가 커밋 요청에 응답할떄까지 일시중지
* 성공ㅇ하거나 재시도 불가능한 에러가 생길 때까지 커밋을 재시도 가능

### 비동기 커밋(commitAsync())
* 재시도 하지 않음
* 비동기 방식이기 떄문에 커밋 순서 보장 X
* 콜백 사용 가능
* 재시도 설정하기가 까다로움

### 동기와 비동기 커밋 동시 사용
* 일반적인 처리에는 비동기 커밋, 컨슈머를 닫을 때는 동기 커밋 사용

## 특정 오프셋을 커밋하기
* commitSync, commitAsyncs는 기본적으로 마지막으로 반환된 오프셋을 커밋
* 커밋을 원하는 파티션과 오프셋을 저장한 Map 인자 전달 가능

# 리밸런싱 리스너
* 컨슈머는 종료되기 전이나 파티션 리밸런싱이 시작되기 전에 클린업하는 처리를 해야 한다.
* 마지막 메시지의 오프셋 커밋, 리소스 정리 등이 필요
* subscribe()를 할 때 ConsumerRebalanceListener인터페이스 구현체 전달 가능
* onPartitionsRevoked(Collection<TopicPartition> partitions)
  * 리밸런싱이 시작되기 전에, 컨슈머가 메시지 소비를 중단한 후 호출된다.
  * 오프셋을 커밋해야 하는 곳
* onPartitionsAssigned(Collection<TopicPartition> partitions)
  * 파티션이 브로커에게 재할당된 후에, 컨슈머가 파티션을 새로 할당 받아 메시지 소비를 시작하기 전에 호출
