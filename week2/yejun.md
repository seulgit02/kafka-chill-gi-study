## Chapter 1. Meet Kafka
___

> 모든 애플리케이션은 데이터를 만들고, 그 데이터는 next thing을 알려줄 중요한 정보가 된다.
> 
> 예를 들면, 아마존에서 우리가 관심 있는 항목을 클릭하면, 추천 항목에 뜨게되며 모든 기업이 데이터로 움직인다는 것을 의미한다.
___

### Publish/Subscribe Messaging

- pub/sub 메시징은 데이터(메시지)의 발신자(publisher)가 특정 수신자(subscriber)에게 직접적으로 데이터를 보내지 않는 특징을 가진 패턴
- publisher는 메시지를 어떤 식으로든 분류하고, subscriber는 메시지를 수신하기 위해 구독(subscribe)하는 형태

pub/sub 사례는 단순한 메시지 큐 혹은 프로세스 간 통신 채널로 시작하는 것과 동일하다 

예를들어, 애플리케이션의 메트릭을 수집하는 서버가 있고 애플리케이션이 있다면 간단하게 서로 `직접 연결`하여 메트릭을 push하고 모니터링할수 있다.

추후에 더 많은 애플리케이션으로부터 메트릭을 수집하거나, 데이터를 분석하는 등의 새로운 서비스를 시작하게 된다면 애플리케이션에 직접 연결을 다시 수행해야하며 점점 많은 연결이 생기게되고 이는 추적을 어렵게 만든다.
<p align="center"><img width="966" alt="image" src="https://github.com/user-attachments/assets/cac04aa5-d90c-45d2-b780-7041badf8278" /></p>

여기서 생기는 기술 부채는 너무 명확하여 이를 해결하기 위해 다음과 같이 구축할 수 있다.
- 모든 애플리케이션으로부터 측정 지표를 받는 단일 애플리케이션을 설정하고, 필요한 모든 시스템이 해당 측정 지표를 쿼리할 수 있는 서버를 제공
<p align="center"><img width="966" alt="image" src="https://github.com/user-attachments/assets/1b4c39f9-e56b-45e1-b6ee-2cffc269aac8" /></p>

아키텍처 복잡성을 줄였으며 이는 publish/subscribe messaging system을 구축한 것이다.
___

### Individual Queue Systems

메트릭, 로깅, 사용자 추적 3개의 job에 대하여 pub/sub system을 구축

3개의 시스템으로 분리하였지만 상황에 따라 중복되어 메시지 큐가 쌓일 수 있으며, 위에서 살펴본 상황과 결국 동일한 상황에 놓이게 될 것임 
-> 서비스가 확장하면서 더 다양한 지표를 얻고자 한다면 중복되는 pub/sub system이 확장될 것

비즈니스가 확장되어도 single centralized system을 구축하고 싶음
___

### Enter Kafka

아파치 카프카가 위에서 설명하는 문제를 해결하는 publish/subscribe messaging system으로 개발되었음
-> `분산 커밋 로그` 또는 `분산 스트리밍 플랫폼`이라고 부름

파일 시스템 또는 데이터베이스 커밋 로그는 모든 트랜잭션의 지속적인 기록을 제공하여 시스템의 상태를 일관되게 구축하기 위해 재생될 수 있도록 설계된 것 같이
카프카 내의 데이터는 내구성과 순서에 맞게 저장되고, 데이터는 시스템 내에서 분산되어 장애에 대한 추가적인 보호와 확장에 유연한 성능을 보인다.
___

### Messages and Batches

- 카프카 내의 데이터 단위는 메시지 (DB로 따지면 행, 레코드와 유사)
- 카프카에 관한 메시지는 단순히 바이트 배열이므로, 그 안에 포함된 데이터는 카프카에 특정 형식이나 의미를 가지지 않음
- 메시지는 메타데이터를 선택적으로 키값으로 설정할 수 있음
  - 키 값은 파티션에 기록될 때 사용되는데, 키 값을 해시하여 토픽의 총 파티션 수로 나눈 몫으로 메시지의 파티션 번호를 선택함
- 효율성을 위해 메시지는 카프카에 배치 형식으로 저장되며, 배치는 동일한 토픽 및 파티션에 생성되는 메시지 모음이다.
___

### Schemas

