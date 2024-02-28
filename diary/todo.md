# todo  

---

## 20240228  
### 블로그 정리  

---

## 20240227  
### 블로그 정리  

---

## 20240226  
### 블로그 정리  

---

## 20240225  
### 블로그 정리  

---

## 20240224  
### 블로그 정리  

---

## 20240223  
### 블로그 정리  


---

## 20240222  
### 블로그 정리  

---

## 20240201  
### 인프런 리플렉션  
지난번에 배운 프록시방식도 결국 문제는 controller, service, repository 이렇게 넘어갈때마다 프록시를 코드로 내가 생성해줘야한다. 이떄는 3번  
사실 controller, service, repository 3개다 프록시 코드는 비슷한데 이를 묶을 수 있는 방법은 없을까.

```java
@Test
void reflection0() {
Hello target = new Hello();
//공통 로직1 시작
log.info("start");
String result1 = target.callA(); //호출하는 메서드가 다름
log.info("result={}", result1);
//공통 로직1 종료
//공통 로직2 시작
log.info("start");
String result2 = target.callB(); //호출하는 메서드가 다름
log.info("result={}", result2);
//공통 로직2 종료
}
```
이런 코드 보면 모양이 정말 비슷하지만 함수 callA callB를 동적으로 바꾸기는 뭔가 힘들다 이때 이용할 수 있는 기술이 리플렉션  

```java
Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");
Hello target = new Hello()
//callA 메서드 정보
Method methodCallA = classHello.getMethod("callA");
Object result1 = methodCallA.invoke(target)
```
이런식으로 어떤위치의 어떤함수인지 냅다 적으면 그 실제 class 정보를 가져올 수 있고
내부의 함수명을 String으로 적으면서 함수를 꺼낼 수 있다.  
내가 실행할 class의 인스턴스만 만들어서 그함수 실행시켜줘 하면 실행가능 이것으로 동적으로 함수도 실행가능하다.  
다만 String으로 적는것에서 보면 컴파일에러로 함수명 잘못적은거는 확인이 불가능하기에 실제서비스에서는 조심해야한다.  

### JDK동적 프록시

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {
private final Object target;
public TimeInvocationHandler(Object target) {
this.target = target;
}
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws
Throwable {
log.info("TimeProxy 실행");
long startTime = System.currentTimeMillis();
Object result = method.invoke(target, args);
long endTime = System.currentTimeMillis();
long resultTime = endTime - startTime;
log.info("TimeProxy 종료 resultTime={}", resultTime);
return result;
}
}
```
리플렉션을 이용한 동적프록시 JDK버전이다.  
지정된 인터페이스 만들어서 그것을 구현하며 동작가능한데 `invoke`에 대상 객체, 함수, 함수의 매개변수들등 이런거 정해진거 구현하며 함수사에이 실행할거 넣어주면 된다.  
다만 JDK버전은 실재로직이 인터페이스로 구현되어있어야 가능하다는거같다.(이해 확실하지않음)  
인터페이스를 생성해주면서 프록시를만들기때문에 그렇다.  

### CGLIB  
JDK CGLIB 보다 더 편한 기술이 있기때문에 이해만해도 좋다.  
CGLIB 기술도 JDK와 비슷하긴하다  

```java
public interface MethodInterceptor extends Callback {
Object intercept(Object obj, Method method, Object[] args, MethodProxy
proxy) throws Throwable;
}
```
이런걸 구현하면서 구현할 수 있는거같은데 얘는 실체를 상속받으면서 프록시를 생성해주기 때문에 이미 인퍼테이스 없이 구현된 기능도 프록시 가능하게 만들 수 있다. 다만 상속이기에 final로 되어있는 class, 함수등은 예외가 발생하는등의 문제가 있다.  
  
이렇게 인터페이스로 되어있으면 jdk로 그냥 class만 있으면 CGLIB로 만들어주면 좋지않을까? 해서 나오는것이 다음에 배울것  


---

### 프로그래머스 여행결로 DFS 답(내가푼거 아님 정답지)

```java
import java.util.*;

class Solution {

    static ArrayList<String> list = new ArrayList<>();
    static boolean useTickets[];

    public String[] solution(String[][] tickets) {
        useTickets = new boolean[tickets.length];

        dfs(0, "ICN", "ICN", tickets);

        Collections.sort(list);

        return list.get(0).split(" ");
    }

