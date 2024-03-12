# todo  


## 20240307  
### 스프링이 제공하는 빈 후처리기  
스프링 라이브러리에 `spring-boot-starter-aop`을 추가하면 프록시 팩토리를 자동으로 `bean`으로 등록해주며 이전에는 프록시 팩토리를 `bean`으로 등록하면서 페키지 설정 이런거 좀 해줬는데 아에 할 필요가 없다.  

생각해보면 어드바이저에는 포인트컷이 있어서 이걸 보고 찾아서 해당하는것만 등록해주면 되는것이다.  

설정파일
```java
@Bean
public Advisor advisor3(LogTrace logTrace) {
 AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
 pointcut.setExpression("execution(* hello.proxy.app..*(..)) && !execution(* 
hello.proxy.app..noLog(..))");
 LogTraceAdvice advice = new LogTraceAdvice(logTrace);
 //advisor = pointcut + advice
 return new DefaultPointcutAdvisor(pointcut, advice);
}

```
이렇게 하나만 등록해주면 된다. 포인트 컷에서 `"execution(* hello.proxy.app..*(..)) && !execution(* hello.proxy.app..noLog(..))")`  
좀 특이한 수식을 사용했는데 페키지기반으로 `nolog`함수만 제외하고 어드바이스를 적용 시켜달라는 뜻이다. 이걸보고 그냥 적용 시켜준다.  

중요한점은 여러 어드바이스를 적용해도 한 클레스에서는 프록시가 단 하나만 생성되며 포인트컷 확인작업이 2번 실행된다는것이다.  

1. 모든 생성되는 `bean`을 모든 등록된 어드바이저의 포인트컷과 대조해서 단 하나의 함수라도 사용하면 일단 프록시 펙토리돌려서 프록시를 어드바이저 넣어 만든다. 없다면 그냥 실제 그녀석을 `bean`으로 등록
2. 만약 한개 더 적용시켜야할 어드바이저가 있다면 이미 존재하는 프록시 팩토리에 어드바이저만 하나더 추가한다. (이것이 중요)  
3. 이렇게 모든 `bean`이 완성되고나서 실제 함수가 실행될때 그 함수가 어드바이스를 적용해야하는 놈인지 아닌지 다시 포인트컷을 통해 확인후 적용한다.

1,3번에서 포인트컷을 통한 확인작업이 2번 진행되며 2번으로 프록시 자체는 단 하나만 생성된다는것이 중요하다.  

---

### 프로그래머스 k번째수 정렬

배운것
* Arrays.copyOfRange 암기  
* `numberList.stream().mapToInt(Integer::intValue).toArray();`스트림 익숙해지기

배열 array의 i번째 숫자부터 j번째 숫자까지 자르고 정렬했을 때, k번째에 있는 수를 구하려 합니다.  
예를 들어 array가 1, 5, 2, 6, 3, 7, 4  
i = 2, j = 5, k = 3이라면  
array의 2번째부터 5번째까지 자르면 5, 2, 6, 3입니다.  
1에서 나온 배열을 정렬하면 2, 3, 5, 6입니다.  
2에서 나온 배열의 3번째 숫자는 5입니다.  
배열 array, i, j, k를 원소로 가진 2차원 배열 commands가 매개변수로 주어질 때, commands의 모든 원소에 대해 앞서 설명한 연산을 적용했을 때 나온 결과를 배열에 담아 return 하도록 solution 함수를 작성해주세요.  

arr1(1, 5, 2, 6, 3, 7, 4) arr2((2, 5, 3), (4, 4, 1), (1, 7, 3)) 결과(5, 6, 3)  

내가풀기  
```java
import java.util.*;

public class Main {

	public static int[] arr1 = {1, 5, 2, 6, 3, 7, 4};
	public static int[][] arr2 = {{2, 5, 3}, {4, 4, 1}, {1, 7, 3}};

	public static void main(String[] args) {
		Solution solution = new Solution();
		for (int num : solution.solution(arr1, arr2)) {
			System.out.println("result" + num);

		}
	}

	public static class Solution {

		public int[] solution(int[] array, int[][] commands) {
			ArrayList<Integer> numberList = new ArrayList<>();
			for (int[] arr : commands) {
				numberList.add(resultOne(array, arr));
			}
			int[] answer = numberList.stream().mapToInt(Integer::intValue).toArray();
			return answer;
		}

		public int resultOne(int[] array, int[] command) {
			int[] slicedArray = Arrays.copyOfRange(array, command[0]-1, command[1]);
			Arrays.sort(slicedArray);
			return slicedArray[command[2] - 1];
		}

	}

}
```
다른정답을 봐도 이정도에서 정리하면 될거같다.  
완전히 모르는건 아니였는데 `Arrays.copyOfRange`이거랑 스트림 사용법 검색없이 써야하니 자꾸 익숙해져야한다.  

