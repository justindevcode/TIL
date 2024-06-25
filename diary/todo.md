# todo

---
## 20240625
### 레디스 실사용

#### 레디스 설치
리눅스 우분투 22.04 환경  
```
#!/bin/bash

sudo apt update

sudo apt install redis-server
```

#### 레디스 설정

```
서버시작
sudo systemctl start redis-server

redis 서버 종료
sudo systemctl stop redis-server

redis 서버 상태 확인
sudo systemctl status redis-server

redis 서버 재시작
sudo systemctl restart redis-server

redis.conf 파일 위치
sudo vi /etc/redis/redis.conf

conf 파일 접근
sudo vi /etc/redis/redis.conf

비밀번호 설정
requirepass 비밀번호

redis-cli 비밀번호 입력
auth 비밀번호
//auth 사용자명 비밀번호

서버 재시작
sudo systemctl restart redis-server

Redis Bind 설정
conf 파일 접근
sudo vi /etc/redis/redis.conf

bind 항목 변경
bind 0.0.0.0 ::1

서버 재시작
sudo systemctl restart redis-server

스냅샷 설정
redis conf 파일 접속
sudo vi /etc/redis/redis.conf

save 파라미터 설정 : 스냅샷 주기
save 초단위시간 KEY변경수

save 900 1
save 300 10
save 60 10000

스냅샷 실패시 레디스 쓰기 요청 거부 설정
stop-wrotes-on-bgsave-error yes

스냅샷 압축 여부
rdbcompression yes

rdb 파일 마지막에 CRC64 checksum 기록 여부 (순환 중복 검사)
rdbchecksum yes

스냅샷 파일명 설정
dbfilename dump.rdb

스냅샷 파일 사용 후 제거 여부
rdb-del-sync-files no

스냅샷 저장 위치
dir /var/lib/redis
```

#### Redis CLI
```
Redis CLI 접근 : 로컬
redis-cli

Redis CLI 접근 : 원격
redis-cli -h IP주소 -p 포트 -a 비밀번호

Redis CLI 접근시 database 설정
redis-cli -n 데이터베이스번호

n 옵션을 통해 데이터베이스번호 명시가능, 이때 db번호는 0번부터 시작
```

#### Redis 데이터 타입

```
Redis 데이터 타입
레디스의 모든 데이터는 “key” - “value” 형태로 저장된다. 문자열 key에 대응되는 여러 타입의 value가 존재하며 그 종류는 아래와 같다.

String
단순 문자열 데이터로 최대 길이 512MB

Bitmaps
비트 단위의 이진수 데이터

Lists
배열형태의 데이터로 key와 value의 관계는 1 대 N이다. value의 최대 개수는 2^32 - 1 개며 주로 큐와 스택으로 활용된다.

Sets
집합 (중복되는 데이터가 없는 Lists)

Sorted Sets
집합에서 각각의 원소에 스코어라는 고유 숫자가 부여된다.

Hashes
json과 같은 데이터 형태로 key : value 내부에 key : value 가 존재한다.

HyperLogLogs
긴 길이의 Bitmaps 데이터

Streams
로그와 같은 스트림 데이터

//

데이터 CRUD
String 데이터

set : 데이터 쓰기
set KEY VALUE
//EX. set 1 kim

set과 함께 expire 시간 설정
set KEY VALUE EX 초단위시간
set KEY VALUE PX 밀리초단위시간

get : 데이터 읽기
get KEY
//EX. get 1

scan : 단위 키 조회
scan 단위숫자
//EX. scan 0

keys : 특정 키 조회
keys 패턴
//EX. keys *

exists : 키 존재 여부 확인
exists KEY
//EX. exists 1

del : 키 삭제
del KEY
//EX. del 1

flushall : 모든 키 삭제
flushall

//

### Lists 데이터


lpush : 좌측에서 데이터 추가
lpush KEY VALUE
//EX. lpush testlist a

rpush : 우측에서 데이터 추가
rpush KEY VALUE

lpop : 좌측에서 데이터 제거
lpop KEY
//EX. lpop testlist

rpop : 우측에서 데이터 제거
rpop KEY

llen : 저장된 데이터 개수 출력
llen KEY

lrange : 지정된 단위 VALUE 출력
lrange KEY 시작인덱스 종료인덱스
```

#### Replication, 클러스터

### Replication : 복제

Replication은 Redis에서 지원하는 복제 로직으로 동일한 데이터에 대해서 마스터 - 슬레이브 구성을 갖추어 마스터 서버에 장애가 발생하는 경우 슬레이브 서버를 통해 마스터를 대체할 수 있도록 구성할 수 있다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/1c76df36-ccc5-42fb-931c-05a506ffb13d)  