    static void dfs(int depth, String now, String path, String[][] tickets){
        if (depth == tickets.length) {
            list.add(path);
            return;
        }

        for (int i = 0; i < useTickets.length; i++) {
            if (!useTickets[i] && now.equals(tickets[i][0])) {
                useTickets[i] = true;
                dfs(depth+1, tickets[i][1], path + " " +tickets[i][1], tickets);
                useTickets[i] = false;
            }
        }
    }
}
```


### 프로그래머스 게임 맵 최단거리 답(내가푼거 아님 정답지)

```java
import java.util.LinkedList;
import java.util.Queue;

public class Main {
    public static void main(String[] args) {
        int[][] maps = {{1,0,1,1,1},{1,0,1,0,1},{1,0,1,1,1},{1,1,1,0,1},{0,0,0,0,1}};
        int[][] maps2 = {{1,0,1,1,1},{1,0,1,0,1},{1,0,1,1,1},{1,1,1,0,0},{0,0,0,0,1}};
        Solution solution = new Solution();

        System.out.println("최종결과" + solution.solution(maps));



    }
    static class Solution {
        static int answer = -1;
        static boolean[][] visited;
        // 상 하 좌 우
        static int[] mrow = {-1, 1, 0, 0};
        static int[] mcol = {0, 0, -1, 1};


        public int solution(int[][] maps) {
            visited = new boolean[maps.length][maps[0].length];

            bfs(0, 0, maps);

            return answer;
        }



        public void bfs(int row, int col, int[][] maps) {
            Queue<Node> q = new LinkedList<>();

            q.add(new Node(0, 0, 1));
            visited[0][0] = true;

            while (!q.isEmpty()) {
                Node cur = q.poll();


                // System.out.printf("row=%d, col=%d, count = %d\n", cur.row, cur.col, cur.count);

                if (cur.row == maps.length - 1 && cur.col == maps[0].length - 1) {
                    answer = cur.count;
                    // System.out.println("catch");
                    return;
                }
                for (int i = 0; i < 4; i++) {
                    int nextRow = cur.row + mrow[i];
                    int nextCol = cur.col + mcol[i];

                    if (canMove(nextRow, nextCol, maps)) {
                        // System.out.printf("add row=%d, col=%d, count = %d\n", cur.row, cur.col, cur.count);
                        visited[nextRow][nextCol] = true;
                        q.add(new Node(nextRow, nextCol, cur.count + 1));
                    }

                }
            }
        }

        class Node {
            int row;
            int col;
            int count;

            public Node(int row, int col, int count) {
                this.row = row;
                this.col = col;
                this.count = count;
            }
        }