---
### 네컷아카이브 집에서 배포
우분투 ISO다운
버추얼박스 설치후 생성  
언어한번바꿔야 터미널실행가능?  

```
우분투 로그인 계정 root권한 등록
su -
usermod -aG sudo 사용자명

(다시로그인)
su - 사용자명

sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo systemctl status docker

도커 다운 하려면 도커 명령어 다시 root권한 등록
su -
sudo usermod -aG docker 사용자명
su - 사용자명

sudo docker pull witwint/filmarchive:20240306-1 이런식으로 걍 저장소이름으로 다운가능
```

이후에 저번처럼 run시키면 동작 잘됨

#### 포트포워딩  
1도커 실행할때 8000:8000연결
2버츄얼박스 ip비워두고 8000연결하니 전부다 되는듯
3노트북방화벽 네트워크 고급설정가니 인바운드설정에 그냥 TCP,UDP등록가능
4공유기 똑같이 포트포워딩 도착지(노트북이사용하는 와이파이의 주소로)
5모뎀 브릿지 설정으로 바꿈, 이거 바꾸니깐 외부주소 아에바뀜? 뭔가 내가 손못쓰는게 있나 저번에 그냥 여기서 포트포워딩하려니 안됨  

---

## 20240306  
### 빈 후처리기  
빈등록은 컴포넌트 스캔이나 설정파일을 통해서 등록할 수 있다.  
이전에는 설정파일을 만들면서 스프링빈에 등록해줄때 내가 프록시를 만들어서 연결해준후에 프록시를 등록해줬는데 너무 코드가 복잡했다.  

이에 스프링은 빈 후처리기라는것을 지원한다.  
빈이 등록될때는 설정파일이나 컴포넌트 스캔을 통해 1. 객체를 생성하고, 2 객체를 빈 저장소에 등록한다.  
이때 생성이 완료된것들을 하나하나 확인해가면서 조작,변경할것을 수정후에 빈 저장소에 등록할 수 있는 기능이다.  

빈 후처리기 간단코드
```java
public class BeanPostProcessorTest {
 @Test
 void postProcessor() {
 ApplicationContext applicationContext = new
AnnotationConfigApplicationContext(BeanPostProcessorConfig.class);
 //beanA 이름으로 B 객체가 빈으로 등록된다.
 B b = applicationContext.getBean("beanA", B.class);
 b.helloB();
 //A는 빈으로 등록되지 않는다.
 Assertions.assertThrows(NoSuchBeanDefinitionException.class,
 () -> applicationContext.getBean(A.class));
 }
 @Slf4j
 @Configuration
 static class BeanPostProcessorConfig {
 @Bean(name = "beanA")
 public A a() {
 return new A();
 }
 @Bean
 public AToBPostProcessor helloPostProcessor() {
 return new AToBPostProcessor();
 }
 }
 @Slf4j
 static class A {
 public void helloA() {
 log.info("hello A");
 }
 }
 @Slf4j
 static class B {
 public void helloB() {
 log.info("hello B");
 }
 }
 @Slf4j
 static class AToBPostProcessor implements BeanPostProcessor {
 @Override
 public Object postProcessAfterInitialization(Object bean, String
beanName) throws BeansException {
 log.info("beanName={} bean={}", beanName, bean);
 if (bean instanceof A) {
 return new B();
 }
 return bean;
 }
 }
}
```
`ApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanPostProcessorConfig.class);` 수동으로 빈 컨테이너에 설정파일을 집어넣는 코드이다.  