Replication 설정  

복제의 경우 슬레이브 노드의 conf 파일에서 설정을 진행한다  

```
redis.conf 파일
sudo vi /etc/redis/redis.conf

복제 설정
replicaof 마스터IP 마스터포트
//EX. replicaof 127.0.0.1 6379

설정 후 재시작
sudo systemctl restart redis-server
```

마스터 서버 장애 대비  
마스터 서버에 장애가 발생하였을 경우 슬레이브 서버를 마스터 서버로 승격시켜야 한다.  
이때 수동으로 설정을 진행하는 것이 거의 불가능하기 때문에 Redis는 Sentinel이라는 도구를 지원한다.  

* 클러스터 사용 이유

물리적인 휘발성 메모리의 저장용량 한계로 인해 한계 이상의 데이터를 입력해야하는 경우 여러개의 노드를 클러스터링시켜 저장공간을 확보시킬 수 있다.  
단일 노드 구성대비 서버 장애에 대한 부분 동작이 가능하다.  

클러스터 : 마스터 - 슬레이브  
클러스터 설정시 마스터 - 슬레이브 설정 또한 진행되면 장애 발생시 자동으로 슬레이브 → 마스터 승격 또한 가능하다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/a43309c3-0ca6-47b0-8ef9-726e21ecaf73)  

클러스터 조건  

1. 레디스 클러스터는 최소 3개 이상의 노드로 구성된다.  
2. 클러스터 기능을 사용하면 database 기능을 활용하지 못하고 database는 0번 인덱스만 사용가능하다.  
3. 클러스터간 통신을 위해 기본포트 + 10000번 포트를 오픈 (16379)

클러스터 설정  

```
redis.conf 파일 접근
sudo vi /etc/redis/redis.conf

protected-mode 종료
protected-mode no

클러스터 설정
cluster-enavled yes

클러스터 구성 파일 이름 설정 (자동 생성)
cluster-config-gile nodes-6379.conf

노드간 통신 장애시 타임아웃 설정 (10000 = 10s)
cluster-node-timeout 10000

appendonly 실행
appendonly yes

재시작
sudo systemctl restart redis-server

redis-cli로 클러스터 시작 명령
redis-cli --cluster create 아이피:포트 아이피:포트 아이피:포트

추가로 클러스터 생성시 슬레이브 노드 옵션을 추가하려면
—cluster-replicas 개수
```

#### AWS ElastiCache

AWS ElastiCace는 AWS에서 제공하는 SaaS형 레디스로 성능, 리전, 클러스터 설정만 GUI로 진행하면 추후 발생하는 모든 유지보수(노드 추가, 장애 대응, 업데이트)는 AWS에서 관리하게 된다.  
ElastiCache의 경우 Redis와 Memcached 엔진을 지원한다.  

```
ElastiCache : Redis 생성
AWS 콘솔 홈 접근 후 ElastiCache 접속
Redis 클러스터 클릭
Redis 클러스터 생성 클릭
새 클러스터 구성 및 생성 선택 후

클러스터 모드를 설정할 수 있다. 
클러스터 모드 활성화시 다중 노드를 통해 스케일 아웃 및 Master-Slave 구조를 형성할 수 있으며 비활성화시 단일 마스터 노드로 구성된다.

클러스터 이름 설정
클러스터 생성 위치 설정

기본적으로 AWS 클라우드이며 온프레미스의 경우 실물 서버를 소유하고 있는 경우 연결할 수 있다.
다중 AZ는 클러스터 모드 활성화시 여러 AZ에 걸쳐 ElastiCache 노드를 분산하여 고가용성을 유지할 수 있다.

Redis 버전, 포트, 파라미터값, 노드 성능, 샤드 개수, Slave 노드 수 설정
서브넷 설정

클러스터 데이터 입력시 노드별 균등 저장 방법
암호화 및 보안그룹 설정

보안 그룹 설정을 통해 클러스터에 접근할 포트 오픈을 진행해야 한다.
(추후 보안 그룹에 대한 설정은 : 콘솔홈 - EC2 - 보안그룹)

백업 관련 설정

업데이트 및 로그 관련 설정
이후 확인사항을 거쳐 생성을 할 수 있다.

//


접속 원리

ElastiCache는 외부 접속이 불가능하며 AWS 내부 접속만 가능하다. 따라서 AWS EC2를 사용하여 접속을 진행한다.

AWS EC2 생성
EC2에 레디스 서버 설치
ElastiCache가 존재하는 보안그룹 6379 포트 열기
EC2에서 redis-cli로 원격 접속 진행

클러스터 IP는 ElastiCache 대시보드에서 확인할 수 있다.

기본 엔드포인트 참조 (IP:포트)

//

접속

보안그룹 설정
ElastiCache가 존재하는 보안 그룹을 선택하여 6379 포트 오픈

AWS EC2 접속
레디스 서버가 설치된 EC2 접속 후 아래 명령어로 접근
redis-cli -h 아이피주소 -p 6379

IP의 경우 기본엔드포인트 참조
```