        public boolean canMove(int row, int col, int[][] maps) {
            return row >= 0 && row < visited.length && col >= 0 && col < visited[0].length
                    && !visited[row][col] && maps[row][col] != 0;
        }
    }
}
```

---

## 20240129  
### 인프런 동시성문제 쓰레드풀  
쓰레드 - 내 코드를 실행하는 묶음  
같은 코드를 여러개 요청한다면 1번째 요청 - 1쓰레드가 묶어서 실행 / 2번째요청 - 2쓰레드가 묶어서 실행  
이때 1,2쓰레드를 동시에 실행시킬 수 있는것  
이런 이때 스프링 싱글톤 변수를 1,2가 동시에 접근해서 수정,조회를 하면 내가 저장한게 안보이고 다른사람이 저장한게 보이는 문재 생길 수 있음  
  
해결법 - 싱글톤을 사용하는 변수에 `ThreadLocal`을 사용해서 변수 선언및 저장  
```java
@Slf4j
public class ThreadLocalService {
private ThreadLocal<String> nameStore = new ThreadLocal<>();

nameStore.set(name);
log.info("조회 nameStore={}",nameStore.get());

//값 제거: `ThreadLocal.remove()
```
이런식으로 `String`을 사용하더라도 `ThreadLocal`을 사용하면 쓰레드마다 별도의 저장소를 만들어서 값 사용가능  
  
다만 값제거는 쓰레드가 기본적으로 자동으로 코드실행완료후 죽으면 상관없는데 톰캣처럼 WAS사용한다면 비용때문에 쓰레드를 죽이지않고 살려서 반환후 재사용하기에  
재사용할때 이전에 저장했던값이 남아있을 수 있어서 코드실행이 끝나면 `ThreadLocal.remove()`이거 꼭 사용해서 비워주기  

### 인프런 템플릿 메서드 패턴  
로그 남기기 코드 보면 핵심 `orderService.orderItem(itemId);`을 가운데 두고 로그남기기코드가 try catch문으로 위아래로 감싸고있음 너무 지저분  
그래서 템플릿 메더스패턴 사용해보는데 이는 추상클레스로 로그남기기코드 쭉 작성해두고 그때그때 바뀌는 핵심코드 `call()` 같은 함수로 선언해두고  
상속받으면서 `call`함수 구현하면서 그 안에다가 `orderService.orderItem(itemId);` 이거 넣는 방식  
```java
@Slf4j
public class SubClassLogic2 extends AbstractTemplate {
@Override
protected void call() {
log.info("비즈니스 로직2 실행");
}
}
```
이런식으로 `AbstractTemplate`여기안에 항상똑같은 로그코드 try catch문 들어있음  
다만 이 방식은 상속의 문제를 그대로 가짐, `extends AbstractTemplate`이렇게 자식 부모 관계까 되면 너무 강력하게 연결되서 `AbstractTemplate`이쪽 부모쪽에서 새로운 함수 정의 하면 자식코드 모두 수정해야하고 더불어서 이 모양애서는 `SubClassLogic2`이 자식 클레스가 부모쪽에서 선언된 함수 사용하는게 없음 구실만 부모자식에 단점만 있는꼴  

### 인프런 전략패턴  
변하지 않는 부분을 그냥 클레스로 박아두고 바뀌는 부분의 `coll()`을 인터페이스로 생성에서 변하지않는 클레스에서 인터페이스 받아서 넣어두고  
사용자는 인터페이스를 구현하면서 만드는 패턴  
*변하지않는 부분*
```java
public class ContextV1 {
private Strategy strategy;
public ContextV1(Strategy strategy) {
this.strategy = strategy;
}
public void execute() {
long startTime = System.currentTimeMillis();
//비즈니스 로직 실행
strategy.call(); //위임
//비즈니스 로직 종료
long endTime = System.currentTimeMillis();
long resultTime = endTime - startTime;
log.info("resultTime={}", resultTime);
}
}
```
*인터페이스*
```java
public interface Strategy {
void call();
}
```
*인터페이스 구현*  
```java
public class StrategyLogic1 implements Strategy {
@Override
public void call() {
log.info("비즈니스 로직1 실행");
}
}
```
*실제사용*
```java
void strategyV1() {
Strategy strategyLogic1 = new StrategyLogic1();
ContextV1 context1 = new ContextV1(strategyLogic1);
context1.execute();
Strategy strategyLogic2 = new StrategyLogic2();
ContextV1 context2 = new ContextV1(strategyLogic2);
context2.execute();
}
```
  
변하지않는 부분에서 의존관계를 `execute`의 매개변수로 받아서 사용할 수도있음 같은 패턴(의도가 같기때문에)  
의도는 상속의 문제를 위임으로(인터페이스)로 해결 인터페이스만 문제없으면 `ContextV1`이 변경되어도 다른 함수쪽에서 변경할것 없음  
알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

### 템플릿 콜백 패턴  
이는 전략패턴과 똑같은데 전략패턴중 변하지않는 부분에서 의존관계를 `execute`의 매개변수로 받아서 사용할 수도있는 패턴을 특히 스프링에서만 템플릿 콜백패턴이라고 한다.  
이는 `JdbcTemplate` , `RestTemplate` , `TransactionTemplate` , `RedisTemplate`등으로 부르는 것들이 템플릿콜백 패턴으로 이루어져 있기때문이다.  

별거는 없다. 변하지않는 부분의 클레스명이 `TimeLogTemplate`이런식으로 템플릿이 붙고 변하는부분의 인터페이스명이 `Callback`으로 되어있다.  

```java
public class TraceTemplate {
private final LogTrace trace;
public TraceTemplate(LogTrace trace) {
this.trace = trace;
}
public <T> T execute(String message, TraceCallback<T> callback) {
TraceStatus status = null;
try {
status = trace.begin(message);
//로직 호출
T result = callback.call();
trace.end(status);
return result;
} catch (Exception e) {
trace.exception(status, e);
throw e;
}
}
}
```
다만 명확하게 `public <T> T execute(String message, TraceCallback<T> callback)`이 함수에서 매개변수로 중간에 실행할 함수를 받아서 실행 - callback  
이러한 방식에 람다까지 쓴다면 
```java
@Service
public class OrderServiceV5 {
private final OrderRepositoryV5 orderRepository;
private final TraceTemplate template;
public OrderServiceV5(OrderRepositoryV5 orderRepository, LogTrace trace) {
this.orderRepository = orderRepository;
this.template = new TraceTemplate(trace);
}
public void orderItem(String itemId) {
template.execute("OrderService.orderItem()", () -> {
orderRepository.save(itemId);
return null;
});
}
}
```
이런식으로 `template.execute("OrderService.orderItem()", () -> {orderRepository.save(itemId);` 정말 최소한으로 줄일 수 있다.  
다만 최종적인 문제는 결국 `controller`든 `service`든 `repository`든 무조건 메인동작을 한줄이라도 수정하긴 해야한다는것 이것이 한계다.  

### 인프런 프록시의 이해  

클라이언트가 요청한 결과를 서버에 직접 요청하는 것이 아니라 어떤 대리자를 통해서 대신 간접적으로 서버에  
요청할 수 있다. 예를 들어서 내가 직접 마트에서 장을 볼 수도 있지만, 누군가에게 대신 장을 봐달라고 부탁할 수도 있다.  
여기서 대신 장을 보는 **대리자를 영어로 프록시(Proxy)**라 한다.  
  
클라이언트 -> 프록시 -> 서버  
이때 클라이언트 서버는 단순히 이용자, 서버개발자가 아니라 요청하는 클레스 - 응답하는 클레스의 관계에서도 적용된다.  
  
재미있는 점은 직접 호출과 다르게 간접 호출을 하면 대리자가 중간에서 여러가지 일을 할 수 있다는 점이다.  
엄마에게 라면을 사달라고 부탁 했는데, 엄마는 그 라면은 이미 집에 있다고 할 수도 있다. 그러면 기대한 것 보다 더 빨리 라면을 먹을 수 있다. (접근 제어, 캐싱)  
아버지께 자동차 주유를 부탁했는데, 아버지가 주유 뿐만 아니라 세차까지 하고 왔다. 클라이언트가 기대한 것 외에 세차라는 부가 기능까지 얻게 되었다. (부가 기능 추가)  
프록시가 다른 프록시 요청가능 (프록시 체인)  
  
이 프록시의 핵심은 클라이언트가 프록시가 사용됐는지도 몰라야한다는것 = 인터페이스등을 사용해서 DI로 클라이언트는 인터페이스를 의존하고 실제 인스턴스는 프록시가 될수도 서버가 될수도있는 방식 사용하는것  

프록시를 통해서 할 수 있는 일은 크게 2가지로 구분할 수 있다.  
접근 제어-권한에 따른 접근 차단,캐싱,지연 로딩  
부가 기능 추가-원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다.(예) 요청 값이나, 응답 값을 중간에 변형한다.,예) 실행 시간을 측정해서 추가 로그를 남긴다)  
  
둘다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 의도(intent)에 따라서 프록시 패턴과 데코레이터 패턴으로 구분한다.  
프록시 패턴: 접근 제어가 목적  
데코레이터 패턴: 새로운 기능 추가가 목적  

---

## 20240125  
### 인프런 스프링  
#### 트랜잭션 전파 실무예시  
이 트랜잭션의 전파의 쉬운 예시는 service - repository 관계이다.  
기본적으로 데이터를 하나씩 조회하는 repository에 @Transactional이 적혀있고 이것들을 사용하는 service단에도 메소드마다 @Transactional을 달아주면서 자연스럽게 트랜잭션전파를 쓰는것  
여기서 어떠한 repository든 하나라도 이상이 생기면 service의 함수 전체 내용이 커밋되지않는 방식은 맞다.  
다만 한가지 유의 사항  
만약 두가지 repository기능중에 하나가 오류가 나더라도 그 상황을 커밋, 저장하고 싶을때(임시데이터 저장의 오류 그냥 넘기로 했을때)  
이럴때 문제인것이 임시저장의 repository의 예외 발생 - service에서 catch예외처리완료 정상흐름 - 그렇지만 전체다롤백됨  
이유는 예외 하나가 발생한것은 내가 잡아서 처리하더라도 전체묶음 @Transactional에서 한번 오류가 발생하면 rollbackOnly라고 트랜잭션에 기입함  
예외는 처리완료해도 저 rollbackOnly마크는 계속 있기때문에 이거보고 전체 롤백  
그래서 이때 repository기능중에 하나가 오류가 나더라도 그 상황을 커밋, 저장하고 싶을때(임시데이터 저장의 오류 그냥 넘기로 했을때) 는 그 오류가 난쪽에 @Transactional(~~~ = REQUIRES_NEW)옵션 적용시켜서 따로 돌아가도록 설정가능  
그외에 서비스 앞단을 하나더 만들어서도 가능 여러방법있음  

#### 로그추적기예제의 필드동기화 동시성 문제  
로그추적기를 만든다. 유저가 요청을 보내면 그 요청에 대한 클레스 사용을 지나갈때마다 출력하며 다시 컨트롤러단으로 돌아올때는 시간과 예외가 있다면 예외도 출력  
이때 이것을 수행하는 클레스를 싱글톤으로 등록해서 사용하면 동시성 문제가 생긴다.  
즉 원래 1사람의 요청을 수행하는데 총 1초가 걸린다면 또다른 사람이 1초가 지나기전에 다시 요청을하면  
로그추적지 클레스가 첫번째사람의 깊이를 0,1,2 이렇게 저장하고 있다가 이것이 다시 0으로 초기화 되기전에 다른사람이 요청하면 2부터 시작해서 ,3,4 이런식으로 증가하게되는 = 싱글톤의 동시성 문제가 발생한다.  

### 인프런 코테

#### 1-1
오랜만의 까먹은거 주의  
```java
Scanner sc = new Scanner(System.in); //스캐너받기
String str = sc.next();
char input = sc.next().charAt(0); 
for (char i : str.toUpperCase().toCharArray()) //향상된 for문

String비교시에는 str.equals()쓰는거 주의 
```


---
## 20240124  

인프런  

### 인프런  
#### 트랜잭션 적용범위  
자세한것 우선  

#### 트랜잭션 AOP주의사항  
내부 호출에는 적용 안됨  
```java
class ~~
public one () {
two();
}

@Transactional
public two() {
sout~
}
```
이런상황에서 two();는 this.two()가 생략된것 즉 this = 자신의 인스턴스 본체 = 프록시 객체가 아닌 진짜너셕을 호출한다는 뜻  
해결법은 클래스 나누기  
  
의존관계주입, 생성자 직후 @Postconstruce의 동작은 @Transcational은 적용이 안됨 트랜잭션 프록시등의 기능이 더 나중에 실행되서 안됨 

#### 트랜잭션 옵션  
트랜잭션은 커밋 롤백을 위한 틀이기도 하기에 어떤 상황에서 무엇을 선택할지 옵션등 여러가지 있음 readonly사용하면 어디부분에서 최적화 되는지도 설명  

#### 트랜잭션 커밋롤백  
런타임 예외는 롤백하지만 체크예외는 커밋함 이유는 개발자들이 체크예외는 시스템상의 문제가 아니고 예를 들어 사용자가 잘못한 상황에서 일단 값을 저장하고 다른 안내를 하는등의 방향으로 사용가능 그래서 커밋이 기본값  

#### 트랜잭션 전파  
트랜잭션 안에서 다시 트랜잭션이 시작할경우 그냥 가장 밖의 트랜잭션에 함께가게됨 즉 가장 밖의 시작-커밋 이렇게 한번만 묶어서 들어가는것  

#### REQUIRES_NEW  
내부 트랜잭션에서 REQUIRES_NEW설정을 해주면 얘는 아에 따로 새로운 커넥션 가져와서 완전히 별개로 작동가능  
그외에 옵션들 몇개더있  

---

## 20240122  
cicd  

---

## 20231212

- [ ] 코테1일차
- [ ] 블로그복붙


---

## 20231212

- [ ] 인프런
- [ ] 코테

---

## 20231211

- [x] 캡디플젝마무리
- [ ] 인프런
- [ ] 코테

---

## 20231210  

- [x] 캡디플젝마무리  

---

## 20231209  
1. 개인플젝
1. 코테 1시간
1. 인프런 1시간  

---

## 20231208  
1. spa api o
1. 개인플젝 배포설정  

---

## 20231206  
1. kirukiru초기설정 o
1. 자바로 목적지 입력하면 해당 위지의 위도경도 리턴 함수 방식 검색 o

---