` static class AToBPostProcessor implements BeanPostProcessor` 이 인터페이스를 구현하면서 빈 후처리기를 사용할 수 있다.  
`public Object postProcessAfterInitialization(Object bean, StringbeanName)`인터페이스 함수중에 이를 구현하면 되는데 `bean`에 원래, 실제로 등록될 녀석이 들어있다. 그대로 `return bean`해버리면 기존의 등록 그대로 작동하는데 이 매개변수를 확인해서 조작해서 바꾸고싶은 녀석을 `return`해주면 그녀석이 빈으로 등록되게된다.  

#### 실사용  

실사용에서는 

`public class PackageLogTraceProxyPostProcessor implements BeanPostProcessor`이 class에다가 그냥 
`public Object postProcessAfterInitialization(Object bean, StringbeanName)`에  
```java
if (!packageName.startsWith(basePackage)) {
 return bean;
 }
 //프록시 대상이면 프록시를 만들어서 반환
 ProxyFactory proxyFactory = new ProxyFactory(bean);
 proxyFactory.addAdvisor(advisor);
 Object proxy = proxyFactory.getProxy();
 return proxy;
 }
```
이런 코드 넣어서 만들어준후에 `PackageLogTraceProxyPostProcessor`얘를 bean으로 딱하나만 등록해주면 컴포넌트 스캔녀석들은 그냥 자동으로 되고 판별에 의해 필요한것들은 프록시 개체를 만들어서 그게 빈으로 등록되게 만들 수 있다.  
(코드참조)  


---

### 프로그래머스 같은숫자는 싫어 스텍/큐  

배운것
* Deque
* for문 제한사항 매 바퀴마다 초기화
* 

문제 설명  배열 arr가 주어집니다. 배열 arr의 각 원소는 숫자 0부터 9까지로 이루어져 있습니다. 이때, 배열 arr에서 연속적으로 나타나는 숫자는 하나만 남기고 전부 제거하려고 합니다. 단, 제거된 후 남은 수들을 반환할 때는 배열 arr의 원소들의 순서를 유지해야 합니다.  
예를 들면, 1133011 이면 1301 을 return 합니다.  
배열 arr에서 연속적으로 나타나는 숫자는 제거하고 남은 수들을 return 하는 solution 함수를 완성해 주세요.  
제한사항 배열 arr의 크기 : 1,000,000 이하의 자연수 배열 arr의 원소의 크기 : 0보다 크거나 같고 9보다 작거나 같은 정수  

내가풀기
```java
import java.util.*;

public class Main {

	public static int[] arr1 = {1, 1, 3, 3, 0, 1, 1};
	public static int[] arr2 = {4, 4, 4, 3, 3};

	public static void main(String[] args) {
		Solution solution = new Solution();
		for (int num : solution.solution(arr2)) {
			System.out.println("result" + num);

		}
	}

	public static class Solution {

		public int[] solution(int[] arr) {
			Deque<Integer> deque = new LinkedList<>();
			Deque<Integer> dequeResult = new LinkedList<>();

			for (int num : arr) {
				deque.addFirst(num);
			}
			dequeResult.addFirst(deque.pollLast());

			int size = deque.size();
			for (int i = 0; i < size; i++) {

				if (dequeResult.getFirst() == deque.getLast()) {
					deque.pollLast();

				} else {
					dequeResult.addFirst(deque.pollLast());

				}
			}

			// 세트를 다시 배열로 변환
			int[] result = new int[dequeResult.size()];
			int size2 = dequeResult.size();
			for (int i = 0; i < size2; i++) {
				result[i] = dequeResult.pollLast();
			}

			return result;
		}

	}

}
```
파이썬에도 Deque가 있는데 자바에도 있나해서 찾아봤더니 있었다. 역시 편하다  
쉬운문제인데 삽질한데 for문에 `i < size`이 제한사항을 `i < deque.size()`로 하니깐 첫바퀴의 숫자가 아니라 돌면서 데큐에서는 원소가 빠져나가며 size가 줄어드는데 그게 반영되서 이상하게 for문이 돌았다.  
여기서 삽질을 하다니...  

다른 정답을 보면 핵심 자체는 비슷하긴한데 처음 `int[]`를 받으면서 한번에 처리할 수 있을거같긴하다.  


---

## 20240305  
### 프록시 팩토리 적용과 한계점  

프록시 펙토리를 실제로 적용한다면  