---
## 20240624
### 메트릭 등록 카운터 기본

비즈니스로직적인 부분을 직접 메트릭등록을해서 그라파나에서 확인할 수 있도록 만들어보자  

주문을 하면 개수가올라가고, 최소를 하면 개수가 줄어들되
메트릭에는 주문을 하면 1씩 증가 취소를해도 취소요청개수가 1씩증가 이런식으로 만들어보자

* controller
```java
@Slf4j
@RestController
public class OrderController {
 private final OrderService orderService;
 public OrderController(OrderService orderService) {
 this.orderService = orderService;
 }
 @GetMapping("/order")
 public String order() {
 log.info("order");
 orderService.order();
 return "order";
 }
 @GetMapping("/cancel")
 public String cancel() {
 log.info("cancel");
 orderService.cancel();
 return "cancel";
 }
 @GetMapping("/stock")
 public int stock() {
 log.info("stock");
 return orderService.getStock().get();
 }
}
```

* service
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.atomic.AtomicInteger;
@Slf4j
public class OrderServiceV1 implements OrderService {
 private final MeterRegistry registry;
 private AtomicInteger stock = new AtomicInteger(100);
 public OrderServiceV1(MeterRegistry registry) {
 this.registry = registry;
 }
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "order")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "cancel")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
기본적으로 `private final MeterRegistry registry;`로 등록이 가능하다. static 빌더로 이름, tag들, `.register(registry).increment();`사용되면 증가
이런식으로 등록하면된다.  

* config
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class OrderConfigV1 {
 @Bean
 OrderService orderService(MeterRegistry registry) {
 return new OrderServiceV1(registry);
 }
}
```
config 등록

각 엔드포인드를 요청후에 

* http://localhost:8080/actuator/metrics/my.order
```html
{
 "name": "my.order",
 "description": "order",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 2
 }
 ],
 "availableTags": [
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v1.OrderServiceV1"
 ]
 }
 ]
}
```
들어가보면 기본적으로 엑츄에이터에도 등록되어있고

* http://localhost:8080/actuator/prometheus
```html
# HELP my_order_total order
# TYPE my_order_total counter
my_order_total{class="hello.order.v1.OrderServiceV1",method="order",} 1.0
my_order_total{class="hello.order.v1.OrderServiceV1",method="cancel",} 1.0
```
프로메테우스에도 자기 형식에 맞게 조금 바뀌어서 등록되어진다.  

이걸기반으로 그라파나에서도 요청해서 사용할 수 있다.  

* increase(my_order_total{method="order"}[1m])
* Legend : {{method}}
* increase(my_order_total{method="cancel"}[1m])
* Legend : {{method}}


---
## 20240621
### sql 주문량이 많은 아이스크림 조회하기 4단계

* FIRST_HALF 
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(외래키)|INT(N)|FALSE|
|FLAVOR(기본키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

* JULY
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(기본키)|INT(N)|FALSE|
|FLAVOR(외래키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

7월 아이스크림 총 주문량과 상반기의 아이스크림 총 주문량을 더한 값이 큰 순서대로 상위 3개의 맛을 조회하는 SQL 문을 작성해주세요.  

즉 JULY에서는 같은 FLAVOR에 다른 SHIPMENT_ID로 TOTAL_ORDER여러개 있을 수 있어서 1차적으로 합치고
그다음 TOTAL_ORDER에서 TOTAL_ORDER한번더 합쳐줘야함  

```sql
SELECT fh.FLAVOR
FROM FIRST_HALF AS fh
LEFT JOIN (
    SELECT flavor, SUM(total_order) AS total_order
    FROM JULY
    GROUP BY flavor
) AS jy ON fh.flavor = jy.flavor
ORDER BY fh.total_order + jy.total_order DESC
LIMIT 3;
```
서브쿼리 이용 문제였음 위와같은 상황일경우 서브쿼리로 1차적으로 한번 합치고 그다음 다시 더하는 연산 해야하는듯  
