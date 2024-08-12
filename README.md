# Kafka Streams

## 카프카와 완벽 호환됨

- 보통 카프카에 저장된 데이터를 스파크, 로그스테시와 같은 곳으로 보냄
- 이러한 외부 오픈소스 툴은 빠르게 발전하는 오픈소스 카프카 버전을 못 따라옴
- 반면에 스트림즈는 매번 카프카가 릴리즈 될 때마다 카프카 클러스터와 완벽하게 호환됨
- 또한 성능개선도 빠르게 이루어지고 있으며 유실이나 중복 처리되지 않고 딱 한 번만 처리될 수 있는 강력한 기능 보유
- 카프카와 연동하는 이벤트 프로세싱 도구 중에 거의 유일하다고 볼 수 있음


## 스케쥴링 도구가 필요없음

- 가장 널리 사용되는 카프카와 연동되는 스트림 프로세싱 툴은 스파크 스트림이 있음
- 스파크 스트림을 사용하면 카프카와 연동하여 마이크로 배치 처리를 하는 이벤트 데이터 애플리케이션을 만들 수 있음
- 그러나 스파크를 운영하기 위에선 yarn이나 mesos와 같은 클러스터 관리자(리소스 매니저)가 필요함
- 반면에 스트림즈를 사용하면 스케쥴링 도구가 전혀 필요없음
- 원하는 만큼만 배포하면 됨 (적은 양의 데이터를 처리해야된다면 2개 정도의 스트림즈 애플리케이션만 띄우면 됨)


## 스트림즈DSL과 프로세서API를 제공

### 스트림즈를 구현하는 방법

**1. 스트림즈DSL**

- 대부분의 기능들이 다 여기에 있음
- 이벤트 기반 데이터처리를 할 때 필요한 map, join, window와 같은 메서드들을 제공함
- KStream, KTable, GlobalKTable은 카프카를 스트림 데이터 처리뿐만 아니라 대규모 key-value 저장소로도 사용할 수 있게 함

**2. 프로세서API**

- 스트림즈DSL에서 제공하지 않는 기능을 보완함
- 자체적으로 로컬 상태저장소를 사용함


## 상태기반 처리에 특화됨

### 실시간으로 들어오는 데이터를 처리하는 방식

**1. 비 상태기반 처리(stateless)**
- 필터링이나 데이터를 변환하는 처리
- 데이터가 들어오는 족족 바로 처리하고 프로듀스하면되기 때문에 유실될 일이 없음

**2. 상태기반 처리(stateful)**

- 상태기반 처리를 직접 구현하려면 정말 어려움
- window, join, aggregation과 같은 처리는 이전에 받았던 데이터를 프로세스가 메모리에 저장하고 있으면서 다음 데이터를 참조해서 처리해야됨

- 스트림즈는 이런 어려움을 극복하게 도와줄 수 있음
- 로컬에 rocksdb를 사용해서 상태를 저장하고, 이 상태에 대한 변환 정보는 카프카의 변경로그(changelog) 토픽에 저장함
- 프로세서에 장애가 발생하더라도 상태가 모두 안전하게 저장되기 때문에 자연스럽게 장애복구 가능
- 컨슈머로 폴링하거나 프로듀서를 어렵게 구현할 필요가 없음

```java
// payment 토픽에 메시지 키가 "unknown"인 데이터를
// 필터링해서 unknown-payment 토픽으로 보냄

KStream<String, String> paymentStream = builder.stream("payment");
KStream<String, String> filteredStream = paymentStream.filter((key, value) -> key.equals("unknown"));
filteredStream.to("unknown-payment");

```