```java
@Slf4j
public class LogTraceAdvice implements MethodInterceptor {
 private final LogTrace logTrace;
 public LogTraceAdvice(LogTrace logTrace) {
 this.logTrace = logTrace;
 }
 @Override
 public Object invoke(MethodInvocation invocation) throws Throwable {
 TraceStatus status = null;
 
 try {
 Method method = invocation.getMethod();
 String message = method.getDeclaringClass().getSimpleName() + "."
 + method.getName() + "()";
 status = logTrace.begin(message);
 //로직 호출
 Object result = invocation.proceed();
 logTrace.end(status);
 return result;
 } catch (Exception e) {
 logTrace.exception(status, e);
 throw e;
 }
 }
}
```
우리가 사용할 어드바이스를 코드로 작성해준후에  

```java
@Slf4j
@Configuration
public class ProxyFactoryConfigV2 {
 @Bean
 public OrderControllerV2 orderControllerV2(LogTrace logTrace) {
 OrderControllerV2 orderController = new
OrderControllerV2(orderServiceV2(logTrace));
 ProxyFactory factory = new ProxyFactory(orderController);
 factory.addAdvisor(getAdvisor(logTrace));
 OrderControllerV2 proxy = (OrderControllerV2) factory.getProxy();
 log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), 
orderController.getClass());
 return proxy;
 }
 @Bean
 public OrderServiceV2 orderServiceV2(LogTrace logTrace) {
 OrderServiceV2 orderService = new
OrderServiceV2(orderRepositoryV2(logTrace));
 ProxyFactory factory = new ProxyFactory(orderService);
 factory.addAdvisor(getAdvisor(logTrace));
 OrderServiceV2 proxy = (OrderServiceV2) factory.getProxy();
 log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), 
orderService.getClass());
 return proxy;
 }
 @Bean
 public OrderRepositoryV2 orderRepositoryV2(LogTrace logTrace) {
 OrderRepositoryV2 orderRepository = new OrderRepositoryV2();
 ProxyFactory factory = new ProxyFactory(orderRepository);
 factory.addAdvisor(getAdvisor(logTrace));
 OrderRepositoryV2 proxy = (OrderRepositoryV2) factory.getProxy();
 log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), 
orderRepository.getClass());
 return proxy;
 }
 private Advisor getAdvisor(LogTrace logTrace) {
 //pointcut
 NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
 pointcut.setMappedNames("request*", "order*", "save*");
 //advice
 LogTraceAdvice advice = new LogTraceAdvice(logTrace);
 //advisor = pointcut + advice
 return new DefaultPointcutAdvisor(pointcut, advice);
 }
}
```
빈등록을 프록시를 반환하는 형태로 만들어주면 적용이 완료된다.  

프록시팩토리를 통해서 우리는 넣고싶은 기능을 하나의 클레스에서 작성해준후에 DI를 조합하는 설정 class파일에서 조립만 해주면된다.  
그러나 문제는 기존에 이러한 설정도 귀찮고 힘들어서 이런 설정없이 컴포넌트 스캔을 썼는데 다시 적어줘야하는 문제가 생기고, 빈이 많아질수록 결국 손으로 적어줘야하는 코드도 많아진다.  
이를 해결하기위해 빈 후처리기 라는것이 나오게된다.  

---

### 코테 프로그래머스 전화번호목록 해시  

배운것  
* `Arrays.sort(phone_book);`정렬방식
* `phone_book[j].startsWith(phone_book[i])` startsWith
* 해쉬효율
  
전화번호부에 적힌 전화번호 중, 한 번호가 다른 번호의 접두어인 경우가 있는지 확인하려 합니다. 전화번호가 다음과 같을 경우, 구조대 전화번호는 영석이의 전화번호의 접두사입니다.  
구조대 : 119 박준영 : 97 674 223 지영석 : 11 9552 4421  
전화번호부에 적힌 전화번호를 담은 배열 phone_book 이 solution 함수의 매개변수로 주어질 때, 어떤 번호가 다른 번호의 접두어인 경우가 있으면 false를 그렇지 않으면 true를 return 하도록 solution 함수를 작성해주세요.  