- 메시지는 카프카에선 바이트 배열이지만, 쉽게 이해하기 위한 스키마를 적용하는 것이 좋다.
- 애플리케이션에 따라 메시지 스키마엔 다양한 옵션이 존재하며 JSON, XML이 사람이 읽기엔 좋음
  - 카프카 개발자는 원래 하둡용 직렬화 프레임워크인 Apache Avro 사용을 선호
- `Avro`는 컴팩트한 직렬화 형식, 메시지 페이로드와 분리된 스키마(변경 시 코드를 생성할 필요가 없음), 이전 버전과의 호환성 및 이후 버전과의 호환성을 모두 지원
- 메시지 읽기, 쓰기를 분리하기 위해선 일관된 데이터 형식이 중요
___

### Topics and Partitions

카프카의 메시지는 토픽으로 분류되며 토픽은 데이터베이스 테이블 또는 파일 시스템의 폴더라고 표현할 수 있다.

- 토픽은 추가적으로 여러 파티션으로 나뉘며, 커밋 로그로 말하면 파티션은 단일 로그를 말하고, 이런 파티션들을 순서대로 읽는방식
- 파티션은 여러 서버에 호스팅될 수 있으며, 단일 토픽이 여러 서버에 걸쳐 수평적으로 확장되어 단일 서버의 능력보다 훨씬 뛰어난 성능을 제공할 수 있음을 의미한다 또한, 파티션을 복제하여 한 서버가 실패할 경우 다른 서버가 동일한 파티션의 복사본을 저장할 수 있다.
- `스트림`이라는 용어는 카프카에서 파티션 수에 관계없이 단일 데이터 토픽으로 간주함
- pub to sub으로 이동하는 단일 스트림은 하둡과같이 대량 데이터에서 작동하도록 설계된 방식과 비교할 수 있음
<p align="center"><img width="558" alt="image" src="https://github.com/user-attachments/assets/aa61721f-d290-41d2-9d5a-5e33aa236cf7" /></p>
___

### Producers and Consumers

카프카 클라이언트는 두 가지: producers and consumers (producer와 consumer를 기본요소로 사용하는 advanced client로 카프카 커넥트 API와 카프카 스트림 등이 있음)

- Producer는 메시지를 생성하는 사람으로 publisher 혹은 writer
  - 메시지는 하나의 토픽으로 생성되며, producer는 토픽의 파티션에 메시지를 균등 분산하고자 함 (해싱을 이용한 파티션 분산 -> 키 충돌에 의한 쏠림이 발생할 수 있으며 RR 방식도 있다고 한다.)
- Consumer는 메시지를 읽는 사람으로 subscriber 혹은 reader
  - Consumer는 하나 이상의 토픽을 subscribe하고, 각 파티션에 생성된 토픽은 순서대로 읽는다 -> 순서대로 읽기위해 Offset을 가지며, offset을 통해 순서를 지켜 읽을 수 있고, 장애 시에도 끊긴 부분부터 읽을 수 있다.
- Consumer는 토픽을 함께 처리하기 위해 consumer group으로 작동하기도 함
  - 많은 메시지가 있는 토픽을 소비하기 위해서 consumer group에 consumer를 수평적으로 확장할 수 있음
  - consumer group의 consumer가 파티션에서 작업 중이면 다른 consumer는 접근하지 못하고 이를 ownership of the partition by the consumer 이라고 함. (파티션 소유권)
  - 파티션에서 작업 중이던 consumer가 장애로 인해 실패하게 되면 다른 consumer에 파티션을 재할당하여 처리
___

### Brokers and Clusters

단일 카프카 서버를 Broker라고 부른다.
- 브로커는 producer한테 메시지를 받아 오프셋 할당 및 디스크에 write
- consumer에게 서비스 형태로 파티션에 대한 요청을 응답
- 단일 브로커는 H/W 성능에 따라 수천개 파티션과 초당 수백만개 메시지를 쉽게 처리할 수 있음

클러스터 환경에서 하나의 브로커는 클러스터 컨트롤러로 작동
- 컨트롤러가 브로커에 파티션을 할당하고 장애를 모니터링 하는 등의 작업을 함
- 파티션은 브로커에 할당되며 이를 리더 브로커라하고, 파티션을 복제하여 유지하는 팔로워 브로커가 존재함
  - 복제하여 파티션의 메세지 중복 저장
  - 리더 브로커에 장애가 발생 시, 팔로워 중 하나가 리더 역할 이어 받음
  - 모든 producer는 리더 브로커에 메세지를 발행해야하지만, consumer는 리더나 팔로워 중 하나로 데이터 읽기가 가능

