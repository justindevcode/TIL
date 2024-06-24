# todo

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