#### 혼자시도  
```java
import java.util.Arrays;

public class Main {

	public static String[] phone_book1 = {"119", "97674223", "1195524421"};
	public static String[] phone_book2 = {"123","456","789"};
	public static String[] phone_book3 = {"1","34","132"};

	public static void main(String[] args) {
		Solution solution = new Solution();
		System.out.println(solution.solution(phone_book3));
	}

	public static class Solution {

		public boolean solution(String[] phone_book) {
			Arrays.sort(phone_book);

			boolean answer = true;
			//test
			System.out.println("길이순으로 정렬된 배열:");
			for (String str : phone_book) {
				System.out.println(str);
			}

			for (int i = 0; i < phone_book.length; i++) {
				for (int j = i+1; j < phone_book.length; j++) {
					if (phone_book[j].startsWith(phone_book[i])) {
						answer = false;
					}

				}
			}

			return answer;
		}

	}

}
```
좀 알아가면서 `startsWith`을 처음알았다. 대상 문자열이 특정 문자 또는 문자열로 시작하는지 체크하는 함수이다.  

#### 속도개선
```java
Arrays.sort(phone_book);

for(int i = 0; i < phone_book.length -1; i++) {
 if (phone_book[i+1].startsWith(phone_book[i]))
  return false;
}
```
이식만 있어도된다. `Arrays.sort`가 숫자 오름차순 - 문자열길이오름차순으로 2조건에 정렬해주기에 예를들어
`1, 34, 132`이런 배열이 있다면
`1, 132, 34`순으로 정렬해준다. 그렇기에 정렬한번하고 내뒤의것만 확인하면되는것이다.  

#### 해쉬이용
```java
import java.util.*;

class Solution {
    public boolean solution(String[] phone_book) {
        Map<String, String> map = new HashMap<>();

//값을 전부다 해쉬에 집어넣고
        for (int i = 0; i < phone_book.length; i++) {
            map.put(phone_book[i], phone_book[i]);
        }
        
        for (int i = 0; i < phone_book.length; i++) {
//전화번호 하나하나를
            for (int j = 1; j < phone_book[i].length(); j++) {
// 12345 => 1,12,123,1234 이렇게 반복 잘라서 검증
                String subString = phone_book[i].substring(0, j);

//1,12,123이런 하나하나들이 map에 들어있는지 통째로 검증
                if (map.containsKey(subString)) {
                    return false;
                }
            }
        }
        
        return true;
    }
}
```
이게 정말 더 효율적인가 직관적이지는 않지만 `map.containsKey(subString)`이것이 효율적이긴 한거같다.  


---

## 202240304  
### 프록시 팩토리  

우리의 시스템로직에 앞뒤로 부가기능을 넣기위해서 프록시를 도입해었다. 다만 실제 class 가 있다면 CGLIB를 아용해서 조금 편하게 사용했고,  
인터페이스가 존재한다면 JDK동적 프록시를 통해서 프록시를 도입했다.  
이를 공통적으로 적용하기 위해서 프록시 팩토리라는것을 스프링은 도입했다.  

쉽게 생각하면 JDK 동적프록시, CGLIB두개를 감싸는 로직을 만들어서 들어오는 target class가 어떤모양인지에 따라서 알아서 적용해준다.  

```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {
 @Override
 public Object invoke(MethodInvocation invocation) throws Throwable {
 log.info("TimeProxy 실행");
 long startTime = System.currentTimeMillis();
 Object result = invocation.proceed();
 long endTime = System.currentTimeMillis();
 long resultTime = endTime - startTime;
 log.info("TimeProxy 종료 resultTime={}ms", resultTime);
 return result;
 }
}

```
`public class TimeAdvice implements MethodInterceptor` 이쪽에 로직 앞뒤로 적용하고픈 기능을 구현해주면되고 실제 기존기능은  
` Object result = invocation.proceed();`이렇게만 써두면 된다.  

```java
void interfaceProxy() {
 ServiceInterface target = new ServiceImpl();
 ProxyFactory proxyFactory = new ProxyFactory(target);
 proxyFactory.addAdvice(new TimeAdvice());
 ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
 proxy.save();
```
이런식으로 초기에 조합해주면 target형태에따라서 알아서 작동하게해준다.  

기본적으로 프록시 팩토리 - 실제 class만 있으면 CGLIB, 인터페이스 있으면 JDK동적 프록시이다  
근데 인터페이스 있어도 프록시 팩토리쪽에서 `proxyTargetClass = true` 한줄넣어주면 인터페이스 있어도 CGLIB사용으로 프록시 만들어준다.  
실무에서 종종 등장하는 패턴이기에 주의  
스프링부트에서는 `proxyTargetClass = true`이게 기본적용이기에 항상 CGLIB만 보던것이다.  