<p align="center"><img width="886" alt="image" src="https://github.com/user-attachments/assets/72d1f2b7-ca9b-44b9-9b95-ad02e5033872" /></p>
___

### Multiple Clusters
여러 방식의 클러스터 운영

- Segregation of types of data
- Isolation for security requirements
- Multiple datacenters (disaster recovery)
  - [토스증권 Apache Kafka 데이터센터 이중화 구성](https://toss.tech/article/kafka-distribution-1)
___

### Multiple Producers

- 여러 Producer가 동일한 토픽을 보내든, 다른 토픽을 보내든 Consumer는 각 Producer에 따라 다른 방식을 조정할 필요없이 수신할 수 있음
___

### Multiple Consumers

- 다른 consumer가 메시지를 소비하고있으면 사용할 수 없는 다른 큐 시스템과 다르게 Multiple Kafka consumer는 group으로도 작동하고, 스트림을 공유할 수도 있다.
___

### Disk-Based Retention

카프카는 지속적인 메시지 보존으로 항상 실시간으로 작동할 필요가 없음
- 메시지는 디스크에 저장되며, consumer가 느린 처리와 트래픽 문제로 인한 데이터 손실 위험이 없음
- consumer는 작업이 중단되어도 메시지는 카프카에 보존되어 이어서 작업이 가능

___

### Scalable
카프카의 유연한 확장성은 모든 양의 데이터를 쉽게 처리할 수 있다.

사용자는 단일 브로커로 시작하여, 3개의 브로커로 구성된 클러스터로 확장하고, 데이터가 확장됨에 따라 시간이 지남에 따라 성장하는 수십 또는 수백 개의 브로커로 구성된 더 큰 프로덕션 클러스터로 이동할 수 있음

클러스터의 확장도 온라인에서 가능하고, 여러 브로커로 구성된 클러스터가 개별 브로커의 실패를 처리하고 클라이언트에게 계속 서비스를 제공할 수 있음

- 더 많은 동시 실패를 대비하기 위해선 더 많은 복제 요소를 구성해야 함
___

### Platform Features
스트리밍 플랫폼 기능
- `카프카 커넥트`는 소스 데이터 시스템에서 데이터를 가져와 카프카로 푸시하거나, 카프카에서 데이터를 가져와 싱크 데이터 시스템으로 푸시하는 작업을 지원
- `카프카 스트림`은 확장 가능하고 오류에 강한 스트림 처리 애플리케이션을 쉽게 개발할 수 있는 라이브러리를 제공
___

### The Data Ecosystem
많은 애플리케이션이 데이터 처리를 위해 구축하는 환경
- Activity tracking
  - 페이지 조회 및 클릭 추적과 같은 수동적인 정보, 사용자가 자신의 프로필에 추가하는 정보와 같은 더 복잡한 작업 등의 활동 추적
  - 보고서를 생성하거나, 머신 러닝 시스템에 공급하거나, 검색 결과를 업데이트하는데 사용됨
- Messaging
  - 애플리케이션이 사용자에게 알림(이메일)을 보내야 하는 메시징에도 사용 
- Metrics and logging
  - 시스템 측정 지표와 로그를 수집
  - 모니터링 및 경고를 위한 시스템에서 소비
  - 모니터링하는 대상 시스템의 업데이트가 발생해도, 로그를 수집하는 front에선 변경할 것이 없음 (format 유지)
- Commit log
  - 데이터베이스 변경 사항을 카프카에 published
  - 변경 로그를 보존하여 장애 시 재수행할 수 있도록함
- Stream processing
  - 스트림 처리는 메시지가 생성되는 즉시 실시간으로 데이터에 대해 작동
  - 측정 지표 계산, 다른 애플리케이션의 효율적인 처리를 위한 메시지 분할 또는 여러 소스의 데이터를 사용하여 메시지 변환과 같은 작업을 수행

___

### The Birth of Kafka

카프카의 주요 목표
- Decouple producers and consumers by using a push-pull model
- Provide persistence for message data within the messaging system to allow multiple consumers
- Allow for horizontal scaling of the system to grow as the data streams grew
- Optimize for high throughput of messages

___

## CHAPTER 3. Kafka Producers: Writing Messages to Kafka
