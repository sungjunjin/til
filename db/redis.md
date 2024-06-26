# Redis (Remote Dictionary Server)

Redis는 표준 C로 작성된 오픈소스 기반 인메모리 데이터 저장소이다. Key-Value 형태로 데이터를 저장한다. 높은 성능과 다양한 데이터 타입을 지원하여 다양한 곳에 많이 사용하는 데이터베이스이다. 

## 특징
- In-Memory : 백업 / 스냅샷을 제외한 모든 데이터를 RAM에 저장한다
- Single Threaded : 단일 스레드에서 모든 task를 처리한다
- Cluster Mode 지원 : 다중 노드에 데이터를 분산하여 높은 가용성을 제공한다
- Persistence : RDB(Redis DataBase) + AOF(Append Only File)을 통해 영속성 옵션을 제공한다
- Pub/Sub : Pub/Sub 패턴을 지원하여 채팅, 알림과 같은 어플리케이션을 쉽게 개발할 수 있다

## 사용 사례

레디스의 주요 사용 사례는 다음과 같다.

Caching : 임시 비밀번호, 로그인 세션에 사용된다
Rate Limiter : 특정 API에 대한 요청 횟수 제한에 사용된다. Fixed-Window / Sliding-Window Rate Limtier (비율 계산기)
Message Broker : 레디스의 리스트나 스트림즈를 활용하여 메세지 큐를 구현할 수 있다
실시간 분석 / 계산 : 순위표, 반경 탐색, 방문자 수 계산에 사용된다
실시간 채팅 : Pub / Sub 패턴을 활용하여 구현이 가능하다

## Caching
레디스의 대표적인 사용 목적은 캐싱(Caching)이다.

캐싱은 데이터를 빠르게 읽기 위해 속도가 빠른 메모리 저장소를 활용하여 임시적으로 데이터를 저장했다가 필요할때 데이터를 메모리 저장소에서 바로 꺼내오는 기법이다. 

대표적으로 RAM과의 속도 차이를 줄이기 위해 L1, L2, L3 캐시를 활용하는 CPU 캐시가 있다. 웹 브라우저, DNS, CDN, 데이터베이스 캐싱도 있다.

### Cache Hit & Cache Miss

Cache Hit : 찾고자 하는 데이터가 캐시에 존재하는 경우를 의미한다.

Cache Miss : 찾고자 하는 데이터가 캐시에 존재하지 않는 경우를 의미한다. 이런 경우 RDB와 같은 디스크 데이터 저장소에 쿼리를 하여 해당 데이터가 있는지 확인하고, 있으면 다음 요청에 활용할 수 있게 캐시 메모리에 저장 후 데이터를 반환한다. 이를 Cache-Aside 패턴이라고 한다.

## Persistence
레디스는 기본적으로 인메모리 기반 저장소로 사용되지만 영속성(Persistence) 옵션을 제공한다. 옵션을 활성화하면 SSD와 같은 2차 메모리에 데이터를 저장한다.

레디스의 영속성 옵션은 다음 두 가지가 있다.

### RDB(Redis DataBase)
특정 시간에 저장된 데이터의 스냅샷을 생성하는 기술이다. 아래와 같은 상황에 사용된다.
- 장애 발생 시 스냅샷으로 캐시를 복구
- 동일한 데이터를 복제

하지만 스냅샷 생성 시 순간 성능 저하와 스냅샷 생성 이전의 데이터 유실에 신경을 써야 하는 점이 있다

### AOF(Append Only File)
모든 write 작업을 모두 log로 저장한다. 

데이터 유실의 위험이 상대적으로 적지만 재난 복구 시 write 작업을 다시 실행하기 때문에 RDB보다 느리다는 단점이 있다.

## 데이터 타입

### Strings
Strings 타입은 문자열, 직렬화된 객체 (JSON String) 등을 저장하는데 사용된다. 

Strings 타입을 저장할때 사용하는 명령어는 SET 이다.
```
127.0.0.1:6379> set lecture inflearn-redis
OK

127.0.0.1:6379> get lecture
"inflearn-redis"
```

여러가지 Strings 타입의 데이터를 한번에 저장할때는 MSET(Multi Set) 명령어를 사용한다.
```
127.0.0.1:6379> mset price 100 language ko
OK

127.0.0.1:6379> get price
"100"

127.0.0.1:6379> get language
"ko"
```

JSON String을 저장하는 예시는 다음과 같다. 어플리케이션 개발 시 이 값을 다시 역직렬화하여 JSON 객체로 사용한다.
```
127.0.0.1:6379> set inflearn-redis '{"price" : 1000, "language" : "ko"}'
OK

127.0.0.1:6379> get inflearn-redis
"{\"price\" : 1000, \"language\" : \"ko\"}"
```

### Lists
String을 이중 연결 리스트 (Doubly Linked List)로 저장한다. 큐나 스택을 쉽게 구현할 수 있다.

LPUSH로 데이터를 저장하고 RPOP으로 꺼내오면 아래와 같이 큐로 활용할 수 있다
```
127.0.0.1:6379> lpush queue job1 job2 job3
(integer) 3

127.0.0.1:6379> rpop queue
"job1"
```

LPUSH로 데이터를 저장하고 LPOP으로 꺼내오면 아래와 같이 스택으로 활용할 수 있다.
```
127.0.0.1:6379> lpush queue job1 job2 job3
(integer) 3

127.0.0.1:6379> lpop queue
"job3"
```

### Sets
Sets는 중복없는 문자열을 저장하는 정렬되지 않는 자료구조이다. 두 집합 간의 관계를 구하는 자료구조로 활용된다.

SADD 명령어를 활용해 데이터를 저장한다. SMEMBERS 명령어를 사용하여 Set의 모든 요소를 출력한다.

```
127.0.0.1:6379> sadd user:1:fruits apple banana orange orange
(integer) 3

127.0.0.1:6379> smembers user:1:fruits
1) "orange"
2) "banana"
3) "apple"
```

SCARD 명령어를 사용하면 해당 set의 카디널리티를 가져온다.
```
127.0.0.1:6379> scard user:1:fruits
(integer) 3
```

SISMEMBER 명령어를 사용하면 Set에 포함되어 있는지에 대한 여부를 알 수 있다.
```
127.0.0.1:6379> sismember user:1:fruits banana
(integer) 1
```

두 Set 간의 집합 관계를 구할 수 있는 명령어는 다음과 같다.
- SINTER : 두 Set 간의 교집합을 구한다
- SDIFF: 두 Set 간의 대칭 차집합을 구한다
- SUNION : 두 Set 간의 합집합을 구한다

```
127.0.0.1:6379> sadd user:1:fruits apple banana orange
(integer) 3

127.0.0.1:6379> sadd user:2:fruits apple lemon
(integer) 2

127.0.0.1:6379> sinter user:1:fruits user:2:fruits
1) "apple"

127.0.0.1:6379> sdiff user:1:fruits user:2:fruits
1) "orange"
2) "banana"

127.0.0.1:6379> sunion user:1:fruits user:2:fruits
1) "orange"
2) "lemon"
3) "apple"
4) "banana"
```

### Hashes
Field-Value 구조를 갖는 데이터 타입으로써 Dictionary, Map의 자료구조와 유사하다. 다양한 속성을 갖는 객체의 데이터를 저장할 때 유용하다.

### Sorted Sets

### Streams

### Geospatial

### Bitmaps

### HyperLogLog

### BloomFilter