### 포인트컷, 어드바이스, 어드바이저  
이런 프록시 패턴을 보면 역할을 나눌 수 있다.  
어떠한 target들에게 적용할것인가 = 포인트컷  
어떠한 기능들을 적용할것인가 = 어드바이스  
목표 target하나 + 적용기능하나 = 어드바이저  

```java

@Test
@DisplayName("스프링이 제공하는 포인트컷")
void advisorTest3() {
 ServiceImpl target = new ServiceImpl();
 ProxyFactory proxyFactory = new ProxyFactory(target);
 NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
 pointcut.setMappedNames("save");
 DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new
TimeAdvice());
 proxyFactory.addAdvisor(advisor);
 ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
 proxy.save();
 proxy.find();
}
```
위쪽에서 배운 프록시펙토리에 
```
NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
 pointcut.setMappedNames("save");
 DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new
TimeAdvice());
```
이부분이 추가된느낌이다.  
위에서 배운 그냥 어드바이스를 넣는게 포인트컷을 allTrue하는 느낌으로 자동적용해준다.  


#### 여러 어드바이저 적용
```java
@Test
@DisplayName("하나의 프록시, 여러 어드바이저")
void multiAdvisorTest2() {
 //proxy -> advisor2 -> advisor1 -> target
 DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, 
new Advice2());
 DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(Pointcut.TRUE, 
new Advice1());
 ServiceInterface target = new ServiceImpl();
 ProxyFactory proxyFactory1 = new ProxyFactory(target);
 proxyFactory1.addAdvisor(advisor2);
 proxyFactory1.addAdvisor(advisor1);
 ServiceInterface proxy = (ServiceInterface) proxyFactory1.getProxy();
 //실행
 proxy.save();
}
```

`proxyFactory1.addAdvisor(advisor2);`사용해서 추가해주면 등록된만큼 적용되고 실제 target을 부르게된다.  
이것이 중요한 이유는 나중에 좀더 편하게 사용하면 `roxyFactory proxyFactory1 = new ProxyFactory(target);`이 프록시 펙토리가  
`advisor`의 개수만큼 새로 `new`로 생성된다고 착각할 수 있는데 위와같이 하나의 프록시 펙토리헤서 여러개를 적용후 `target`을 부르는 형식으로 진행된다.  


---

## 20240303
### 블로그 코테

---

## 20240302  
### 버추얼박스, 포트포워딩  

리눅스 환경 도커이미지를 남는 노트북에서 서버로 돌려보려 한 시도에서 포트포워딩을 한번 정리할 수 있어서 적어본다.  

노트북에서 윈도우와 개별된 환경에서만 서버를 돌리고싶은 마음이 생겨서 버추얼 박스를 사용했다.  
버추얼 박스는 윈도우 위에서 완전히 개별된 os를 실행할 수 있는데 주로 리눅스 우분투를 많이 사용한다.  
  
버추얼 박스에서 우분투를 설치하고 도커를 설치해서 이미지를 받아서 실행해주기만 하면 되는데  
문제는 이 컴퓨터를 서버로 사용하기위해서는 외부사람이 이 컴퓨터에 접근할 수 있어야한다.  

가정집에서는 주로 사설아이피를 사용하는데 실제 우리집IP로 요청을하면 공유기를 거쳐서 우리집IP.포트 요청이 어떤 기기에포트로 요청되는지 내가 설정해 줘야한다.  

대략 공유기단 에서의 포트포워딩이라면 알고 있었는데. 위의 서버를 만들면서 우리집이 생각보다 복잡한 환경이라 기억에 남았다.  

모뎀(skt) - 공유기 - 노트북 방화벽 - 버추얼박스 - 도커  
이 순서로 내가 설정해주지않으면 특정 포트로의 요청을 다막고 있어서 내가 전부다 포트포워딩으로 열어줘야했다.  
  
이걸 해보면서 이런 포트포워딩을 열어뒀을때 내가 켜둔 서버로만 데이터가 오가긴 하겠다만 뭔가 보안적으로 문제가 없는지 궁금해지긴 했다.  

---

## 20240301
### 블로그 정리  

---

## 20240229  
### 블로그 정리  

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
