# 1주차

범위: 1-3장 + 인프런 강의(카프카 완벽 가이드 - 코어편)

## 목차

### Ch1. Kafka 시작하기
- [🌳 Kafka란 무엇인가?](#-kafka란-무엇인가)
- [🌳 Kafka의 주요 개념](#-kafka의-주요-개념)
- [🌳 Kafka의 주요 특징 및 장점](#-kafka의-주요-특징-및-장점)
- [🌳 Kafka의 역사](#-kafka의-역사)

### Ch2. Kafka 설치하기
- [🌳 Broker 구성하기](#-broker-구성하기)
- [🌳 Topic 기본 설정(default)](#-topic-기본-설정default)
- [🌳 하드웨어 선택하기](#-하드웨어-선택하기)
- [🌳 Kafka Cluster 구성](#-kafka-cluster-구성)
- [🌳 프로덕션 환경 고려사항](#-프로덕션-환경-고려사항)

### Ch3. Kafka Producer: 카프카에 메시지 쓰기
- [🌳 프로듀서 개요](#-프로듀서-개요)
- [🌳 프로듀서 기본 구조](#-프로듀서-기본-구조)
- [🌳 필수 설정](#-필수-설정)
- [🌳 메시지 전송 방식](#-메시지-전송-방식)
- [🌳 주요 프로듀서 설정](#-주요-프로듀서-설정)
- [🌳 시리얼라이저와 스키마](#-시리얼라이저와-스키마)
- [🌳 파티션과 접착성 처리](#-파티션과-접착성-처리)
- [🌳 레코드 헤더](#-레코드-헤더)
- [🌳 인터셉터](#-인터셉터)
- [🌳 쿼터 및 쓰로틀링](#-쿼터-및-쓰로틀링)

### 카프카 완벽 가이드 - 코어편
- [🍅 Kafka 소개, 그리고 왜 Kafka를 배워야 하는가?](#-kafka-소개-그리고-왜-kafka를-배워야-하는가)
- [🍅 Kafka 실습 환경 구축](#-kafka-실습-환경-구축)
- [🍅 Kafka 설치 Binary 개요](#-kafka-설치-binary-개요)
- [🍅 Oracle Virtualbox 설치](#-oracle-virtualbox-설치)
- [🍅 Kafka 설치하기](#-kafka-설치하기)
- [🍅 Kafka 구성하기](#-kafka-구성하기)
- [🍅 Topic 생성하기](#-topic-생성하기)
- [🍅 Topic 생성 및 정보 확인하기](#-topic-생성-및-정보-확인하기)
- [🍅 kafka-console-producer와 kafka-console-consumer로 Producer와 Consumer 실습](#-kafka-console-producer와-kafka-console-consumer로-producer와-consumer-실습)
- [🍅 Producer의 객체 직렬화(Serializer) 전송의 이해](#-producer의-객체-직렬화serializer-전송의-이해)
- [🍅 Key 값을 가지지 않는 메시지 전송](#-key-값을-가지지-않는-메시지-전송)
- [🍅 Key 값을 가지는 메시지 전송](#-key-값을-가지는-메시지-전송)
- [🍅 여러 개의 파티션을 가지는 메시지 전송 실습](#-여러-개의-파티션을-가지는-메시지-전송-실습)
- [🍅 Kafka Producer의 send() 메소드 호출 프로세스](#-kafka-producer의-send-메소드-호출-프로세스)
- [🍅 key 값을 가지지 않는 메시지 전송 시 파티션 분배 전략 - Sticky 파티셔닝](#-key-값을-가지지-않는-메시지-전송-시-파티션-분배-전략---sticky-파티셔닝)

<br> <br>

# Ch1. Kafka 시작하기

MSA 적용 프로젝트가 늘어남에 따라, Kafka 적용도 증가하고 있다.
![image](https://github.com/user-attachments/assets/79edce5a-24e7-4427-80e3-b3be2769643a)


## 🌳 Kafka란 무엇인가?

- pub/sub 메시지 전달 시스템
- 메시지 큐를 관리하며, 브로커를 통해 데이터를 발생자와 구독자 간에 중계
- 데이터 생성과 처리를 분리하고 효율적을 유연하게 관리

### 🌿 Pub/Sub 메시징의 필요성

- 전통적인 방식은 발행자와 구독자가 직접 연결되어 유지보수 비용이 높고, 확장성이 낮다.
- 발행/구독 모델은 데이터 흐름을 중앙 집중화하여 효율성과 확장성을 높임
- Kafka는 발행/구독 모델을 효과적으로 구현해 안정성과 성능을 제공
    - 데이터를 순서대로 지속적으로 저장
    - 데이터를 분산 저장하여 장애 보호와 성능 확장 가능

## 🌳 Kafka의 주요 개념

![image](https://github.com/user-attachments/assets/caa736d1-e5f0-43dc-b9a3-932d7a7d39cd)

### 🌿 메시지와 배치

- Kafka의 기본 대아토 단위는 메시지(message)
- 메시지는 키(key)와 값(value)로 구성되며, 효율적인 처리를 위해 메시지를 배치(batch) 단위로, 한 번에 묶어서 전송
- 배치 처리는 네트워크 I/O를 줄이고 처리 효율을 높임

### 🌿 토픽과 파티션

- 토픽(Topic) : 데이터 스트림을 나타내는 논리적 단위
    
    > Topic은 Partition으로 구성된 일련의 **로그** 파일
    > 
    > 
    > ![image](https://github.com/user-attachments/assets/eb2006c7-5899-4239-b6cf-87f6a9059e1b)

    > 
    - RDBMS의 Table과 유사한 기능
    - Topic은 Key와 Value 기반의 메시지 구조이며, Value로 어떤 타입의 메시지도 가능(문자열, 숫자값, 객체, Json, Avro, Protobuf 등)
    - Topic은 시간의 흐름에 따라 메시지가 **순차적으로** 물리적인 파일에 write된다.
- 파티션(Partition) : 물리적으로 데이터를 저장하는 단위(하나의 큰 토픽을 여러 개의 작은 부분으로 나눈 것)
    
    > Partition은 Kafka의 **병렬 성능과 가용성 기능의 핵심 요소**이며, 메시지는 병렬 성능과 가용성을 고려한 개별 파티션에 분산 저장된다.
    > 
    - 개별 파티션은 정렬되고, 변경 할 수 없는(immutable) 일련의 레코드로 구성된 로그 메시지
    - 개별 레코드는 `offset`으로 불리는 일련 번호를 할당 받음
    - 파티션 내에서는 메시지 순서 보장
    - 토픽 전체에서는 순서 보장하지 않음
- 파티션은 여러 서버에 분산 저장되어 확장성과 가용성을 제공 (복제도 파티션 단위로 이루어짐)
- 아래는 `replication-factor=2` 의 클러스터 구성이다.
    - 각 파티션은 2개의 브로커에 저장된다. (원본 1개 + 복제본 1개)
    - Producer는 메시지를 Leader로만 보내고, Follower는 그걸 복제한다. 그리고 Leader의 Broker가 망가졌을 때, Follwer가 Leader가 된다.
    - 한 개의 브로커가 실패해도 데이터를 잃지 않는다.
    - ※ replication-factor 수 이상의 broker가 있어야 한다.

![image](https://github.com/user-attachments/assets/48139638-58f4-406d-9ef5-7490d85d63b5)


### 🌿 생산자(Producer)

- 메시지를 생성해 특정 토픽에 기록
- 파티셔너(partitioner)를 통해 데이터를 파티션에 분배
    - 키 값을 해싱해 특정 파티션에 매핑가능
    - 동일 키 값을 가진 메시지는 같은 파티션에 저장돼 순서 보장
- 커스텀 파티셔너를 구현해 데이터 분배 방식 변경 가능
- Producer는 성능/로드밸런싱/가용성/업무 정합성등을 고려하여 어떤 브로커의 파티션으로 메시지를 보내야 할지 **전략적**으로 결정됨

### 🌿 소비자(Consumer)

- 메시지를 읽고 처리하는 역할
- 메시지를 읽어가더라도, 여전히 메시지는 보존되어 있다.
- 하나 이상의 토픽을 구독하여 데이터를 순차적으로 가져옴
- 오프셋(offset)을 기록해 읽은 위치를 관리
    - 자동 또는 수동으로 오프셋 관리 가능
    - 필요 시 이전 오프셋으로 이동해 메시지 재처리 가능
- 여러 개의 Consumer들로 구성될 경우 어떤 브로커의 파티션에서 메시지를 읽어들일지 **전략적**으로 결정함

### 🌿 브로커(Broker)와 클러스터(Cluster)

- **브로커**: 메시지를 보관하고 관리하는 Kafka 서버
    
    > 학교 도서관의 사서 선생님과 비슷하게, 책(메시지)를 받아서, 적절한 책장(토픽)에 분류하고 정리해두었다가, 누군가 필요할 때 찾아서 빌려준다. 브로커는 도서관 건물!
    > 
- **클러스터**: 여러 브로커로 구성된 카프카 단위(여러 브로커가 함께 일하는 그룹)
    
    > 하나의 큰 학교에 여러 도서관이 있고, 각 도서관마다 사서 선생님들이 협력해서 일하는 것과 같다. 만약 한 도서관(브로커)에 문제가 생겨도 다른 도서관이 대신 책을 빌려줄 수 있어서 항상 서비스가 유지된다. 또한 학생이 많아지면(데이터가 증가하면) 도서관을 더 지어서(브로커 추가) 대응할 수 있다.
    > 
    - **컨트롤러 브로커**: 파티션 할당과 브로커 관리 담당
    - **파티션 리더**: 파티션 데이터를 읽고 쓰는 작업 처리
    - **팔로워**: 리더 데이터를 복제해 장애 발생 시 리더 역할 수행
- 리더-팔로워 구조를 통해 데이터 복구 및 가용성 확보

## 🌳 Kafka의 주요 특징 및 장점

- **다중 프로듀서 및 컨슈머 지원**
    - 여러 프로듀서와 컨슈머를 동시에 지원
    - 컨슈머 그룹으로 데이터를 병렬 처리 가능
- **높은 확장성**
    - 브로커 추가 및 파티션 분배로 처리량 확장 가능
- **디스크 기반 데이터 보존**
    - 데이터를 지속적으로 저장하여 장애 상황에서도 데이터 복구 가능

## 🌳 Kafka의 역사

- LinkedIn에서 개발
- 처음에는 메트릭 모니터링과 사용자 활동 추적 시스템의 문제 해결을 위해 설계
- 2010년 오픈소스로 공개, 2012년 Apache 프로젝트로 승격
- 현재 Netflix, Uber 등 많은 기업의 데이터 파이프라인에 사용됨

# Ch2. Kafka 설치하기

> 실습은 아래 강의 핸즈온으로 대체
> 
- Kafka는 클러스터와 consumer 클라이언트 세부 정보에 대한 메타데이터를 저장하기 위해 Zookeeper를 사용한다.
- Zookeeper는 구성 정보 유지, 이름 지정, 분산 동기화 및 그룹 서비스 제공을 위한 중앙 집중식 서비스이다.

## 🌳 Broker 구성하기

### 🌿 broker.id

- 브로커를 식별하는 고유 정수 값
- 클러스터 내 [`broker.id`](http://broker.id) 는 중복 불가

### 🌿 listeners

- 브로커가 클라이언트와 통신하기 위한 리스너 설정
- 프로토콜, 호스트 이름, 포트를 정의
- [`listener.security.protocol.map`](http://listener.security.protocol.map) 으로 리스너와 프로토콜 매핑 필요
- 형식 : `listeners={프로토콜}://{호스트명}:{포트}`

```bash
# 단일 리스너 설정
listeners=PLAINTEXT://localhost:9092
listener.security.protocol.map=PLAINTEXT:PLAINTEXT

# 다중 리스너 설정 (쉼표로 구분)
listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL

# 모든 네트워크 인터페이스에서 연결 허용
listeners=PLAINTEXT://0.0.0.0:9092
listener.security.protocol.map=PLAINTEXT:PLAINTEXT
```

### 🌿 zookeeper.connect

- 브로커의 메타데이터가 저장되는 zookeeper의 위치를 정의
- 형식: `zookeeper.connect={호스트}:{포트}`

```bash
# localhost의 2181번 포트에서 실행 중인 주키퍼 사용
zookeeper.connect=localhost:2181

# 원격 주키퍼 서버 지정
zookeeper.connect=remote-zookeeper-host:2181

# 다중 주키퍼 환경에서 쉼표로 여러 서버 지정
zookeeper.connect=zk1:2181,zk2:2181,zk3:2181
```

### 🌿 log.dirs

- 로그를 저장할 디렉토리 경로를 설정

```bash
# 단일 디렉토리 설정
log.dirs=/var/lib/kafka-logs

# 다중 디렉토리 설정 (쉼표로 구분)
log.dirs=/var/lib/kafka-logs1,/var/lib/kafka-logs2
```

### 🌿 **num.recovery.threads.per.data.dir**

- 로그를 관리하는 스레드 수를 설정
- 설정된 값은 디렉토리당 스레드 수를 의미

```bash
# 디렉토리당 4개의 스레드 사용
num.recovery.threads.per.data.dir=4

# Tip : 디렉토리가 3개라면 총 스레드 수는 4 x 3 = 12
```

### 🌿 auto.create.topics.enable

- 브로커는 다음 상황에서 토픽을 자동 생성 (default `true` )
    - 프로듀서가 토픽에 메시지를 쓰기 시작할 때
    - 컨슈머가 토픽으로부터 메시지를 읽기 시작할 때
    - 클라이언트가 토픽에 대한 메타데이터를 요청할 때
- 명시적으로 토픽을 관리하려면 `false` 로 설정

```bash
# 토픽 자동 생성 활성화 (기본값)
auto.create.topics.enable=true

# 토픽 자동 생성 비활성화
auto.create.topics.enable=false
```

### 🌿 delete.topic.enable

- 토픽 삭제 가능 여부를 설정
- 삭제 요청 시 데이터와 메타데이터가 모두 제거됨

```bash
# 토픽 삭제 허용
delete.topic.enable=true

# 토픽 삭제 금지
delete.topic.enable=false
```

## 🌳 Topic 기본 설정(default)

### 🌿 **num.partitions**

- 새로운 토픽 생성 시 **파티션 수**를 결정
- **기본값:** `1`

```
# 파티션 수 기본값 설정
num.partitions=1

# 파티션 수를 10으로 설정
num.partitions=10

```

- 10개의 파티션이 10개의 브로커에 분산되면 각 브로커에 하나의 파티션 리더 배치
- 병렬 처리 성능이 최적화되고 처리량 증가

### 🌿 **default.replication.factor**

- 자동 생성된 토픽의 복제 팩터(레플리카 수)를 결정
- **복제 팩터:** 토픽 데이터가 복제되는 브로커의 개수

```
# 복제 팩터 기본값 설정
default.replication.factor=3

```

- 복제 팩터는 클러스터의 브로커 수보다 작거나 같아야 함
- **`min.insync.replicas`** 설정값보다 **최소 1 이상** 크게 설정하는 것을 권장

### 🌿  **log.retention.ms**

- 메시지의 **보존 기간**을 설정
- 기본값: `7일(604,800,000ms)`

```
# 메시지 보존 기간을 7일로 설정 (기본값)
log.retention.ms=604800000

# 메시지 보존 기간을 3일로 설정
log.retention.ms=259200000

```

### 🌿 **log.retention.bytes**

- 메시지 보존 기준을 메시지 크기(바이트)로 설정
- 설정된 용량을 초과하면 가장 오래된 메시지부터 삭제

```
# 메시지 보존 크기를 1GB로 설정
log.retention.bytes=1073741824

# 메시지 보존 크기를 500MB로 설정
log.retention.bytes=524288000

```

### 🌿 **log.segment.bytes**

- **로그 세그먼트 크기**를 설정
- 설정된 크기에 도달하면 기존 세그먼트를 닫고, 새로운 세그먼트 파일을 생성

```
# 세그먼트 크기를 1GB로 설정
log.segment.bytes=1073741824

```

### 🌿 **log.roll.ms**

- **로그 세그먼트 파일이 닫히는 시간 기준**을 설정

```
# 로그 세그먼트를 1시간(3600000ms)마다 닫음
log.roll.ms=3600000
```

### 🌿 **min.insync.replicas**

- 최소 동기화 레플리카 수 설정
- 설정된 값만큼의 레플리카가 최신 상태여야 쓰기 작업 성공

### 🌿 **message.max.bytes**

- **메시지 크기 상한**을 설정하여 브로커가 허용하는 최대 메시지 크기를 정의
- **기본값:** `1MB`
- 메시지 크기를 초과하면 브로커는 메시지를 거부하고 오류를 반환
- 프로듀서(`max.request.size`)와 컨슈머(`fetch.message.max.bytes`) 설정 값과 일치해야 함

## 🌳 하드웨어 선택하기

### 🌿 디스크 처리량

- 메시지를 디스크에 기록할 때 디스크 처리량이 쓰기 지연에 영향을 미침
- 고속 디스크 사용 권장

### 🌿 디스크 용량

- 메시지 보존 기간 또는 보존 용량에 따라 디스크 크기를 설정해야 함
- ex) 하루 1TB 트래픽, 일주일 보존 → 7TB + 10% 여유 공간

### 🌿 메모리

- 페이지 캐시 활용 : 더 많은 시스템 메모리가 있으면 페이지 캐시를 위해 사용할 수 있어, consumer 클라이언트의 성능이 향상됨
    
    > 페이지 캐시란, 운영체제가 디스크 I/O 성능을 향상시키기 위해 사용하는 메모리 영역으로, 자주 접근하는 디스크 데이터를 RAM에 캐싱한다.
    > 
- kafka와 다른 애플리케이션을 **같은 시스템에서 운영하지 않는 것을 권장**
    
    > Kafka는 성능을 위해 가능한 많은 메모리를 페이지 캐시로 활용하려고 한다. 다른 애플리케이션이 같은 시스템에서 실행되면 페이지 캐시를 두고 경쟁하게 되고, 이로 인해 Kafka의 페이지 캐시 히트율이 낮아지고 디스크 I/O가 증가하여 성능이 저하된다.
    > 

### 🌿 네트워크

- 네트워크 대역폭이 카프카 처리량 상한선을 결정함
- 네트워크가 포화 상태가 되면, 클러스터 복제 작업에 지연이 발생할 수 있음
- 예상 트래픽과 복제 작업을 감안해 충분한 네트워크 대역폭 확보 필요

### 🌿 CPU

- Kafka는 메시지 압축, 체크섬 확인, 오프셋 부여 등에 CPU를 사용
- 디스크나 메모리만큼 중요하지 않으므로 적정 수준의 할당 권장

## 🌳 Kafka Cluster 구성

단일 카프카 브로커는 로컬 개발 작업이나 개념 자격 증명 시스템에 적합하지만, 클러스터로 구성된 여러 브로커가 있으면 상당한 이점이 있다. 가장 큰 이점은 여러 서버에 부하를 분산시킬 수 있다는 것이다. 또한 복제를 사용하면 단일 시스템 장애로 인한 데이터 손실을 방지할 수 있다.

### 🌿 브로커 수

- 클러스터 크기 결정 요소
    - 디스크 용량
    - 브로커당 레프리카 용량
    - CPU 용량
    - 네트워크 대여곡
- 브로커 수 설정 예시
    - 10 TB 데이터를 저장하려면 (브로커당 저장 용량이 2TB일 경우) → 최소 5개의 브로커 필요
    - replication-factor를 2로 설정하려면 데이터가 각 브로커에 복제되어 10개의 브로커 필요

### 🌿 운영체제 튜닝하기

- 가상 메모리
    - 페이지 캐시 활용 : 디스크 I/O 성능을 위해 더티 페이지 관리 필요
    - 스왑 메모리 최소화 : 스왑 공간 대신 페이지 캐시에 메모리를 우선 할당
- 디스크
    - 파일 시스템 추천
        - Ext4 또는 XFS 사용
        - XFS는 추가 튜닝 없이도 카프카 워크로드에 적합
- 네트워킹
    - 송신 및 수신 버퍼 크기 설정
    - 브로커 동시 클라이언트 연결 수 설정

## 🌳 프로덕션 환경 고려사항

### 🌿 GC 옵션

- kafka는 G1GC(Garbage-First garbage collector)를 기본 GC로 사용하는 것을 권장
- G1GC는 다양한 작업 부하를 조절하고 일정한 GC 정지 시간을 유지
- **주요 옵션**
    - **`MaxGCPauseMillis`**
        - GC의 최대 정지 시간 설정
        - 짧게 설정하면 응답성이 향상되지만 처리량이 줄어들 수 있음
    - **`InitiatingHeapOccupancyPercent`**
        - GC가 시작되는 힙 메모리 사용 비율 설정
        - ex. `45`로 설정 시 힙의 45%를 사용하면 GC 시작
- **권장 사항**
    - 카프카는 힙 메모리를 효율적으로 사용하며 GC 대상 객체를 적게 생성
    - 위 설정값을 낮게 잡아도 성능에 큰 영향 없음

### 🌿 **데이터센터 레이아웃**

- 브로커 간 **랙(Rack) 위치**를 고려하여 장애 발생 시 데이터 가용성을 보장
    
    > 랙(Rack)은 데이터 센터에서 서버와 네트워크 장비를 물리적으로 보관하는 캐비닛 또는 프레임을 말한다. 현대 DC에서는 여러 서버를 효율적으로 관리하고 공간을 절약하기 위해 표준화된 랙을 사용한다.
    > 
- 동일 랙에 레플리카가 배치되지 않도록 구성(장애 도메인 분리)
- **설계 가이드**
    - 각 브로커를 서로 다른 랙 또는 데이터센터에 배치
    - 단일 장애점(전원, 네트워크 등)이 발생하지 않도록 구성

### 🌿 **주키퍼 공유하기**

- 카프카는 **ZooKeeper**를 사용해 브로커, 토픽, 파티션 메타데이터 관리
- **주키퍼 작동 방식**
    - 컨슈머 그룹 또는 클러스터 구성 변경 시에만 쓰기 작업 발생
    - 이러한 작업은 빈번하게 발생하지 않으므로 주키퍼에 발생하는 부하는 일반적으로 적다.
    - 하나의 카프카 클러스터만을 위해 주키퍼 앙상블(여러 주키퍼 서버로 구성된 그룹)을 따로 구축하는 것은 리소스 낭비일 수 있다 → 하나의 주키퍼 앙상블을 여러 카프카 클러스터가 공유하는 것이 더 효율적이다.

# Ch3. Kafka Producer **: 카프카에 메시지 쓰기**

## 🌳 프로듀서 개요

- **프로듀서의 역할**: 데이터를 카프카 브로커에 전송
- **주요 개념**
    - **`ProducerRecord`**: 카프카에 메시지를 작성하기 위한 객체
    - **`KafkaProducer`**: 프로듀서를 구성하고 브로커와 통신을 담당
    - **파티션**: 메시지가 저장되는 브로커의 논리적 단위
- **사용 예시**
    - 신용카드 트랜잭션 처리
    - 웹사이트 조회 로그 전송 등

## 🌳 **프로듀서 기본 구조**

- **`ProducerRecord`** 객체를 생성하며 시작
    - **토픽**과 **밸류**는 필수, 키와 파티션은 선택적으로 지정
- 메시지 전송 과정
    1. **직렬화**: 키와 값을 바이트 배열로 변환
    2. **파티셔닝**: 키가 있으면 해싱, 없으면 Round Robin 방식으로 파티션 결정
    3. **배치 추가**: 직렬화된 메시지를 레코드 배치에 추가
    4. **전송**: 백그라운드 스레드가 레코드 배치를 브로커로 전송

## 🌳 **필수 설정**

- **`bootstrap.servers`**
    - Kafka 클러스터에 대한 초기 연결을 설정하기 위해, producer가 사용할 브로커의 `host:port` 목록을 지정
    - 최소 2개 이상의 브로커를 설정하여 연결 안정성을 높임
- **`key.serializer`, `value.serializer`**
    - 메시지 키와 값을 직렬화하기 위한 설정
    - Producer가 key-value를 직렬화하는데 사용할 클래스의 이름
    - 일반적으로 `StringSerializer` 또는 `ByteArraySerializer`를 사용함

## 🌳 **메시지 전송 방식**

- **파이어 앤 포겟**
    - 메시지를 전송한 뒤 성공 여부를 확인하지 않음
    - 처리 속도가 빠르지만 메시지 손실 가능성이 있음
- **동기 전송**
    - `send()` 호출 후 `get()`으로 결과를 반환받음
    - 전송 결과를 기다리는 동안 스레드가 블록됨
    - `Future` 객체를 활용하여 비동기 결과를 동기적으로 처리
- **비동기 전송**
    - 콜백 함수를 사용해 메시지 전송 성공 또는 실패를 처리
    - 전송 결과에 따라 자동으로 콜백이 호출됨
    - 네트워크 지연을 최소화하며 효율적인 방식임

## 🌳 **주요 프로듀서 설정**

- **`acks` (Acknowledgment Level)**
    - 메시지 전송 신뢰성을 설정(producer가 쓰기 성공으로 간주하기 전에 몇 개의 복제본이 레코드를 받아야 하는지 제어
        - `acks=0`: 최소 전송 보장 At Most Once → 메시지가 성공적으로 전송되었다고 가정하기 전에 브로커로부터 응답을 기다리지 않음
            - 문제가 발생하여 브로커가 메시지를 받지 못한 경우 producer는 알지 못하고 메시지가 손실됨
        - `acks=1`: 중복 가능성을 허용하는 At Least Once → 리더 복제본이 메시지를 받는 순간 브로커로부터 성공 응답을 받는다.
            - 리더가 충돌하고 최신 메시지가 아직 새 리더에 복제되지 않은 경우에는 메시지가 여전히 손실될 수 있다.
        - `acks=all`: 모든 레플리카의 확인을 요구하는 Exactly Once → 모든 동기화 복제본이 메시지를 받으면 브로커로부터 성공 응답을 받는다.
- **`linger.ms`**
    - 배치 전송 전 대기 시간을 설정하여 효율성을 높임
- **`buffer.memory`**
    - 전송 대기 중인 메시지를 저장할 버퍼 크기
- **`compression.type`**
    - 메시지 압축 방식 설정 (`gzip`, `snappy`, `lz4`)
- **`enable.idempotence`**
    - 메시지를 정확히 한 번만 전송하도록 보장
    - 메시지 순서와 중복 방지를 위한 필수 설정

## 🌳 **시리얼라이저와 스키마**

- **기본 시리얼라이저**
    - `StringSerializer`, `ByteArraySerializer` 등 제공
- **범용 직렬화 라이브러리**
    - Apache Avro와 같은 라이브러리 권장
    - 데이터 타입과 스키마를 유지하면서 효율적인 직렬화 지원
    - 스키마 변경 시 기존 컨슈머와의 호환성 유지 가능

## 🌳 **파티션과 접착성 처리**

- **파티션 할당 방식**
    - 키 값이 존재하면 해싱하여 파티션 결정
    - ~~키 값이 없으면 Round Robin 방식으로 파티션 할당~~
        - 2.4 버전부터 기본적으로 Sticky Partitioning 방식을 사용함
- **접착성 처리**
    - 키 값이 없는 메시지들을 같은 배치로 묶어 전송하여 효율성을 높임

## 🌳 **레코드 헤더**

- 메시지의 키와 값 외에 추가적인 메타데이터를 포함할 수 있음
- 메시지를 파싱하지 않고도 헤더 정보를 활용해 메시지를 해석 가능

## 🌳 **인터셉터**

- **`onSend`**
    - 메시지가 브로커로 전송되기 전에 호출되며, 메시지 내용을 수정할 수 있음
- **`onAcknowledgement`**
    - 브로커 응답을 수신한 후 호출되며, 메시지를 읽기 전용으로 처리

## 🌳 **쿼터 및 쓰로틀링**

- **브로커 처리량 초과 시 발생하는 문제**
    - 클라이언트 버퍼 메모리가 부족하면 `send()` 호출이 블록됨
    - `delivery.timeout.ms`를 초과하면 메시지가 무효화됨
- **해결 방법**
    - 브로커와 프로듀서 간 병목현상을 모니터링하며 설정값 조정 필요

# 카프카 완벽 가이드 - 코어편

> 🔗 [https://www.inflearn.com/course/카프카-완벽가이드-코어?srsltid=AfmBOoqevFk6v7QNyxpS8gOYwIryzN68uyOuMqNrVJtxpeK4JDdlmiIX](https://www.inflearn.com/course/%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%99%84%EB%B2%BD%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%BD%94%EC%96%B4?srsltid=AfmBOoqevFk6v7QNyxpS8gOYwIryzN68uyOuMqNrVJtxpeK4JDdlmiIX)
> 

## 🍅 Kafka 소개, 그리고 왜 Kafka를 배워야 하는가?

- 다양한 대량 데이터의 수집/가공/저장을 위한 핵심 솔루션
- 기업 데이터 저장소의 통합 구축 기반 제공하며, 고성능 데이터 파이프라인 구축의 핵심 역할을 수행
- 분산 대용량 메시징 시스템의 가장 대표적인 솔루션

### Kafka의 주요 특징

- 뛰어난 핵심 기능
- 다양한 환경 지원
- 신뢰성과 편의성

### 왜 Kafka를 배워야 하는가

- 좋은 솔루션이고 현재도 많이 사용되고 있으며, 앞으로도 많이 사용될 것이다.
- 끝없는 비즈니스의 변화와 데이터 활용 욕구, 새로운 기술의 등장, 늘어만 가는 서브 시스템들은 변함없이 지속될 것이다.
- 불과 몇년 전 수준에서는 상상할 수 없엇던 데이터 처리를 기반으로 하는 비즈니스 요구사항과 이를 가능하게 하는 솔루션들이 등장하고 있다.
- Kafka는 급변하는 비즈니스 요구사항을 수용하는 기업 데이터 인프라들을 효율적으로 통합하고 유연하게 활용하는데 기여하기에, 그 중요도는 앞으로 더욱 높아질 것이다.

## 🍅 Kafka 실습 환경 구축

### Kafka 서버 구축

- Oracle Virtualbox 다운로드
- Ubuntu 20 설치 및 환경 구성
- JDK 및 Confluent Kafka 설치

### Kafka 클라이언트 구축

- JDK 설치
- IntelliJ 설치
- 기타 툴 설치

![image](https://github.com/user-attachments/assets/a1e35431-05fe-4742-8395-4f75c3d7113c)


## 🍅 Kafka 설치 Binary 개요

### Apache Kafka

> 링크드인에서 내부적으로 메시징 시스템 쓰려고 자기들이 만든 것
> 
- Apache Kafka는 Open Source 기반으로 비용 Free
- 많은 Community 사용자 층을 가지고 있음

### Confluent Kafka

> 컨플루언트 플랫폼이 포함된 가장 완전한 카프카의 배포판
> 
- Apache Kafka를 기반으로 하지만, 추가 기능이 포함된 상용 배포판
- Connector, Schema Registry 등 Kafka 기반의 다른 모듈과 일체화된 platform 제공
- Confluent Kafka Communtity 버전은 Open Source
기반으로 비용 Free
- Confluent Kafka Local Platform은 개발용도로 단 1개의 Broker기반에서만 동작. 비용 Free이며 Admin UI를 이용하여 Kafka의 다양한 Component 들을 관리하고 모니터링 수행 가능
    - Admin UI : 모니터링 용이, 오픈소스 UI보다 더 직관적
- Confluent Kafka Distributed Platform은 유료

### 환경 스택

- 서버 Kafka 환경은 Confluent Kafka 7.1.x(Apacha Kafa 3.1.x와 호환) 설치
- 서버 Java 환경은 JDK 11
- 클라이언트 Java 환경은 JDK 17

## 🍅 Oracle Virtualbox 설치

[Downloads     – Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)

![image](https://github.com/user-attachments/assets/1e54d994-19b5-4321-85fe-1922a125a76a)


changeme

- 고정 IP 설치하기
- mtputty 설치하기

## 🍅 Kafka 설치하기

```bash
wget https://packages.confluent.io/archive/7.1/confluent-community-7.1.2.tar.gz?_ga=2.257883202.813221415.1741676735-356653725.1741676735 --no-check-certificate
```

```bash
$ mv 'confluent-community-7.1.2.tar.gz?_ga=2.257883202.813221415.1741676735-356653725.1741676735' confluent-community.tar.gz
$ tar -xvf confluent-community.tar.gz
```

`.bashrc` 환경 변수 등록

```bash
export CONFLUENT_HOME=/home/vboxuser/confluent
export PATH=.:$PATH:$CONFLUENT_HOME/bin
```

zookeeper 기동하기

```bash
zookeeper-server-start $CONFLUENT_HOME/etc/kafka/zookeeper.properties
```

kafka 기동하기(반드시 zookeeper부터 기동)

```bash
kafka-server-start $CONFLUENT_HOME/etc/kafka/server.properties
```

<aside>
💡

shutdown 시에는 반드시 kafka부터 진행해야 한다. 

기동 순서 : zookeeper → kafka
종료 순서 : kafka → zookeeper

</aside>

[`kafka-server-stop.sh`](http://kafka-server-stop.sh) 의 내용

> 단순한 signal kill을 수행함
> 

```bash
#!/bin/bash
....
SIGNAL=${SIGNAL:-TERM}

OSNAME=$(uname -s)
if [[ "$OSNAME" == "OS/390" ]]; then
    if [ -z $JOBNAME ]; then
        JOBNAME="KAFKSTRT"
    fi
    PIDS=$(ps -A -o pid,jobname,comm | grep -i $JOBNAME | grep java | grep -v grep | awk '{print $1}')
elif [[ "$OSNAME" == "OS400" ]]; then
    PIDS=$(ps -Af | grep -i 'kafka\.Kafka' | grep java | grep -v grep | awk '{print $2}')
else
    PIDS=$(ps ax | grep ' kafka\.Kafka ' | grep java | grep -v grep | awk '{print $1}')
    PIDS_SUPPORT=$(ps ax | grep -i 'io\.confluent\.support\.metrics\.SupportedKafka' | grep java | grep -v grep | awk '{print $1}')
fi

if [ -z "$PIDS" ]; then
  # Normal Kafka is not running, but maybe we are running the support wrapper?
  if [ -z "${PIDS_SUPPORT}" ]; then
    echo "No kafka server to stop"
    exit 1
  else
    kill -s $SIGNAL $PIDS_SUPPORT
  fi
else
  kill -s $SIGNAL $PIDS
fi
```

## 🍅 Kafka 구성하기

topic 생성하기

```bash
$ kafka-topics --bootstrap-server localhost:9092 --create --topic welcome-topic
Created topic welcome-topic.
```

### Zookeeper, Kafka 기동 종료 스크립트 만들기

`zoo_start.sh`

```bash
$CONFLUENT_HOME/bin/zookeeper-server-start $CONFLUENT_HOME/etc/kafka/zookeeper.properties
```

`kafka_start.sh`

```bash
$CONFLUENT_HOME/bin/kafka-server-start $CONFLUENT_HOME/etc/kafka/server.properties
```

```bash
chmod +x *.sh

```

### server.properties

Kafka의 [`server.properties`](http://server.properties) 는 kafka broker를 설정하기 위한 구성 파일이다. 이 파일에는 브로커의 동작을 제어하는 다양한 설정이 포함되어 있다.

- 로그 설정 `log.dirs` : 로그 데이터가 저장된 디렉터리 경로를 지정한다.
- Kafka에서 로그란, 메시지 데이터를 의미한다. (시스템 로그가 아님)
- 모든 토픽의 파티션 데이터가 이 디렉토리에 저장된다.

```bash
############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/home/vboxuser/data/kafka-logs
```

```bash
INFO Attempting recovery for all logs in /home/vboxuser/data/kafka-logs since no clean 
shutdown file was found (kafka.log.LogManager)
```

다음과 같은 형식으로 저장된다.

```bash
$ ll
total 20
drwxrwxr-x 2 vboxuser vboxuser 4096  3월 11 17:40 ./
drwxrwxr-x 4 vboxuser vboxuser 4096  3월 11 17:30 ../
-rw-rw-r-- 1 vboxuser vboxuser    0  3월 11 17:40 cleaner-offset-checkpoint       
-rw-rw-r-- 1 vboxuser vboxuser    0  3월 11 17:40 .lock
-rw-rw-r-- 1 vboxuser vboxuser    4  3월 11 17:40 log-start-offset-checkpoint     
-rw-rw-r-- 1 vboxuser vboxuser   88  3월 11 17:40 meta.properties
-rw-rw-r-- 1 vboxuser vboxuser    4  3월 11 17:40 recovery-point-offset-checkpoint
-rw-rw-r-- 1 vboxuser vboxuser    0  3월 11 17:40 replication-offset-checkpoint
```

### zookeeper.properties

[`zookeeper.properties`](http://zookeeper.properties) 는 Zookeeper의 구성 파일이다. Zookeeper는 분산 시스템 조정 서비스로, Kafka 클러스터의 브로커 상태 관리, 토픽 구성, 액세스 제어 목록(ACL), 컨슈머 그룹 정보 등을 추적하는 데 사용된다.

> Kafka 2.8.0부터 ZooKeeper 의존성을 제거하기 위한 KRaft(Kafka Raft) 모드가 도입되었다고 한다. → Kafka 4.0부터는 ZooKeeper 모드가 더 이상 지원되지 않을 예정
> 
- 데이터 설정 `dataDir` : zookeeper가 데이터를 저장하는 디렉토리 경로

```bash
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
```

```bash
INFO Snapshotting: 0x0 to /home/vboxuser/data/zookeeper/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
```

다음과 같은 형식으로 저장된다.

```bash
$ ll 
total 12
drwxrwxr-x 3 vboxuser vboxuser 4096  3월 11 17:39 ./        
drwxrwxr-x 4 vboxuser vboxuser 4096  3월 11 17:30 ../       
drwxrwxr-x 2 vboxuser vboxuser 4096  3월 11 17:40 version-2/
```

## 🍅 Topic 생성하기

`welcome-topic` 을 생성한다.

```bash
$ kafka-topics --bootstrap-server localhost:9092 --create --topic welcome-to
pic
Created topic welcome-topic.
```

그러면 다음과 같이 `log.Dirs` 경로에 topic 이 생성된 것을 확인할 수 있다.

`welcome-topic-0` 에서 0은 파티션 번호로, kafka에서 모든 로그는 파티션 단위로 저장된다.

```bash
 $ ll
total 28
drwxrwxr-x 3 vboxuser vboxuser 4096  3월 11 17:43 ./
drwxrwxr-x 4 vboxuser vboxuser 4096  3월 11 17:30 ../
-rw-rw-r-- 1 vboxuser vboxuser    0  3월 11 17:40 cleaner-offset-checkpoint       
-rw-rw-r-- 1 vboxuser vboxuser    0  3월 11 17:40 .lock
-rw-rw-r-- 1 vboxuser vboxuser    4  3월 11 17:43 log-start-offset-checkpoint     
-rw-rw-r-- 1 vboxuser vboxuser   88  3월 11 17:40 meta.properties
-rw-rw-r-- 1 vboxuser vboxuser   22  3월 11 17:43 recovery-point-offset-checkpoint
-rw-rw-r-- 1 vboxuser vboxuser   22  3월 11 17:43 replication-offset-checkpoint   
drwxrwxr-x 2 vboxuser vboxuser 4096  3월 11 17:43 **welcome-topic-0/**
```

## 🍅 Topic 생성 및 정보 확인하기

`$CONFLUENT_HOME/bin/kafka-topics` command를 이용한다.

| 주요 인자 | 설명 |
| --- | --- |
| `--bootstrap-server` | Topic을 생성할 Kafka Broker 서버 주소: Port
`--bootstrap-server localhost:9092` |
| `--create` | `--topic` : 기술된 topic명으로 topic 신규 생성
`--partitions` : Topic의 파티션 개수
`--replication-factor` : replication 개수 |
| `--list` | 브로커에 있는 Topic들의 리스트 |
| `--describe` | `-- topic`: 기술된 topic명으로 상세 정보 표시 |

`test_topic_01` 생성하기

```bash
 kafka-topics --bootstrap-server localhost:9092 --create --topic test_topic01
```

topic 목록 확인하기

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

partition 3개를 가지는 topic 만들어보기

> 개수를 지정하지 않았을 때 default partition 개수는 [`server.properties`](http://server.properties) 의 `num.partitions` 에 설정된다.
> 

```bash
 kafka-topics --bootstrap-server localhost:9092 --create --topic test_topic_02 --partitions 3
```

개별 topic의 상세 내용 확인하기

```bash
$ kafka-topics --bootstrap-server localhost:9092 --describe --topic test_topic_02
Topic: test_topic_02    TopicId: 7WMF61LNR26xxA9KogSE_A PartitionCount: 3       ReplicationFactor: 1    Configs: 
segment.bytes=1073741824
        Topic: test_topic_02    Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: test_topic_02    Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: test_topic_02    Partition: 2    Leader: 0       Replicas: 0     Isr: 0
```

복제본 수(replication-factor) 지정해서 topic 생성하기

여기서 주의! 현재 다음과 같이 에러가 발생하고 있는데, Broker가 1개인 상황에서 2개의 replication-factor를 구성하려 했기 때문이다. 

```bash
$ kafka-topics --bootstrap-server localhost:9092 --create --topic test_topic_03 --partitions 3 -
-replication-factor 2
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Error while executing topic command : Replication factor: 2 larger than available brokers: 1.
[2025-03-12 11:02:52,581] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 2 larger than available brokers: 1.
 (kafka.admin.TopicCommand$)
```

topic 삭제하기

> 즉각적으로 사라지지는 않고, 조금 기다려야 한다.
> 

```bash
kafka-topics --bootstrap-server localhost:9092 --delete --topic test_topic_02
```

## 🍅 kafka-console-producer와 kafka-console-consumer로 Producer와 Consumer 실습

다음과 같이 test 토픽에 프로듀싱해보자.

```bash
$ kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
>aaa
>bbb
>111
>ccc
>ddd
> 안녕하세요
>eee
```

그리고 다음과 같이 컨슈밍해올 수 있다.

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic
```

그러나 명령어를 실행해도, 이전에 프로듀싱한 메시지는 컨슈밍되지 않을 것이다.

여기서 주의해야 할, 알아야 할 개념이 있다.

### Consumer의 auto.offset.reset

- consumer가 topic에 **처음 접속하여** Message를 가져올 때 가장 오래된 처음 offset부터(earlist) 가져올 것인지, 가장 최근인 마지막 offset 부터 가져올 것인지를 설정하는 파라미터
- `auto.offset.reset` = earlist : 처음 offset 부터 읽음
- `auto.offset.reset` = latest : 마지막 offset 부터 읽음

`kafka-console-consumer` 명령어를 사용할 때 `--from-beginning` 을 사용해야만 `auto.offset.reset` 이 earlist로 지정됨

![image](https://github.com/user-attachments/assets/80f5200d-ad48-4efd-8646-64f796492e04)


그래서 다음과 같이 옵션을 사용해서 다시 명령어를 실행하면, 이전의 메시지들을 받아오는 것을 볼 수 있다.

```bash
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic 
--from-beginning
aaa
bbb        
111        
ccc        
ddd        
�안녕하세요
eee        
```

`--from-beginning` 이 없다면, 명령어를 실행한 순간(latest)부터의 메시지를 받아올 준비를 하고 있는 것이다. 지금까지 key 값이 없는 메시지를 보내는 실습을 해보았다.

## 🍅 Producer의 객체 직렬화(Serializer) 전송의 이해

메시지는 항상 **Byte Array 형식으로 전송**된다.

이렇게 했을 때, 훨씬 네트워크 대역폭을 잘 사용할 수 있고, 압축의 효율성도 좋다.

- Serializaer
- Deserializer

![image](https://github.com/user-attachments/assets/38b73d02-1925-456f-8129-96a52a3e9065)


### 자바 객체(Object)의 Serialization

객체(Object)를 객체의 유형, 데이터의 포맷, 적용 시스템에 상관없이 이동/저장/복원을 자유롭게 하기 위해서 바이트 배열(바이트 스트림) 형태로 저장하는 것이다. 객체는 Serialization과 Deserialization을 통해서 System to System 또는 서로 다른 저장 영역에 이동/저장/복원을 자유롭게 수행한다.

![image](https://github.com/user-attachments/assets/b820c602-0190-4bca-9d66-60e511846e44)


### Serialization과 Deserialization을 통한 노드간의 데이터 전송

Serialization을 통해서 객체가 바이트 스트림으로 변환되어 네트워크를 통해 손쉽게 데이터를 전송할 수 있으며, Deserilization을 통해서 바이트 스트림은 다시 원본 객체로 변환되어 자유로운 객체 데이터 이동을 수행한다.

> TCP/IP와 같은 네트워크 프로토콜은 바이너리 데이터(바이트)를 주고받는 것을 기반으로 설계되었기 때문이다.
> 

![image](https://github.com/user-attachments/assets/6a8bd4e0-865a-421b-b675-0f5a3a0fd72c)


Producer와 Consumer에서 직렬화/역직렬화 적용

![image](https://github.com/user-attachments/assets/44d42969-d38e-42ef-b772-6c7dbec49dbc)


### Kafka에서 기본 제공하는 Serializer

- StringSerializer
- ShortSerializer
- IntegerSerializer
- LongSerializer
- DoubleSerializer
- BytesSerializer

![image](https://github.com/user-attachments/assets/b10de257-3e3b-4057-aeba-760241f37006)


## 🍅 Key 값을 가지지 않는 메시지 전송

![image](https://github.com/user-attachments/assets/4a3a2700-0b3f-494b-b0c5-7743a406352a)


- 메시지는 Producer를 통해 전송 시 Partitioner를 통해 토픽의 어떤 파티션으로 전송되어야 할 지 미리 결정이 됨.
- key 값을 가지지 않는 경우 라운드 로빈(Round Robin), 스티키 파티션(Sticky Partition) 등의 파티션 전략 등이 선택되어 파티션 별로 메시지가 전송될 수 있음
- Topic이 여러 개의 파티션을 가질 때 메시지의 **전송 순서가 보장되지 않은** 채로 Consumer에서 읽혀질 수 있음

## 🍅 Key 값을 가지는 메시지 전송

![image](https://github.com/user-attachments/assets/d01e90db-857c-4c11-aa7e-dbc82dc37afc)

- 메시지 key는 업무 로직이나 메시지 Produce/Consume 시 분산 성능 영향을 고려하여 생성
- 특정 key 값을 가지는 메시지는 **특정 파티션으로 고정되어 전송**된다.
    - 해싱 알고리즘을 이용하여, 특정 key → 특정 partition으로 전송되도록 한다.
- 특정 key 값을 가지는 메시지는 단일 파티션 내에서 전송 순서가 보장되어 Consumer에서 읽혀짐

> **Kafka는 하나의 파티션 내에서만 메시지의 순서를 보장한다.**
> 

이제 콘솔 프로듀서를 통해 토픽에, key 값을 가지는 메시지를 전송해보자.

```bash
kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic \
--property key.separator=: --property parse.key=true
```

그리고 다음과 같이 메시지를 전송하자.

```bash
>01:aaa
>02:bbb
>01:ccc
>02:ddd
```

그리고 콘솔 컨슈머를 통해 메시지를 확인해보자.

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic \
--property print.key=true --property print.value=true --from-beginning
```

```bash
01      aaa
02      bbb
01      ccc
02      ddd
```

## 🍅 여러 개의 파티션을 가지는 메시지 전송 실습

우선 3개의 파티션을 가지는 토픽을 만들고 메시지를 보내보자.

```bash
$ kafka-console-producer --bootstrap-server localhost:9092 --topic multipart-topic
>aaa
>bbb
>ccc
>ddd
>eee
>fff
>ggg
>bbb
>kkk
>rrr 
```

컨슈밍할 때, `--property print.partition=true` 를 설정하면 파티션과 함께 확인할 수 있다

```bash
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic multipart-topic --from-beginning --property print.partition=true
Partition:2     ddd
Partition:2     bbb
Partition:0     bbb
Partition:0     ccc
Partition:0     fff
Partition:0     kkk
Partition:1     aaa
Partition:1     eee
Partition:1     ggg
Partition:1     rrr
```

파티션 단위로 순서가 보장되는 것을 확인할 수 있다.

`console-consumer`는 배치 단위로 읽어올 때, 파티션을 묶음으로 불러온다.

## 🍅 Kafka Producer의 send() 메소드 호출 프로세스

![image](https://github.com/user-attachments/assets/787a18da-3b3d-4012-9a23-c8857ddecd63)

## 🍅 key 값을 가지지 않는 메시지 전송 시 파티션 분배 전략 - Sticky 파티셔닝

> 파티션 분배 : Producer가 메시지를 어떤 파티션에 기록할 지 결정하는 과정
> 
- Apache Kafka 2.4부터 스티키 파티셔닝 (**Sticky Partitioner**) 전략이 채택됨.
- 라운드 로빈의 성능을 개선하고자, 특정 파티션으로 전송되는 하나의 배치에 메시지를 빠르게 먼저 채워서 보내는 방식
- 한 배치가 채워질 때까지 같은 파티션에 메시지를 보낸 후 다음 파티션으로 이동한다. 이는 더 적은 요청으로 더 많은 메시지를 보낼 수 있게 해 지연 시간을 줄이고 Broker의 CPU 사용률을 감소시킨다.
- 배치를 채우지 못하고 전송을 하거나 배치는 채우는데 시간이 너무 많이 걸리는 문제를 개선

![image](https://github.com/user-attachments/assets/72dd812f-bcc0-4460-bef2-91ffe260d866)

- [`linger.ms`](http://linger.ms) : 현재 배치가 전송되기 전에 producer가 추가 메시지를 기다리는 시간
    - 시간 기반 배치 완료 기준 “얼마나 오래 기다리면 전송할 것인가”
- `batch.size` : 동일 파티션으로 전송되는 여러 레코드를 묶을 때 사용되는 각 배치의 최대 크기
    - 크기 기반 배치 완료 기준 “얼마나 많은 데이터가 모이면 전송할 것인가”

> 두 가지 중 하나라도 조건이 충족되면 전송된다.
>
