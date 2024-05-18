# todo  

---
## 20240317  
### 프로그래머스 타겟넘버 DFS/BFS 2단계  

배운것
* 문제 접근방식
* DFS, BFS복습

내코드  
```java
public class Main {

	public static int[] arr1 = {4,1,2,1};
	public static int arr2 = 4;

	public static void main(String[] args) {
		Solution solution = new Solution();

		System.out.println(solution.solution(arr1,arr2));

	}

	public static class Solution {

		static int count = 0;

		public int solution(int[] numbers, int target) {
			int depth = 0;
			DFS(numbers, target, depth);
			return count;
		}

		public void DFS(int[] numbers, int target , int depth) {
			if (depth == numbers.length) {
				int sum = 0;
				for (int num : numbers) {
					sum += num;
				}
				if (sum == target) {
					count++;
				}
				return;
			}
			DFS(numbers, target, depth+1);
			numbers[depth] = -numbers[depth];
			DFS(numbers, target, depth+1);
		}

	}

}
```
DFS 제귀로 풀었다. 첫 접근이 어려웠다. 처음에 분기를 +++++ > (-++++,+++++)이런식으로 나누려다 답이 아닌거같아서 검색으로 힌트를 얻었다.  
0 > (+,-) > (++, +-, -+, --) > (+++ ...) 어런식으로 한번에 모든 부호가 아니라 내려갈 수록 위쪽 원소들의 부호를 나누는방식이란걸..  
이런 문제는 접근이 어려운거같다.  

---

## 20240318
### 프로그래머스 의상 해쉬 2단계  

배운것
* 해쉬이점
* 해쉬순회방법
* 경우의 수 연산

코니는 매일 다른 옷을 조합하여 입는것을 좋아합니다.  예를 들어 코니가 가진 옷이 아래와 같고, 오늘 코니가 동그란 안경, 긴 코트, 파란색 티셔츠를 입었다면 다음날은 청바지를 추가로 입거나 동그란 안경 대신 검정 선글라스를 착용하거나 해야합니다.  
코니는 각 종류별로 최대 1가지 의상만 착용할 수 있습니다. 예를 들어 위 예시의 경우 동그란 안경과 검정 선글라스를 동시에 착용할 수는 없습니다. 착용한 의상의 일부가 겹치더라도, 다른 의상이 겹치지 않거나, 혹은 의상을 추가로 더 착용한 경우에는 서로 다른 방법으로 옷을 착용한 것으로 계산합니다. 코니는 하루에 최소 한 개의 의상은 입습니다.  
input{{"yellow_hat", "headgear"}, {"blue_sunglasses", "eyewear"}, {"green_turban", "headgear"}} 답5  

내풀이  
```java
import java.util.*;

public class Main {

	public static String[][] arr1 = {{"yellow_hat", "headgear"}, {"blue_sunglasses", "eyewear"}, {"green_turban", "headgear"}};


	public static void main(String[] args) {
		Solution solution = new Solution();

		System.out.println(solution.solution(arr1));

	}

	public static class Solution {
		public int solution(String[][] clothes) {
			Map<String, Integer> map = new HashMap<>();
			int answer = 1;
			for (String[] arr : clothes) {
				if (map.containsKey(arr[1])) {
					map.put(arr[1], map.get(arr[1]) + 1);
				} else {
					map.put(arr[1], 1);
				}
			}
			Iterator<Integer> it = map.values().iterator();
			while (it.hasNext()) {
				answer *= it.next().intValue() + 1;
			}

			return answer-1;
		}

	}

}
```
해쉬를 사용하면 넣을때부터 정리되서 넣기 때문에 후연산이 편리하다. 정리가 우선적으로 필요한 경우 사용가능할거같다.  
해쉬를 만들고나니 나중에 순회할때 어찌하나 보니깐 `Iterator<Integer> it = map.values().iterator();`새로운 자료형에 넣어야한다.  
정답은 공식이 필요했는데 `(옷종류1+1) x (옷종류2의개수 +1) X ... -1`이런식 이였다. 먼가 공식이 필요할거같은데 라고 생각은 했는데 스스로 생각하기 어려웠다.  


---

## 20240319  
### 포인트컷 분리  

`@Aspect`사용할때 포인트컷을 분리해서 사용할 수 있다.

```java
@Slf4j
@Aspect
public class AspectV2 {

 //hello.aop.order 패키지와 하위 패키지
 ("execution(* hello.aop.order..*(..))") //pointcut expression
 private void allOrder(){} //pointcut signature

 @Around("allOrder()")
 public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
 log.info("[log] {}", joinPoint.getSignature());
 return joinPoint.proceed();
 }
}
```
`@Pointcut`이런식으로 여기에다만 경로를 지정하고 함수명을 ` @Around("allOrder()")`사용할곳에 가져다 쓰면된다.  
반환타입은 void  
코드내용비우기  
public, private둘다가능  

이런 조건들로 경로에 대해서 함수명으로 이름을 지어줄 수 있고  
접근자 설정으로 이런 경로만 모아두고 다른곳에는 어드바이저만 모아둬서 여러개 꺼내서 쓸수도있다 모듈화  


---

## 20240320  
### 프로그래머스 기능개발 스택/큐 2단계

프로그래머스 팀에서는 기능 개선 작업을 수행 중입니다. 각 기능은 진도가 100%일 때 서비스에 반영할 수 있습니다.  또, 각 기능의 개발속도는 모두 다르기 때문에 뒤에 있는 기능이 앞에 있는 기능보다 먼저 개발될 수 있고, 이때 뒤에 있는 기능은 앞에 있는 기능이 배포될 때 함께 배포됩니다.  먼저 배포되어야 하는 순서대로 작업의 진도가 적힌 정수 배열 progresses와 각 작업의 개발 속도가 적힌 정수 배열 speeds가 주어질 때 각 배포마다 몇 개의 기능이 배포되는지를 return 하도록 solution 함수를 완성하세요.

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        int[] arr1 = {95, 90, 99, 99, 80, 99};
        int[] arr2 = {1, 1, 1, 1, 1, 1};
        Solution solution = new Solution();
        System.out.println("Hello World");

        int[] arr3 = solution.solution(arr1, arr2);
        for (int i : arr3) {
            System.out.println(i);
        }



    }

    static class Solution {
        public static int[] solution(int[] progresses, int[] speeds) {
            Deque<ArrayList<Integer>> stack = new ArrayDeque<>();
            ArrayList<Integer> result = new ArrayList<>();

            for (int i = 0; i < progresses.length; i++) {
                ArrayList<Integer> arrayList = new ArrayList<>();
                arrayList.add(progresses[i] + speeds[i]);
                arrayList.add(speeds[i]);
                stack.addFirst(arrayList);
            }
            while (true) {
                int stackLen1 = stack.size();
                int count = 0;
                for (int i = 0; i < stackLen1; i++) {
                    if (stack.getLast().get(0) >= 100) {
                        count++;
                        stack.removeLast();
                    } else {
                        break;
                    }
                }
                if (count > 0) {
                    result.add(count);
                }
                if (stack.isEmpty()) {
                    break;
                }


                int stackLen2 = stack.size();
                for (int i = 0; i < stackLen2; i++) {
                    ArrayList<Integer> arrayList2 = stack.removeLast();
                    arrayList2.set(0, arrayList2.get(0) + arrayList2.get(1));
                    stack.addFirst(arrayList2);
                }
            }
            int[] answer = result.stream().mapToInt(Integer::intValue).toArray();
            return answer;
        }
    }
}
```
더 효율이 좋다고 본 정답
```
for(int i=0; i<progresses.length; i++){
        //progresses의 길이만큼 for문을 돌면서 , 
        //progresses 각 인덱스 값이 100을 넘기 위한 최소일수 계산 후 queue에 add 메소드로 넣기
            queue.add((int)Math.ceil((100.0-progresses[i])/speeds[i]));        
        }
.
.
.
```
내가풀은거도 정답은 나오는데 보통 이런식으로 첨부터 100%될수있는 날짜를 구해서 집어넣더라.  
좀더 깊은 생각이 필요할거같다. 내 코드 보면서 전혀 직관적이지 않다는 생각은 들었다.  

---

## 20240321
### 프로그래머스 H-index 정렬 2단계

배운것  
* `ArrayList` 역순 정렬법 `Collections.sort(arrayList, Collections.reverseOrder());`

H-Index는 과학자의 생산성과 영향력을 나타내는 지표입니다. 어느 과학자의 H-Index를 나타내는 값인 h를 구하려고 합니다. 위키백과1에 따르면, H-Index는 다음과 같이 구합니다.  어떤 과학자가 발표한 논문 n편 중, h번 이상 인용된 논문이 h편 이상이고 나머지 논문이 h번 이하 인용되었다면 h의 최댓값이 이 과학자의 H-Index입니다.  어떤 과학자가 발표한 논문의 인용 횟수를 담은 배열 citations가 매개변수로 주어질 때, 이 과학자의 H-Index를 return 하도록 solution 함수를 작성해주세요.  
인풋 {3, 0, 6, 1, 5} 리턴 3  

내가푼것
```java
import java.util.*;

public class Main {
	public static int[] arr1 = {3, 0, 6, 1, 5};
	
	public static void main(String[] args) {
		Solution solution = new Solution();
		System.out.println(solution.solution(arr1));
	}

	public static class Solution {
		public int solution(int[] citations) {
			ArrayList<Integer> arrayList = new ArrayList<>();
			for (int citation : citations) {
				arrayList.add(citation);
			}
			Collections.sort(arrayList, Collections.reverseOrder());
			for (int i = 0; i < arrayList.size(); i++) {
				if (arrayList.get(i) < i + 1) {
					return i;
				}
			}
			return arrayList.size();
		}
	}
}
```
H-index가 뭔지 그냥 글을 이해하는데 오래걸렸다 결국 그냥 구글링해서 뭔지 한글로 다시 봤다.  
더 깔끔한 정답 코드보니 처음부터 return값으로 최대값 넣어두고 돌리면 좀더 간편할거같다. 그리고 `ArrayList`까지 쓸 필요는 없는 문제 이긴 했다.  

---

## 20240322  
### 스프링 AOP구현  
#### 어드바이스 추가  

```java
@Around("allOrder()")
 public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
 log.info("[log] {}", joinPoint.getSignature());
 return joinPoint.proceed();
 }
 //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
 @Around("allOrder() && allService()")
 public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable
{
 try {
 log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
 Object result = joinPoint.proceed();
 log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
 return result;
 } catch (Exception e) {
 log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
 throw e;
 } finally {
 log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
 }
 }
```
이런식으로 그냥 `@Around`에 포인트컷 같은거 하나더 추가해서 넣어주면 작동이됩니다.  
다만 순서는 보장이 안된다 해결법은 아래쪽에서 다시 확인해 보겠습니다.  

#### 포인트컷 참조  

포인트컷 들을 따로 클레스를 빼서 사용하는 방법이 있습니다.  

```java
public class Pointcuts {
 //hello.springaop.app 패키지와 하위 패키지
 @Pointcut("execution(* hello.aop.order..*(..))")
 public void allOrder(){}
```
이런식으로 일단 따로 클레스 만들어주고(함수는 퍼블릭으로 열어줘야합니다.)  

```java
@Around("hello.aop.order.aop.Pointcuts.allOrder()")
 public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
 log.info("[log] {}", joinPoint.getSignature());
 return joinPoint.proceed();
 }
```
실제사용 `@Around`쪽에서는 조금 귀찮지만 페키지명까지 다 복사해서 넣어주면 작동합니다.

#### 어드바이스 순서  

순서를 설정하는데 기본적으로 `@Order(2)`를 사용합니다. 다만 이순서의 기준이 `@Aspect`를 달고 있는 `클레스` 기반이란것이 문제입니다. 그래서 클레스를 분리해줘야합니다.  

```java
@Slf4j
public class AspectV5Order {
 @Aspect
 @Order(2)
 public static class LogAspect {
 @Around("hello.aop.order.aop.Pointcuts.allOrder()")
 public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
 log.info("[log] {}", joinPoint.getSignature());
 return joinPoint.proceed();
 }
 }
 @Aspect
 @Order(1)
 public static class TxAspect {
 @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
 public Object doTransaction(ProceedingJoinPoint joinPoint) throws
Throwable {
 try {
.
.
.
```
지금 예시로는 이너클래스로 만들었는데 `public static class LogAspect` 아무튼 클레스를 분리해서 ` @Aspect`를 기준으로 ` @Order(2)`를 붙여주면 순서데로 작동합니다.  

#### 어드바이스 종류  

항상 `@Around`만 써왔는데 여러가지 종류가 있습니다.  

`@Before` : 조인 포인트 실행 이전에 실행  
`@AfterReturning` : 조인 포인트가 정상 완료후 실행  
`@AfterThrowing` : 메서드가 예외를 던지는 경우 실행  
`@After` : 조인 포인트가 정상 또는 예외에 관계없이 실행(finally)  

기본적으로는 아래의 모든기능을 `@Around`가 포함하고는 있습니다.

```java
@Around("hello.aop.order.aop.Pointcuts.orderAndService()")
 public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable
{
 try {

 //@Before
 log.info("[around][트랜잭션 시작] {}", joinPoint.getSignature());

 Object result = joinPoint.proceed();

 //@AfterReturning
 log.info("[around][트랜잭션 커밋] {}", joinPoint.getSignature());

 return result;

 } catch (Exception e) {

 //@AfterThrowing
 log.info("[around][트랜잭션 롤백] {}", joinPoint.getSignature());

 throw e;
 } finally {

 //@After
 log.info("[around][리소스 릴리즈] {}", joinPoint.getSignature());

 }
 }
```

하위의 4개들은 `@Around`의 특정 부분만 담당합니다,

```java
@Before("hello.aop.order.aop.Pointcuts.orderAndService()")
 public void doBefore(JoinPoint joinPoint) {
 log.info("[before] {}", joinPoint.getSignature());
 }
```
`@Before`는 이후의 ` Object result = joinPoint.proceed();`를 자동으로 알아서 실행시켜줍니다.

```java
@AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", 
returning = "result")
 public void doReturn(JoinPoint joinPoint, Object result) {
 log.info("[return] {} return={}", joinPoint.getSignature(), result);
 }
```
`@AfterReturning`는 `return`으로 돌아와서 다시 `return`하기전의 사이만 담당합니다. 자동으로 기존의 넘어온 객체를 리턴해줍니다. 객체자체에 `setter`같은게 없으면 조작이 어렵습니다.  

`@AfterThrowing`도 예외가 발생했을때 다시 `throw e;`하기전 까지 코드이고  

`@After` 도 `finally`코드만 담당 합니다.  

순서는 같은  ` @Aspect` 안에서는  `@Around , @Before , @After , @AfterReturning , @AfterThrowing`를 보장하고 동일한 어노테이션끼리는 순서를 보장하지 않습니다.  
더불어 사용하는 매개변수가 조금 다릅니다.  

이렇게 `@Around`의 쪼개진 버전이 있는 이유는 `@Around`에서 실수로 `Object result = joinPoint.proceed();`를 빼먹으면 엄청난 오류가 생길 수 있지만 파악하기 어렵고  
다른 사람들이 코드를 이해할때 어려운 부분도 많으며 스스로 내가 사용범위까지만 제약하며 사용하면 이해도 쉽고 오류날 확률도 줄어듭니다.  

---

## 20240323  
### 프로그래머스 모의고사 완전탐색 1단계  

배운것
* `Math`나 `first[i%5]`이런거 익숙해지기

수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.  1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ... 2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ... 3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...  1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.

{1,3,2,4,2} 리턴 1,2,3

내풀이
```java
import java.util.*;

public class Main {
	public static int[] arr1 = {1,3,2,4,2};
	public static int[] arr2 = {1,2,3,4,5};

	public static void main(String[] args) {
		Solution solution = new Solution();
		int[] arr3 = solution.solution(arr2);
		for (int i : arr3) {
			System.out.println(i);
		}
	}

	public static class Solution {
		public int[] solution(int[] answers) {
			int[] array1 = {1, 2, 3, 4, 5};
			int[] array2 = {2, 1, 2, 3, 2, 4, 2, 5};
			int[] array3 = {3, 3, 1, 1, 2, 2, 4, 4, 5, 5};
			Deque<Integer> deque1 = new ArrayDeque<>();
			Deque<Integer> deque2 = new ArrayDeque<>();
			Deque<Integer> deque3 = new ArrayDeque<>();
			for (int i : array1) {
				deque1.addFirst(i);
			}
			for (int i : array2) {
				deque2.addFirst(i);
			}
			for (int i : array3) {
				deque3.addFirst(i);
			}
			HashMap<Integer, Integer> hashMap = new HashMap<>();
			hashMap.put(1,0);
			hashMap.put(2,0);
			hashMap.put(3,0);
			for (int i = 0; i < answers.length; i++) {
				if (answers[i] == deque1.getLast()) {
					hashMap.put(1,hashMap.get(1)+1);
				}
				if (answers[i] == deque2.getLast()) {
					hashMap.put(2,hashMap.get(2)+1);
				}
				if (answers[i] == deque3.getLast()) {
					hashMap.put(3,hashMap.get(3)+1);
				}
				deque1.addFirst(deque1.removeLast());
				deque2.addFirst(deque2.removeLast());
				deque3.addFirst(deque3.removeLast());
			}

			int maxValue = Collections.max(hashMap.values());

			// 최대값을 가진 키들을 추출
			ArrayList<Integer> maxKeys = new ArrayList<>();
			for (Map.Entry<Integer, Integer> entry : hashMap.entrySet()) {
				if (entry.getValue() == maxValue) {
					maxKeys.add(entry.getKey());
				}
			}
			Collections.sort(maxKeys);
			int[] maxKeysArray = new int[maxKeys.size()];
			for (int i = 0; i < maxKeys.size(); i++) {
				maxKeysArray[i] = maxKeys.get(i);
			}
			return maxKeysArray;
		}
	}
}
```
쉬운방법이 있긴할거같은데 그냥 논리적으로 내가 이해하기 편하게 데큐랑 해쉬를 사용했다. 그랬더니 코드가 좀 복잡해졌다.  
간단한 풀이 보니 점수계산은 `if(answers[i] == first[i%5]) score[0]++;`이런식으로 나머지로 처리했고  
최대값을 `int max = Math.max(score[0], Math.max(score[1], score[2]));`그냥 값만 구한후에 `for`문으로 1부터 찾아서 같은거 넣으면  
자동으로 내림차순 완성  

쉬운문제이긴 하지만 `Math`나 `first[i%5]`이런거 좀 익숙해져야한다.

---

## 20240324  
### 스프링AOP 포인트컷지시자 execution
포인트컷을 편리하게 표현하기위해 표현식을 사용하는데 `@Pointcut("execution(* hello.aop.order..*(..))")`보통 이와같다.  
이것의 종류는 여러가지가 있는데 그중 가장 많이 사용하는 `execution`부터 알아보자  
`public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)`정확히 이런녀석을 가르켜보자  
접근제어자?: public  
반환타입: String  
선언타입?: hello.aop.member.MemberServiceImpl  
메서드이름: hello  
파라미터: (String)  
예외?: 생략  

이중 ? 는 생략 가능한것  

가장많이 생략한것
`pointcut.setExpression("execution(* *(..))");`  
접근제어자?: 생략  
반환타입: *  
선언타입?: 생략  
메서드이름: *  
파라미터: (..)  
예외?: 없음  

*은 아무값이 들어와도 된다는뜻이다.  
파라미터에서 .. 은 파라미터의 타입과 파라미터 수가 상관없다는 뜻이다.

메서드 이름 관련
```
pointcut.setExpression("execution(* hello(..))");
pointcut.setExpression("execution(* hel*(..))");
pointcut.setExpression("execution(* *el*(..))");
```
이런식으로 단어 앞이든 뒤든 서치가능  

페키지관련 
```
//전체
pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.hello(..))")

//*활용
pointcut.setExpression("execution(* hello.aop.member.*.*(..))")

// ..은 하위의 모든것 (그래서.이 하나면 정확히 그위치 라서 오류남)
pointcut.setExpression("execution(* hello.aop..*.*(..))");
```
. : 정확하게 해당 위치의 패키지  
.. : 해당 위치의 패키지와 그 하위 패키지도 포함  


타입매칭 부모타입
```
pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
```
`MemberService`는 우리가 찾을 녀석의 인터페이스다. 중요한건 인터페이스에 `존재하고` 자식녀석에게 구현되어있으면 자식을 찾을 수 있다.  
다만 실제객체에서 내가 새로 구현한것은 찾을 수 없다.  

파라미터
```
//정확히 하나의 파라미터 허용, 모든 타입 허용
pointcut.setExpression("execution(* *(*))");

//숫자와 무관하게 모든 파라미터, 모든 타입 허용
pointcut.setExpression("execution(* *(..))");

//String 타입으로 시작, 숫자와 무관하게 모든 파라미터, 모든 타입 허용
pointcut.setExpression("execution(* *(String, ..))");
```
(String) : 정확하게 String 타입 파라미터  
() : 파라미터가 없어야 한다.  
(*) : 정확히 하나의 파라미터, 단 모든 타입을 허용한다.  
(*, *) : 정확히 두 개의 파라미터, 단 모든 타입을 허용한다.  
(..) : 숫자와 무관하게 모든 파라미터, 모든 타입을 허용한다. 참고로 파라미터가 없어도 된다. 0..* 로 이해하 면 된다.  
(String, ..) : String 타입으로 시작해야 한다. 숫자와 무관하게 모든 파라미터, 모든 타입을 허용한다.  
예) (String) , (String, Xxx) , (String, Xxx, Xxx) 허용  

생각보다 말장난같고 헷갈리는것이 많아 유의하자.  

---

## 20240325  
### 프로그래머스 소수찾기 완전탐색(DFS) 2단계

* 순열DFS
* 소수찾기

한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.  각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.  

제한사항
* numbers는 길이 1 이상 7 이하인 문자열입니다.
* numbers는 0~9까지 숫자만으로 이루어져 있습니다.
* "013"은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.

먼가 DFS같이 조합을해야하며 에라토스테네스의 체를 이용해야겠다는 생각은 들었는데 그냥 구현하기가 너무 어려웠다.  

순열의 경우 (https://bcp0109.tistory.com/14) 해당 블로그를 참고하여 공부했다.  
더불어 에라토스테네스의 체는 원리를 몰라 검색하여 찾아보았다.  

```
import java.util.*;

public class Main {
	public static String arr1 = "17";
	public static int[] arr2 = {1,2,3,4,5};

	public static void main(String[] args) {
		Solution solution = new Solution();
		int result = solution.solution(arr1);
		System.out.println(result);
	}

	public static class Solution {
		public int solution(String numbers) {
			Set<Integer> set = new HashSet<>();
			int[] numbersInt = new int[numbers.length()];
			for (int i = 0; i < numbers.length(); i++) {
				numbersInt[i] = Character.getNumericValue(numbers.charAt(i));
			}
			ArrayList arrayList = new ArrayList<>();
			boolean[] visited = new boolean[numbers.length()];
			perm(numbersInt, arrayList, visited, 0, numbers.length(), numbers.length(),set);
			int count = 0;
			for (Integer integer : set) {
				if (isPrime(integer)) {
					count++;
				}
			}

			return count;
		}

		public void perm(int[] arr, ArrayList<Integer> arrayList, boolean[] visited, int depth, int n, int r, Set<Integer> set) {
			for (int i=0; i<n; i++) {
				if (visited[i] != true) {
					visited[i] = true;
					arrayList.add(arr[i]);
					int result = combineArrayToNumber(arrayList);
					System.out.println(result + "DFS");
					set.add(result);
					perm(arr, arrayList, visited, depth + 1, n, r, set);
					arrayList.remove(depth);
					visited[i] = false;
				}
			}
		}

		public int combineArrayToNumber(ArrayList<Integer> arrayList) {
			int result = 0;
			for (Integer integer : arrayList) {
				result = result * 10 + integer;
			}
			return result;
		}

		public boolean isPrime(int n){
			if(n<2) return false;

			for(int i=2; i<=(int) Math.sqrt(n); i++) {
				if(n%i == 0) {
					return false;
				}
			}

			return true;
		}
	}
}
```

---

## 20240326  
### 스프링 포인트컷 그외 지시자1

#### within
`execution`의 기능에서 타입부분만 사용하는 느낌이다.  

```java

//MemberServiceImpl 적용
pointcut.setExpression("within(hello.aop.member.MemberServiceImpl)");

//MemberServiceImpl의 부모타임 적으면 적용안됨!
pointcut.setExpression("within(hello.aop.member.MemberService)"); = False

pointcut.setExpression("within(hello.aop.member.*Service*)");
```

하나 주의할점은 `execution`과는 다르게 부모타입을 적는다고 하위까지 인정안해준다.  


#### args
`execution`의 기능에서 `args`만 사용하는 느낌이다.  

```java
pointcut("args(String)")

pointcut("args(Object)")

pointcut("args()")

pointcut("args(String,..)")

//스트링 타입 매개변수
pointcut("args(java.io.Serializable)") = true
pointcut("args(Object)") = true

pointcut("execution(* *(java.io.Serializable))") = false
(pointcut("execution(* *(Object))") = false
```

여기서도 주의할점이 있는데 `execution`은 정직으로 판단(코드에 박힌 글자 그데로판단) = 완벽하게 타입이 일치해야함 (부모안됨)  
`args` 실제 런타임에서 객체넘어온거로 확인 (동적) = 부모 클레스도 받아줄 수 있다.  

#### @target, @within  
둘다 해당 클레스에 어떤 어노테이션이 붙어있는지로 확인  

```
@Around("execution(* hello.aop..*(..)) && @target(hello.aop.member.annotation.ClassAop)")
@Around("execution(* hello.aop..*(..)) && @within(hello.aop.member.annotation.ClassAop)")
```

주의할 점 2가지  
1. target은 자식 클래스타입에서 부모에 존재하는(자식클레스에서 오버라이딩 등 안하고 부모꺼 바로쓰기) 함수 사용해도 적용 within은 오버라이딩 등 실제 지정된 클래스에 함수가 안보이면 적용 안함
2. `args, @args, @target`이 3가지는 런타임환경에서 실제 스프링 컨테이너가 만들어지는 시점에 적용이 가능하기에 `execution(* hello.aop..*(..)) &&` 앞쪽에서 이렇게 범위를 한번 지정해주지 않으면 스프링에 존재 하는 모든 `bean`에 프록시를 한번 만들어보고 확인하고 하는 작업을 하게 된다. 문제는 그와중에 `final`로 지정된 `bean`들도 있기때문에 오류가 날 수 있다.

#### @annotation, @args
`@annotation` 메소드에 붙은 어노테이션으로 적용하거나 `@args` 함수의 매개변수 타입속에 존재하는 어노테이션으로 적용  

```
@MethodAop 란게 붙어있는 함수적용
@Around("@annotation(hello.aop.member.annotation.MethodAop)")

@Check 란게 함수의 매개변수 타입에 붙어있으면 적용
@args(test.Check)
```

#### bean 

bean : 스프링 전용 포인트컷 지시자, 빈의 이름으로 지정한다.
스프링 빈의 이름으로 AOP 적용 여부를 지정한다. 이것은 스프링에서만 사용할 수 있는 특별한 지시자이다.

```
스프링 컨테이너의 orderService아름, Repository로 끝나는 빈에 적용
@Around("bean(orderService) || bean(*Repository)")
```

---

## 20240327
### 쿼리 실행 계획

> 
>Q : 데이터베이스의 쿼리 최적화를 어떻게 하면 좋을까요?
>  

1. 최적화를 위해서는 쿼리를 실행하는 방식과 해당 쿼리의 성능을 알고 있어야 합니다.

2. 그러기 위해서는 데이터베이스의 쿼리 실행 계획을 보고 분석할 수 있어야 합니다.

> 
>Q : 쿼리 실행 계획이 무엇인가요?
>  


1.  데이터베이스 관리 시스템(DBMS)이 SQL 쿼리를 처리하기 위해 사용하는 실행 계획입니다. 쿼리 실행에 필요한 단계를 보여주며, 각 단계에서 DBMS가 사용하는 액세스 경로를 보여주고, 쿼리 실행에 필요한 리소스 및 비용 정보를 제공합니다.

* 특징
	* 쿼리 실행에 필요한 단계를 보여줍니다.
 	* 각 단계에서 DBMS가 사용하는 엑세스 경로를 보여줍니다.
	* 쿼리 실행에 필요한 리소스 및 비용 정보를 제공합니다.

 * 장점
	* 성능 문제를 식별하는데 도움이 됩니다.
	* 실행 계획을 변경하여 쿼리 성능을 개선할 수 있습니다.
 	* 쿼리 최적화를 위한 정보를 제공합니다.

(실행 계획 순서 읽기 자료 스크랩)  




---

#### 실행 계획 순서 읽기 자료 스크랩
(출처:https://harris91.vercel.app/query-plan)


실행 계획은 여러 가지 단계로 이루어져 있는데 이것을 스텝이라고 한다. 각각의 스텝에는 그 단계에서 어떤 명령이 수행되었고 총 몇 건의 데이터가 처리되었으며 이 처리를 위해 얼마만큼의 비용과 시간이 소요되었다.
실행 계획 순서 읽기
![img](https://github.com/299unknown/diary/assets/151738362/826baf02-c97e-4bc9-8335-44be8fedbd28)

실행 계획 예시


실행 계획을 읽을 때에는 아래와 같은 규칙이 있다. 이 규칙을 토대로 하나씩 읽어나간다.

위에서 아래로 읽어 내려가면서 제일 먼저 읽을 스텝을 찾는다.

내려가는 과정에서 같은 들여 쓰기가 존재한다면 무조건 위 ➡ 아래 순으로 읽는다.

읽고자 하는 스텝보다 들여 쓰기가 된 하위스텝이 존재한다면,  
가장 안쪽으로 들여쓰기 된 스텝을 시작으로 하여 한 단계씩 상위 스텝으로 읽어 나온다.  

![img](https://github.com/299unknown/diary/assets/151738362/dd28caa8-6671-4aa6-ac28-786838a5e706)

위의 예제의 경우 이 규칙으로 실행 계획을 읽는 순서를 정한다면 위와 같이 된다.  
출력된 실행 계획에서 위쪽에 출력된 결과일수록(ID 칼럼의 값이 작을수록) 쿼리의 바깥(Outer) 부분이거나 먼저 접근한 테이블이고, 아래쪽에 출력된 결과일수록(ID 칼럼의 값이 클수록) 쿼리의 안쪽(Inner) 부분 또는 나중에 접근한 테이블에 해당된다.  


실행 계획 해석하기  
실행 계획의 해석 가장 나중에 실행된 것부터 즉 트리의 가장 좌측 아래부터 역순으로 해석한다.  

 ![img](https://github.com/299unknown/diary/assets/151738362/f4dbe260-c8c4-4cce-a7ef-36aa25fee3a6)

위의 예제를 기준으로 한다면 위와 같은 순서로 해석해간다.  
자식들의 좌측부터 차례대로 읽어주고 그 다음에 상위 부모로 올라가는 식으로 반복하면 된다.  
위의 예제는 5 ➡ 4 ➡ 6 ➡ 3 ➡ 7 ➡ 2 ➡ 8 ➡ 1 ➡ 0 순으로 진행  
 ![img](https://github.com/299unknown/diary/assets/151738362/e31914df-1003-4e2a-82bc-45d151f1fb7c)

위의 실행 계획을 해석하자면 위의 그림과 같다.
5번 : PK_EMP 인덱스를 사용하여 INDEX RANGE SCAN을 하면서 조건에 만족하는 인덱스 블록과 키 값을 검색한 결과를 반환한다.  
4번 : 5번에서 읽은 ROWID를 기반으로 EMP 테이블로 이동하여 조건에 부합하는 결과를 반환한다.  
6번 : PK_DEPTNO 인덱스에서 INDEX UNIQUE SCAN 방식으로 검색한 결과의 ROWID를 반환한다.  
3번 : 4번과 6번에서 반환된 데이터들을 기준으로 NESTED LOOP JOIN 방식으로 4번에서 반환된 데이터의 숫자만큼 반복하여 조인한 결과를 반환한다.  
7번 : DEPT 테이블도 4번과 같이 조건에 부합하는 결과를 반환한다.  
2번 : NESTED LOOP JOIN 방식으로 3번과 같이 JOIN의 결과를 만들어준다.  
8번 : SALGRADE 서브쿼리를 실행한다.  
1번 : 서브쿼리를 통해 해당 조건을 만족하는 데이터를 필터링하여 반환한다.  


---

##### 쿼리 최적화 확인하기

DB의 최적화는 사용자의 만족도와 프로젝트의 비용 감소를 위해서 반드시 필요합니다. 이를 실현 하기위한 가장 중요한 기술인 실행 계획 확인 하는 법을 알아보겠습니다.  

#### 쿼리 최적화 절차  
1. 원하는 결과를 조회해 보면서 Fetch time(가져온 결과를 전송하는데 걸리는 시간 쿼리와 무관), Duration time(쿼리를 실행하느 시간)을 확인합니다.
2. 문제가 되는 쿼리를 확인후 실행 계획을 확인하며 조건절, 조인, 서브쿼리 구조, 정렬, 인덱스 현황을 파악합니다.
3. 파악한 정보들을 기반으로 개선합니다.

#### 문제가 되는 쿼리확인
DB에서 문제가 되는 쿼리들의 목록이나 정보를 확인할 수 있을만한 명령어들이 있어서 가져와 봤습니다.

```sql
## 프로세스 목록
SHOW PROCESSLIST;

## 슬로우 쿼리 확인
SELECT query, exec_count, sys.format_time(avg_latency) AS "avg latency", rows_sent_avg, rows_examined_avg, last_seen
FROM sys.x$statement_analysis
ORDER BY avg_latency DESC;

## 성능 개선 대상 식별
SELECT DIGEST_TEXT AS query,
             IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
             COUNT_STAR AS exec_count,
             SUM_ERRORS AS err_count,
             SUM_WARNINGS AS warn_count,
             SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS exec_time_total,
             SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS exec_time_max,
             SEC_TO_TIME(AVG_TIMER_WAIT/1000000000000) AS exec_time_avg_ms, SUM_ROWS_SENT AS rows_sent,
             ROUND(SUM_ROWS_SENT / COUNT_STAR) AS rows_sent_avg, SUM_ROWS_EXAMINED AS rows_scanned,
             DIGEST AS digest
   FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC

##  I/O 요청이 많은 테이블 목록
SELECT * FROM sys.io_global_by_file_by_bytes WHERE file LIKE '%ibd';

## 테이블별 작업량 통계
SELECT table_schema, table_name, rows_fetched, rows_inserted, rows_updated, rows_deleted, io_read, io_write
FROM sys.schema_table_statistics
WHERE table_schema NOT IN ('mysql', 'performance_schema', 'sys');

## 최근 실행된 쿼리 이력 기능 활성화
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history'
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history_long'

## 최근 실행된 쿼리 이력 확인
SELECT * FROM performance_schema.events_statements_history
```

#### 실행 계획

* 실행 계획은 쿼리 옵티마이저가 데이터를 조회하기 위한 계획을 의미합니다.
* 옵티마이저란 개발자가 DB로 조회요청을 하게되면 DB가 쿼리를 분석하고(where인지 join인지 등 ), 인덱스 통계정보를 사용하면 인덱스를 경정하고, 여러 테이블이 엮여 있을경우 어떤 순서로 테이블을 읽을지 결정합니다.
![image](https://github.com/299unknown/diary/assets/151738362/08294880-3814-4a16-8374-45cb18b75705)

1. Query Cache  
SQL문이 key, 결과가 value인 맵입니다. 데이터가 변경되었으면 쿼리캐시가 삭제되어야겠죠?(조회 결과가 달라질 것이기 때문에) 이는 동시 처리 성능 저하를 유발하고, 버그의 원인이 되어 MySQL 8.0 버전부터는 삭제되었습니다.  
2. Parsing  
사용자가 요청한 SQL을 잘게 쪼개어 서버가 이해할 수 있는 수준으로 분리합니다.  
3. Preprocessing  
해당 쿼리가 문법적으로 틀린지 확인하여 부정확하면 처리를 중단합니다. (흔히 만나보는 syntax 에러는 parser와 preprocessor에서 발생합니다.)  
4. Query Optimization  
실행계획은 이 단계에서의 출력을 의미합니다.  
쿼리 분석 : where절의 검색 조건인지, join 조건인지 판단합니다.  
인덱스 선택 : 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스를 결정합니다.  
조인 처리 : 여러 테이블의 조인이 있는 경우, 어떤 순서로 테이블을 읽을지 결정합니다.  
5. Handler (Storage Engine)  
MySQL Execution engine의 요청에 따라 데이터를 디스크로 저장하고, 디스크로부터 읽어오는 역할을 합니다. 대표적인 스토리지 엔진은 InnoDB, MyISAM 이 있습니다. MySQL 엔진에서는 스토리지 엔진으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행합니다.  

#### 실행계획 확인 (Mysql)  

```sql
EXPLAIN
SELECT 사원.사원번호, 급여.연봉
	FROM 사원,
		(SELECT 사원번호, MAX(연봉) as 연봉
		FROM 급여
		WHERE 사원번호 BETWEEN 10001 AND 20000 GROUP BY 사원번호) as 급여
WHERE 사원.사원번호 = 급여.사원번호;
```
<img width="640" alt="1" src="https://github.com/299unknown/diary/assets/151738362/6c9efe6b-0ef5-46e4-b40a-809ce26da350">

기본적으로 사용하던 쿼리에 EXPLAIN을 붙여주면 실행계획정보를 반환해 줍니다.  

* id : SELECT붙은 번호를 말합니다. 만약 서브쿼리가 생기면 숫자가 증가합니다. 다만 join은 하나의 단위로 인식하기에 같은숫자가 나옵니다. 숫자는 실행순서를 의미하지 않습니다.


* select_type : SELECT의 유형을 뜻합니다.
	* SIMPLE : 단순한 SELECT 문
	* PRIMARY : 서브쿼리를 감싸는 외부 쿼리, UNION이 포함될 경우 첫번째 SELECT 문
	* SUBQUERY : 독립적으로 수행되는 서브쿼리(SELECT, WHERE 절에 추가된 서브쿼리)
	* DERIVED : FROM 절에 작성된 서브쿼리UNION : UNION, UNION ALL로 합쳐진 SELECT 문
	* DEPENDENT SUBQUERY : 서브쿼리가 바깥쪽 SELECT 쿼리에 정의된 칼럼을 사용 하는 경우
	* DEPENDENT UNION : 외부에 정의된 칼럼을 UNION으로 결합된 쿼리에서 사용하는 경우
	* MATERIALZED : IN 절 구문의 서브쿼리를 임시 테이블로 생성한 뒤 조인을 수행UNCACHEABLE SUBQUERY : RAND(), UUID() 등 조회마다 결과가 달라지는 경우

* table : 어떤 테이블에 접근하는지 

* partitions : 사전에 정의한 파티션이 있는경우 선택적 접근 표시

* type : 데이블의 데이터를 어떻게 찾을지 관한 정보  
	* system : 테이블에 데이터가 없거나 한 개만 있는 경우
	* const : 조회되는 데이터가 단 1건일 때 (where에 Unique Key사용 등으로 딱 1건 리턴)
	* eq_ref : 조인이 수행될 때 드리븐 테이블의 데이터에 PK 혹은 고유 인덱스로 단 1건의 데이터를 조회할 때
	* ref : eq_ref와 같으나 데이터가 2건 이상일 경우 (join할때 Unique Key가 아닌것으로 매칭할때)
	* index : 인덱스 풀 스캔 (인덱스를 처음부터 끝까지)
	* range : 인덱스 레인지 스캔 (특정범위)
	* all : 테이블 풀 스캔 (처음부터 끝까지)

 * possible_key : 옵티마이저가 SQL문을 최적화하고자 사용할 수 있는 인덱스 목록을 출력 (후보군)

 * key : possible_key중 실제로 사용한 인덱스

 * ket_len : 사용한 인덱스의 바이트(bytes) 수를 의미  

 * ref : 키 칼럼에 나와 있는 인덱스에서 값을 찾기 위해 선행 테이블의 어떤 칼럼이 사용 되었는지

 * rows : SQL문을 수행하기 위해 접근하는 데이터의 모든 행 수(통계적 대략적인값)

 * filtered : filtered는 행 데이터를 가져와 WHERE 구의 검색 조건이 적용되면 몇 행이 남는 지를 표시(정확한값)

 * Extra : 옵티마이저가 동작한 것에대한 대략적 힌트
	* Distinct : 중복 제거시
	* Using where : WHERE 절로 필터시
	* Using index : 물리적인 데이터 파일을 읽지 않고 인덱스만 읽어서 처리, 커버링 인덱스
	* Using temporary : 데이터의 중간 결과를 저장하고자 임시 테이블을 생성, 보통 DISTINCT, GROUP BY, ORDER BY 구문이 포함된 경우 임시 테이블을 생성
	* Using Filesort : 정렬 시
		* 일반적으로 데이터가 많은 경우 Using Filesort 와 Using Temporary 상태는 좋지 않으며 MySQL 쿼리 튜닝 후 성능 최적화를 위한 모니터링이 필요하다.


#### 실행 계획 개선 방향
![image](https://github.com/299unknown/diary/assets/151738362/d8771eac-7bc3-44f5-95c8-417cd28c567b)

* select_type : dependent type은 조회시마다, 외부 테이블에 access하게 되므로 성능에 악역향을 미칩니다. Rand함수들을 활용하면 uncacheable이 나오는데, 이 또한 마찬가지입니다.
* type : 인덱스 레인지 풀 스캔, 혹은 테이블 풀 스캔을 줄일 수 있는 방향으로 개선해야 합니다.
* extra : filesort나 group by를 위한 temp 테이블 생성보다 인덱스를 활용하여 sorting/group by를 수행할 수 있다면 성능을 개선할 수 있습니다.

#### 실제 실행된 소요시간, 비용 측정하여 분석하기  

* 일반적으로 그냥 EXPLAIN 키워드는 실제 SQL문이 실행된 뒤 나온 계획이 아니라 MySQL 서버가 가지고 있는 통계정보들을 활용한 예측된 결과입니다
* ANALYZE 키워드를 사용해야 올바른 결과가 나올 확률이 높습니다.

```sql
EXPLAIN ANALYZE
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 1 and 10000000
+------------+
| EXPLAIN    |
+-----------------------------------------------------------------------------------------------+
| -> Filter: (`사원`.`사원번호` between 1 and 10000000)  (cost=30099 rows=149645) (actual time=0.291..127 rows=300024 loops=1)
    -> Index range scan on 사원 using PRIMARY over (1 <= 사원번호 <= 10000000)  (cost=30099 rows=149645) (actual time=0.237..104 rows=300024 loops=1)
             |
+---------------------------------------------------------------------------------------------+
```

#### 실제 개선 예시  

1. 문제 파악
	* 1000만개의 데이터가 있는 DB에 포인트 등수 같은 순위 서비스를 제공하려 합니다.
	* `SELECT nickname, point FROM member ORDER BY point DESC;`를 이용해 봅니다.
	* 시간이 5,6초나 걸립니다.

위와 같은 상황에서는 1000만개의 데이터를 `ORDER BY point DESC;`로 정렬 해야하니 당연히 오래걸릴거 같습니다.

2. 일반적인 시도
	* 페이징을 한번 적용해 봅니다.
	* `SELECT id, nickname, point FROM member ORDER BY point DESC LIMIT 100 OFFSET 1000;`를 사용해서 1000번째부터 100개만 조회해 봅니다.
	*  여전히 3초나 걸립니다.

3. 실행계획을 분석해 봅니다.

`SELECT id, nickname, point FROM member ORDER BY point DESC LIMIT 100 OFFSET 1000;` 분석

```sql
-> Limit/Offset: 100/1000 row(s)  (cost=998739 rows=100) (actual time=1954..1954 rows=100 loops=1)
    -> Sort: `member`.`point` DESC, limit input to 1100 row(s) per chunk  (cost=998739 rows=9.74e+6) (actual time=1954..1954 rows=1100 loops=1)
        -> Table scan on member  (cost=998739 rows=9.74e+6) (actual time=0.347..1432 rows=10e+6 loops=1)
```
`-> Table scan on member (... rows=10e+6) `에서 여전히 1000만개 데이터를 스캔하고 있다는것을 확인 할 수 있습니다.  

이는
	1. 전체 데이터 스캔
 	2. 정렬
	3. 그 후에 100개를 찾습니다.

100개의 데이터가 필요해도 `ORDER BY`때문에 전체를 스캔해야합니다.  
만약 데이터가 항상 point로 정렬되있으면 전체 데이터를 스캔할 필요가 없어집니다.  

4. 인덱스 적용
책의 마지막에 있는 "찾아보기"가 인덱스에 비유된다면 책의 내용은 데이터 파일에 해당한다고 볼 수 있습니다.
다만 장점만 존재하는 것은 아닙니다. 인덱스가 많은 테이블은 INSERT, UPDATE, DELETE 문장의 처리가 느려집니다. 원본 테이블뿐만 아니라 인덱스에도 변경사항을 반영해야하기 때문입니다.

`CREATE INDEX point_desc_index ON member(point DESC);`  
CREATE INDEX를 통해서 point의 인덱스를 만들어 줍니다.  

point 컬럼을 인덱스 키로 설정하였습니다. 따라서 인덱스의 리프노드는 point 컬럼을 기준으로 내림차순 정렬됩니다. 인덱스 자체가 이미 정렬되어 있기 때문에 별도의 정렬 작업 없이 읽기만 하면 됩니다.

인덱스를 적용하고 `SELECT id, nickname, point FROM member ORDER BY point DESC LIMIT 100 OFFSET 1000;`를 똑같이 시도해보면 시간이 확연 하게 단축된 모습을 볼 수 있습니다.  

다만 위 코드에서 `OFFSET`의 숫자를 전체 데이터 수와 같게 만들면 결국 그만큼의 데이터를 한번에 스캔해야해서 다시 오래걸릴 수 있습니다.  
이때는 No Offset 같은 방식을 또 적용 할 수 있습니다.  




#### 참고자료
https://stackoverflow.com/questions/9425134/mysql-duration-and-fetch-time  
https://sihyung92.oopy.io/database/mysql-query-plan  
https://jhlee-developer.tistory.com/entry/MYSQL-%EC%8B%A4%ED%96%89-%EA%B3%84%ED%9A%8D-%EC%88%98%ED%96%89-table-partitions
https://0soo.tistory.com/235
https://cookie-dev.tistory.com/31  

---

## 20240328
### 루시카토 CS 자바 파라미터 전달방식  

#### 인트로

일반적으로 각 언어마다 변수를 넘겨주는 방법은 2가지가 있습니다. Pass By Value, Pass By Reference 인데 이둘의 차이와 자바에서는 어떻게 쓰이는지 알아보겠습니다.  

문제풀기
```java
class Dog {

    private String name;

    public Dog (String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}

public class Test {

    public static void main(String[] args) {
        int x = 10;
        int[] y = {2, 3, 4};
        Dog dog1 = new Dog("강아지1");
        Dog dog2 = new Dog("강아지2");
        
        // 함수 실행
        foo(x, y, dog1, dog2);
        
        // 어떤 결과가 출력될 것 같은지 혹은 값이 어떻게 변할지 예상해보세요!
        System.out.println("x = " + x);
        System.out.println("y = " + y[0]);
        System.out.println("dog1.name = " + dog1.getName());
        System.out.println("dog2.name = " + dog2.getName());
    }

    public static void foo(int x, int[] y, Dog dog1, Dog dog2) {
        x++;
        y[0]++;
        dog1 = new Dog("이름 바뀐 강아지1");
        dog2.setName("이름 바뀐 강아지2");
    }

}
```

출력답
```java
x = 10
y = 3
dog1.name = 강아지1
dog2.name = 이름 바뀐 강아지2
```

#### 메모리할당 

위의 코드에서 메모리 할당이 어떻게 되는지 확인해보며 자바의 매개변수 전달 방식을 알아보겠습니다.  

어떠한 변수를 선언한다는 것은 메모리를 할당한다는 것을 의미합니다. 변수를 선언하기 위해 할당되는 메모리로는 크게 스택과 힙이 있습니다. 스택 영역에는 함수의 호출과 함께 지역 변수 또는 매개변수 등이 할당되며 정렬된 방식으로 메모리가 할당되고 해제됩니다. 반면에 힙 영역에는 클래스 변수(또는 인스턴스 변수) 또는 객체 등이 할당되며, 우연하고 무질서하게 메모리가 할당됩니다.. (그래서 JVM은 무질서하게 관리되는 힙 영역을 위주로 가비지 컬렉터를 통해 메모리의 해제를 관리합니다.)  

* 원시변수
Java에서 변수는 객체가 아닌 실제 값들인 int, double, float boolean 등과 같은 원시 값(Primitive Value)들이 존재합니다.  
```java
public void test() {
    // Primitive Value
    int x = 3;
    float y = 10.012f;
    boolean isTrue = true;
}
```
이와같은 경우 `stack`영역에 실제로 저장됩니다.  

![1](https://github.com/299unknown/diary/assets/151738362/0dd2ac45-c9ed-46af-9eb4-e75409eb3e69)

* 객체의 메모리 할당

```java
public void test() {
    // Primitive Value
    int x = 3;
    float y = 101.012f;
    boolean isTrue = true;
    
    // Object
    String name = "MangKyu";
}
```
![1](https://github.com/299unknown/diary/assets/151738362/ea6a3f8d-2915-4f1c-bf94-3f927b9a3d64)

객체의 경우는 조금 다릅니다. 우선 변수명 `name`으로 `stack`에 메모리가 할당이 된후에 `heap`영역에 실제 값이 저장되고 그 값의 주소가 저장됩니다.  

```java
public void test() {
    // Primitive Value
    int x = 3;
    float y = 101.012f;
    boolean isTrue = true;
    
    // Object
    String name = "MangKyu";
    
    String[] names = new String[3];
    names[0] = "I";
    names[1] = "am";
    names[2] = new String("MangKyu");
    
}
```
![1](https://github.com/299unknown/diary/assets/151738362/1a967f71-d1ae-4120-8c6f-297d14ca24cd)

배열의 경우에도 변수명으로 `stack`에 메모리가 할당된 후에 `heap`에서 참조의 참조를 통해 값들을 관리하게됩니다.  

#### 문제 복기

```java
public static void main(String[] args) {
        int x = 10;
.
.
.
public static void foo(int x, int[] y, Dog dog1, Dog dog2) {
        x++;
```

![1](https://github.com/299unknown/diary/assets/151738362/24868215-d8a8-4c16-83ed-c8128f94ce5e)
원시값의 경우에는 `stack`에서 새로운 x가 생성되며 그값을 실제로 복사해서 `stack`에 가지고 있습니다. 이것으로 함수안에서 x가 변경되어도 함수가 끝나게 되면 `stack`에서 할당된 메모리가 pop 되면서 없어지게 됩니다.  


```java
public static void main(String[] args) {
        int[] y = {2, 3, 4};
.
.
.
public static void foo(int x, int[] y, Dog dog1, Dog dog2) {
        y[0]++;
```
![1](https://github.com/299unknown/diary/assets/151738362/fa5b5f8c-5ac5-4355-b757-36ad6e2524d3)

y의 경우 함수안에서 새롭게 `stack` 메모리가 생성되지만 주소값이 같은 `heap`을 가르키고 있습니다. 그렇기에 함수 안에서 함수안의 y를 수정하면 기존의 y에도 적용되게됩니다.  

**중요**
```java
public static void main(String[] args) {
        Dog dog1 = new Dog("강아지1");
.
.
.
public static void foo(int x, int[] y, Dog dog1, Dog dog2) {
        dog1 = new Dog("이름 바뀐 강아지1");
```
![1](https://github.com/299unknown/diary/assets/151738362/ac831386-d400-44d7-91df-2bbedf43447c)

dog1의 경우 우선적으로 다른 경우와 똑같이 dog1이라는 변수가 `stack`에 새로 생기며 주소값도 기존의 값을 그대로 복사해서 원래 dog1을 가르키고 있습니다.  

![1](https://github.com/299unknown/diary/assets/151738362/c75fc0d5-95c9-42ba-8b63-d9ef13716b90)

하지만 `new Dog`로 새로 생성하게되면 `heap`영역에서 `new Dog("이름 바뀐 강아지1");` 해당하는 메모리가 할당되고 그 주소를 다시 **함수 dog1메모리에** 저잘하게 됩니다.  
그렇기때문에 함수가 종료되면 함수의 dog1은 완전히 소멸되고 기존의 dog1만 남게됩니다.  


```java
public static void main(String[] args) {
        Dog dog2 = new Dog("강아지2");
.
.
.
public static void foo(int x, int[] y, Dog dog1, Dog dog2) {
        dog2.setName("이름 바뀐 강아지2");
```
![1](https://github.com/299unknown/diary/assets/151738362/917e1163-d835-45aa-9e97-7510efff5b9e)
![2](https://github.com/299unknown/diary/assets/151738362/ab4b1c8e-5902-4c1b-a9f9-fc62571cb06a)

dog2의 경우 똑같이 새로생성된 dog2가 `stack`에서 기존의 dog2와 같은 주소를 가지고 있으며 주소값을 통해 Heap 영역에 존재하는 객체에 접근하여 set을 통해 값을 변화시켜주고 있으므로 원래값의 변경이 가능합니다.  


#### 자바의 Pass By Value방식  

![1](https://github.com/299unknown/diary/assets/151738362/b51921b3-ee40-47a5-9371-6666effc7a65)

자바에서는 `Pass By Value`으로 실제값을 전달한다고는 하지만 객체들은 `heap`영역에서 값들을 할당받고 그 주소를 `stack`에서 가지고있기때문에 함수안에서 새로운 객체를 생성하든 뭐든 밖의 `dog1`이라는 변수의 주소값을 변경해줄수는 없기때문에 밖의 `dog1` 변수의 객체를 변경해줄 수는 없습니다.  

객체가 할당될때 `heap`영역에서 값들이 저장되고 그주소 `stack`에 저장하는 방식때문에 `Pass By Reference`방식이라 착각할 수 있는데 잘 공부해두는것이 좋을거같습니다.  

#### 정리
![`](https://github.com/299unknown/diary/assets/151738362/855dd839-de0a-4d41-b13d-56412ca2723b)


#### 참고자료

[Java] Pass By Value와 Pass By Reference의 차이 및 이해 포스트를 한번에 보기좋게 정리해보았습니다.  
https://mangkyu.tistory.com/105  
https://mangkyu.tistory.com/106  
https://mangkyu.tistory.com/107  

---

## 20240329
### 루시카토 CS 자바 GC  

#### 인트로
자바의 사용하지 않는 객체의 메모리를 GC(Garbage Collector)가 주기적으로 검사해서 청소


#### 가비지 컬렉터와 가비지 컬렉션의 차이

* 가비지 컬렉터 : 메모리 관리를 담당하는 시스템 또는 프로그램의 구성 요소이며, 메모리에서 더 이상 사용되지 않는 객체를 찾아 제거하여 메모리를 회수하는 역할을 수행한다.
* 가비지 컬렉션 : 메모리 관리 기술 중 하나로, 가비지 컬렉터에 의해 수행되는 프로세스를 의미.
가비지 컬렉션은 프로세스 자체를 얘기하고 컬렉터는 실제 역할을 수행하는 주체

#### JVM Heap 메모리 영역  


![1](https://github.com/witwint/TIL/assets/108222981/0eaa7b76-9347-43b8-a2f1-383808ffe4d3)

Young Generation(eden, s1, s2) : 새 객체 할당될때 처음 들어오는 위치  
Old Generation(Tenured) : Young Generation에서 오랜시간 살아남응 객체들이 이동되는곳  
Metasapace : Class의 Meta 정보들이 이 영역에 저장, Native Memory 영역에 위치하며, JVM이 아닌 OS 레벨에서 관리  

#### Reachable과 Unreachable  
메모리를 관리하기위해 결국 어떤게 아직 쓰고있고 쓰지않는지 판별을 해야함 그 기준값들이 Reachable과 Unreachable  
![1](https://github.com/witwint/TIL/assets/108222981/fa509670-4298-4c5f-bbd3-792978a3b83f)

`stack`에서 생성된 `heap`의 객체를 참조한다고 하면 (아 이`heap`의 객체가 `stack`에 연결되어있으니 아직 사용중이구나) 실제사용중인걸 알 수 있다.  
이런 요소들이 몇가지 있다.  

객체가 가질 수 있는 참조 종류

* 힙 내의 다른 객체에 의한 참조
* Java 스택, 즉 Java 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조
* 네이티브 스택, 즉 JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
* 메서드 영역의 정적 변수에 의한 참조

이중 힙 내의 다른 객체에 의한 참조만 제외하면 모두 사용중인것을 알 수있다. 즉 아래 3가지 요소일때 Reachable 이라하고  
아닐떄 Unreachable로 판별하여 Unreachable를 제거하는 방식으로 진행된다는것  


---

## 20240330
### 루시카토 CS 인터넷 통신(http)

#### 인트로  
http 통신에 대해서 알아보자 그전에 기본적인 TCP/IP 4계층만 잠깐 확인하고 넘어자가  

#### 인터넷 프로토콜 스택의 4계층  
* 애플리케이션 계층  - HTTP,FTP
* 전송계층 - TCP, UDP
* 인터넷 계층 - IP
* 네트워크 인터페이스 계층

#### TCP
전송 제어 프로토콜(Transmission Control Protocol)


* 연결지향 - TCP 3 way handshake (가상 연결)
* 데이터 전달 보증
* 순서 보장
* 신뢰할 수 있는 프로토콜
* 현재는 대부분 TCP 사용

#### UDP
사용자 데이터그램 프로토콜(User Datagram Protocol)
* 하얀 도화지에 비유(기능이 거의 없음)
* 연결지향 X - TCP 3 way handshake X
* 데이터 전달 보증 X
* 순서 보장 X
* 데이터 전달 및 순서가 보장되지 않지만, 단순하고 빠름
* 정리
	* IP와 거의 같다. +PORT +체크섬 정도만 추가
	* 애플리케이션에서 추가 작업 필요

#### PORT

* 0 ~ 65535 할당 가능
	* 0 ~ 1023: 잘 알려진 포트, 사용하지 않는 것이 좋음
	* FTP - 20, 21
	* TELNET - 23
	* HTTP - 80
	* HTTPS - 443

#### DNS
도메인 네임 시스템(Domain Name System)

* 전화번호부
* 도메인 명을 IP 주소로 변환

#### HTTP 메시지 전송

1. 웹 브라우저가 HTTP 메시지 생성
2. SOCKET 라이브러리를 통해 전달
	* A: TCP/IP 연결(IP, PORT)
	* B: 데이터 전달
3. TCP/IP 패킷 생성, HTTP 메시지 포함

#### HTTP 
HyperText Transfer Protocol

* 클라이언트 서버 구조
* Stateful, Stateless
* 비 연결성(connectionless)
* HTTP 메시지
* 단순함, 확장 가능

##### 클라이언트 서버 구조
* Request Response 구조
* 클라이언트는 서버에 요청을 보내고, 응답을 대기
* 서버가 요청에 대한 결과를 만들어서 응답

##### 무상태 프로토콜 
* 서버가 클라이언트의 상태를 보존X
* 장점: 서버 확장성 높음(스케일 아웃)
* 단점: 클라이언트가 추가 데이터 전송
(모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다 ex 로그인)

##### 비 연결성
* HTTP는 기본이 연결을 유지하지 않는 모델
* 일반적으로 초 단위의 이하의 빠른 속도로 응답
* 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작음
* 서버 자원을 매우 효율적으로 사용할 수 있음
* TCP/IP 연결을 새로 맺어야 함 - 3 way handshake 시간 추가
* 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등등 수 많은 자원이 함께 다운로드
* 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결
* HTTP/2, HTTP/3에서 더 많은 최적화

#### Http 메시지
![1](https://github.com/witwint/TIL/assets/108222981/06a4ea1b-6f31-431d-bb3d-629765e9bc42)

---

## 20240402
### 스프링 AOP 포인트컷 매개변수 전달,this, target

#### 매개변수 전달

다음은 포인트컷 표현식을 사용해서 어드바이스에 매개변수를 전달할 수 있다.
this, target, args,@target, @within, @annotation, @args

```java
@Before("allMember() && args(arg,..)")
public void logArgs3(String arg) {
log.info("[logArgs3] arg={}", arg);
}
```
이런식으로 사용하면 `memberService.hello("helloA");` 여기의 `helloA`를 받아올 수 있는 느낌이다. 


상세 예시들
```java
@Slf4j
@Aspect
static class ParameterAspect {
@Pointcut("execution(* hello.aop.member..*.*(..))")
private void allMember() {}
@Around("allMember()")
public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
Object arg1 = joinPoint.getArgs()[0];
log.info("[logArgs1]{}, arg={}", joinPoint.getSignature(), arg1);
return joinPoint.proceed();
}
@Around("allMember() && args(arg,..)")
public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws
Throwable {
log.info("[logArgs2]{}, arg={}", joinPoint.getSignature(), arg);
return joinPoint.proceed();
}
@Before("allMember() && args(arg,..)")
public void logArgs3(String arg) {
log.info("[logArgs3] arg={}", arg);
}
@Before("allMember() && this(obj)")
public void thisArgs(JoinPoint joinPoint, MemberService obj) {
log.info("[this]{}, obj={}", joinPoint.getSignature(),
obj.getClass());
}
@Before("allMember() && target(obj)")
public void targetArgs(JoinPoint joinPoint, MemberService obj) {
log.info("[target]{}, obj={}", joinPoint.getSignature(),
obj.getClass());
}
@Before("allMember() && @target(annotation)")
public void atTarget(JoinPoint joinPoint, ClassAop annotation) {
log.info("[@target]{}, obj={}", joinPoint.getSignature(),
annotation);
}
@Before("allMember() && @within(annotation)")
public void atWithin(JoinPoint joinPoint, ClassAop annotation) {
log.info("[@within]{}, obj={}", joinPoint.getSignature(),
annotation);
}
@Before("allMember() && @annotation(annotation)")
public void atAnnotation(JoinPoint joinPoint, MethodAop annotation) {
log.info("[@annotation]{}, annotationValue={}",
joinPoint.getSignature(), annotation.value());
}
}
```
`logArgs1` : `joinPoint.getArgs()[0]` 와 같이 매개변수를 전달 받는다.  
`logArgs2` : `args(arg,..)` 와 같이 매개변수를 전달 받는다.  
`logArgs3` : `@Before` 를 사용한 축약 버전이다. 추가로 타입을 `String` 으로 제한했다.  
`this` : 프록시 객체를 전달 받는다.  
`target` : 실제 대상 객체를 전달 받는다.  
`@target` , `@within` : 타입의 애노테이션을 전달 받는다.  
`@annotation` : 메서드의 애노테이션을 전달 받는다. 여기서는 `annotation.value()` 로 해당 애노테이션의 값을 출력하는 모습을 확인할 수 있다.  

#### this, target

**정의**
`this` : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트  
`target` : Target 객체(스프링 AOP 프록시가 가리키는 실제 대상)를 대상으로 하는 조인 포인트  

**설명**
`this` , `target` 은 다음과 같이 적용 타입 하나를 정확하게 지정해야 한다.  

```java
this(hello.aop.member.MemberService)
target(hello.aop.member.MemberService)

* 같은거 불가, 부모타입은 허용
```

**주의점**  
두개를 사용하는데 조금 다른부분이 있는데  
`this` 는 스프링 빈으로 등록되어 있는 **프록시 객체**를 대상으로 포인트컷을 매칭한다.  
`target` 은 실제 **target 객체**를 대상으로 포인트컷을 매칭한다.  

이는 스프링 AOP에서 JDK동적프록시를 쓰냐, CGLIB를 쓰냐에 따라서 결과가 다르게 나올 수 있다.

**JDK**  

상황 : service(인터페이스) -> serviceImpl(상속받아만든클래스)  
스프링빈 : **인터페이스 상속받은 사용한프록시** , (serviceImpl존재여부도 모름)  

target을 사용하면 `상황 : service(인터페이스) -> serviceImpl(상속받아만든클래스)`이쪽의 객체를 보면  
`@Around("target(hello.aop.member.MemberService)")`
`@Around("target(hello.aop.member.MemberServiceImpl)")`
뭘하든 적용이 된다.

하지만 this를 사용하면 `스프링빈 : **인터페이스 상속받은 사용한프록시** , (serviceImpl존재여부도 모름)` 이 상황이기 때문에
`@Around("this(hello.aop.member.MemberService)")` (`**스프링빈 : **인터페이스 상속받은 사용한프록시**` 인터페이스는 OK)
`@Around("this(hello.aop.member.MemberServiceImpl)")` (???? `MemberServiceImpl` 아에 모름)  적용 x  

가장 아래 코드는 적용이 되지않는다.  

**CGLIB**  

상황 : service(인터페이스) -> serviceImpl(상속받아만든클래스)  
스프링빈 : **serviceImpl을 상속받은 프록시** , (프록시 -> `serviceImpl` -> service(인터페이스) 거슬러 올라가며 확인가능)  

`@Around("target(hello.aop.member.MemberService)")`  
`@Around("target(hello.aop.member.MemberServiceImpl)")`  
둘다 적용  

`@Around("this(hello.aop.member.MemberService)")` (`**스프링빈 : **인터페이스 상속받은 사용한프록시**` 인터페이스는 OK)  
`@Around("this(hello.aop.member.MemberServiceImpl)")` (프록시 -> `serviceImpl` -> service(인터페이스)) 프록시의 부모가 `serviceImpl`이라 확인가능  


여담
```java
/**
* application.properties
* spring.aop.proxy-target-class=true CGLIB
* spring.aop.proxy-target-class=false JDK 동적 프록시
*/
@Slf4j
@Import(ThisTargetTest.ThisTargetAspect.class)
@SpringBootTest(properties = "spring.aop.proxy-target-class=false") //JDK 동적 프록
시
//@SpringBootTest(properties = "spring.aop.proxy-target-class=true") //CGLIB
public class ThisTargetTest {
```
기본 스프링의 프록시는 모두 CGLIB로 만들어지는데 기본적으로 properties파일 설정으로 기본값을 변경할 수 있다.  
하지만 프로젝트 전체에서 변경하고 테스트하는건 번거롭기때문에 test 클래스에 어노테이션을 붙어주는거로 테스트할때만 변경할 수 있다.  
참고로 JDK 동적 프록시로 설정해도 인터페이스가 아에 없는건 CGLIB로 생성된다.  

---
## 20240403  
### DB 파티셔닝

#### DB 파티셔닝이란

* 서비스가 커지면 데이터가 대용량화되면서 DB시스템의 용량과 성능 한계가 생기게 됩니다.
* 이런 이슈를 해결하기 위해서 `table`을 파티션 단위로 나누어 관리하는 파티셔닝 기법이 나타나게 됩니다.
* 즉 큰 `table`을 파티션이라는 작은 단위로 분리하는것을 말합니다.

**장점**
* 특정DML 쿼리성능 향상
* 대용량 Data WRITE환경
* Full Scan에서 범위축소
* OLTP(온라인트랜랙션) 에서 INSERT 작업을 파티션단위로 분산시켜 경합감소

* 회손가능성감소
* 파티션별 독립적 백업, 복구 가능
* table의 partition 단위로 Disk I/O을 분산하여 경합을 줄이기 때문에 UPDATE 성능을 향상
* 관리가 용의

**단점**
* table간 JOIN비용 증가
* table과 index를 별도로 파티셔닝할 수 없음 (같이 해야함)

#### DB 파티셔닝의 종류

**수평 파티셔닝**
![1](https://github.com/299unknown/diary/assets/151738362/9e1d2abf-4cf4-4943-9e23-9784fa3ab9d9)

하나의 테이블의 각 행을 다른 테이블에 분산시키는 것입니다.

* 샤딩과 동일한 개념
* 가장 일반적인 파티셔닝
* KEY를 기반으로 분산저장

* 장점
	* 데이터 개수가 작아지고 index개수도 작아지기 때문에 성능향상
	* 데이터 개수 기준 파티셔닝

* 단점
	* 서버 연결과정 많아짐
 	* 데이터를 찾는 과정이 기존보다 복잡하기 때문에 latency증가
  * 하나의 서버가 고장나면 데이터 무결성 깨질 수 있음

**수직 파티셔닝**
![1](https://github.com/299unknown/diary/assets/151738362/57b460f1-2e59-4b46-a9af-8c798436dcb4)

테이블의 일부 열을 빼내는 형태로 분할합니다.

* 관계형 DB에서 3정규화와 같은 개념으로 접근하면 이해하기 쉬움
* 하지만 수직 파티셔닝은 이미 정규화된 데이터를 분리하는 과정
* 예) 유저의 주소를 보안이슈등으로 CustomerId를 참조하여 다른 테이블 분리

* 장점
	* 자주 사용하는 컬럼 등을 분리시켜 성능을 향상시킬 수 있음
	* I/O 측면에서 봤을 때 필요한 컬럼만 올리면 훨씬 많은 수의 ROW를 메모리에 올릴 수 있으니 성능상의 이점이 있음
	* 데이터 타입끼리 파티셔닝하게되면 저장시 데이터 압축률 높일 수 있음
 
#### DB 파티셔닝 분할 기준

데이터베이스 관리 시스템은 분할에 대해 각종 기준(분할 기법)을 제공하고 있다. 분할은 ‘분할 키(partitioning key)’를 사용합니다.

![1](https://github.com/299unknown/diary/assets/151738362/50d0bc68-3899-439f-a0c8-9f86817c8156)

* 범위 분할 (range partitioning)
	* 분할 키 값이 범위 내에 있는지 여부로 구분한다.
	* 예를 들어, 우편 번호를 분할 키로 수평 분할하는 경우이다.

* 목록 분할 (list partitioning)
	* 값 목록에 파티션을 할당 분할 키 값을 그 목록에 비추어 파티션을 선택한다.
	* 예를 들어, Country 라는 컬럼의 값이 Iceland , Norway , Sweden , Finland , Denmark 중 하나에 있는 행을 빼낼 때 북유럽 국가 파티션을 구축 할 수 있다.

* 해시 분할 (hash partitioning)
	* 해시 함수의 값에 따라 파티션에 포함할지 여부를 결정한다.
	* 예를 들어, 4개의 파티션으로 분할하는 경우 해시 함수는 0-3의 정수를 돌려준다.

* 합성 분할 (composite partitioning)
	* 상기 기술을 결합하는 것을 의미하며, 예를 들면 먼저 범위 분할하고, 다음에 해시 분할 같은 것을 생각할 수 있다.
	* 컨시스턴트 해시법은 해시 분할 및 목록 분할의 합성으로 간주 될 수 있고 키 공간을 해시 축소함으로써 일람할 수 있게 한다.

#### MySQL Partition종류
* Range
> 범위를 기반으로 파티션을 나누는 형식이다.
> Range Partition은 날씨 기반 데이터가 누적되고 날짜에 따라 분석 삭제할 경우, 범위 기반으로 데이터를 여러 파티션에 균등하게 나눌 수 있는 경우, 대량의 과거 데이터 삭제 같은 경우에 사용한다.
> Range Partition에서 null은 어떤 값 보다 작은 값으로 취급되기 때문에 컬럼에 null인 데이터가 insert 된다면 가장 작은 값을 저장하는 파티션에 저장된다.

```sql
CREATE TABLE test (
    id INT NOT NULL,
    lname VARCHAR(30),
    datt DATE NOT NULL DEFAULT '2000-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    store_id INT NOT NULL
) PARTITION BY RANGE (YEAR(datt)) (
    PARTITION p0 VALUES LESS THAN (2020) ,
    PARTITION p1 VALUES LESS THAN (2021) ,
    PARTITION p2 VALUES LESS THAN (2022) ,
    PARTITION p3 VALUES LESS THAN MAXVALUE
    );
```
연도를 범위를 기준으로 파티션을 생성한 것이다.

* List
> List Partition은 RangePartition과 비슷하고,코드나 카테고리 등 특정 값을 기반으로 파티션을 나눈다.
> List Partition은 파티션 키 값이 코드 값이나 카테고리와 같이 고정 값일 경우에 사용하고 파티션 키 값을 기준으로 레코드 건수가 균일하고 검색 조건에 파티션 키가 자주 사용되는 경우에 사용한다.
>List는 Range와 다르게 null을 명시할 수 있지만, MAXVALUE는 지정할 수 없다.

```sql
CREATE TABLE test (
    id INT NOT NULL,
    name VARCHAR(30),
    datt DATE NOT NULL DEFAULT '2000-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
) PARTITION BY LIST (job_code) (
    PARTITION p0 VALUES IN (2) ,
    PARTITION p1 VALUES IN (1,9) ,
    PARTITION p2 VALUES IN (3,6,7) ,
    PARTITION p3 VALUES IN (4,5,8, NULL)
    );
```

* Hash
> Hash Partition은 Hash 함수에 의해 레코드가 저장될 파티션을 결정하는 방식이다.
> Hash Partition은 테이블의 모든 레코드가 비슷한 사용빈도를 보이지만 너무 커서 파티션이 필요한 경우 사용된다.

```sql
CREATE TABLE test (
    id INT NOT NULL,
    name VARCHAR(30),
    datt DATE NOT NULL DEFAULT '2000-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
) PARTITION BY HASH (id)
PARTITIONS 4;
```

* Key
> 파티션 키 값을 MD5 함수를 이용하여 계산한 후 MOD 연산하여 파티션에 분배하는 방식. 정수 타입 외에 다른 데이터 타입들을 파티션 키로 사용할 수 있다는 것이 해시 파티션과의 차이점이다.

```sql
CREATE TABLE employees(
	id INT NOT NULL,
	...
) PARTITION BY KEY() PARTITIONS 4;
```


#### MySQL Partition

연월 별 데이터를 나누고 싶다면 아래와 같이 정의할 수 있습니다.

```sql
mysql> CREATE TABLE users (
    user_id INT NOT NULL, 
    reg_at DATETIME NOT NULL,
    ...
    PRIMARY KEY(user_id, reg_at)
) PARTITION BY RANGE (TO_DAYS(reg_at)) (
	PARTITION P_202206 VALUES LESS THAN (TO_DAYS('2022-07-01')),
	PARTITION P_202207 VALUES LESS THAN (TO_DAYS('2022-08-01')),
	PARTITION P_202208 VALUES LESS THAN (TO_DAYS('2022-09-01')),
	PARTITION P_maxvalue VALUES LESS THAN MAXVALUE
);
```
`PRIMARY KEY(user_id, reg_at)`여기서 파티션을 할 컬럼은 PRIMARY KEY로 등록되어야 합니다.   
참고로, 파티션은 동일한 테이블의 형식으로 나눌 뿐, 파티션 별 다른 형태를 갖지 않습니다.  
가령 각각 다른 인덱스를 생성하는 등의 형태는 지원하지 않습니다.  

users 테이블은 reg_at 칼럼에서 TO_DAYS( ) 라는 MySQL 내장 함수를 이용해 날짜만 추출하고, 그 날짜를 이용해 테이블을 연도 범위별로 파티션합니다.  

(mysql 내장함수들)
```sql
ABS(), CEILING(), EXTRACT(), FLOOR(), MOD(),
DATEDIFF(), DAY(), DAYOFMONTH(), DAYOFWEEK(), DAYOFYEAR(), HOUR(), MICROSECOND(), MINUTE(),
MONTH(), QUARTER(), SECOND(), TIME_TO_SEC(), TO_DAYS(), TO_SECONDS(), UNIX_TIMESTAMP(),
WEEKDAY(), YEAR(), YEARWEEK()
```

**1. PARTITION BY RANGE ( ... )**  
RANGE 기반의 파티셔닝을 한다는 의미입니다.  
RANGE 이외에도 LIST, HASH, KEY Partition 이 존재합니다.  
 
**2. TO_DAYS(reg_at)**  
Partition Key로 사용할 컬럼을 지정합니다.  
예시에서는 reg_at 컬럼을 지정했는데, 하나 이상의 컬럼이 Partition Key가 될 수 있습니다.  
TO_DAYS는 날짜 데이터를 정수형으로 변환시키기 위한 MySQL 내장함수입니다.  
내장 함수는 지난 포스팅의 Function 섹션을 참고해주세요.  
   
**3. PARTITION  VALUES LESS THAN (TO_DAYS('2022-07-01'))**  
파티셔닝을 진행할 기준 값들을 정의합니다.  
정의한 값을 기준으로 데이터를 삽입하고 조회하는 검색 시점의 기준 값이 됩니다.  
이 기준 값에 따라 데이터가 저장되기 때문에 데이터가 모든 파티션에 골고루 퍼지도록 잘 정의해야 합니다.  
   
**4. PARTITION P_maxvalue VALUES LESS THAN MAXVALUE**  
마지막 구문인 MAXVALUE 는 만약 정의한 날짜 값 그 이외의 값이 삽입될 때를 대비합니다.  
위의 예시에서 2022년 10월의 데이터, 혹은 그 이후의 들어온다면 처리되지 못하니, maxvalue 파티션에 추가하라는 의미입니다.  
-MAXVALUE <= intval <= MAXVALUE  
   
위와 같은 범위를 갖습니다.  


![1](https://github.com/299unknown/diary/assets/151738362/ae3a5e2d-dc8d-4902-9619-993a7cbd3012)
실제 데이터들은 파일로 관리되며, my.conf 내에 정의된 데이터 저장 위치에서 확인이 가능하죠.  
파티션 데이터는 <table_name>#P#<partition_name>.ibd 형식으로 나눠집니다.   
파티션 설명 중 논리적으로는 하나의 테이블과 같은데 물리적으로 분리된 구조라는 의미라는것이 이런 의미 입니다.  

**Alter Exist Table**
혹은 기존의 테이블을 파티셔닝할 수 있습니다. 
형식은 거의 동일합니다.

```sql
CREATE TABLE mirrorline.users_after (
  `user_id` INT NOT NULL AUTO_INCREMENT,
  `email` VARCHAR(100) NOT NULL,
  `name` VARCHAR(15) NOT NULL,
  `age` INT NOT NULL,
  `reg_at` DATETIME NOT NULL,
  PRIMARY KEY (`user_id`, `reg_at`)
);
```
기본 테이블이 이렇게 존재할때

```sql
ALTER TABLE users_after PARTITION BY RANGE (TO_DAYS(reg_at)) (
	PARTITION P_202206 VALUES LESS THAN (TO_DAYS('2022-07-01')),
	PARTITION P_202207 VALUES LESS THAN (TO_DAYS('2022-08-01')),
	PARTITION P_202208 VALUES LESS THAN (TO_DAYS('2022-09-01')),
	PARTITION P_maxvalue VALUES LESS THAN MAXVALUE
);
```
파티션을 추가합니다. 참고로, 데이터가 존재할 때에도 파티션으로 나눌 수 있습니다. 다만, 데이터가 많을 경우에 서비스되고 있는 테이블을 나누려면 부하가 크게 갈테니 주의하세요.  

**Partition 확인**
```sql
SELECT PARTITION_NAME,TABLE_ROWS
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_NAME = 'users';
```
<img width="549" alt="1" src="https://github.com/299unknown/diary/assets/151738362/6cc88b49-3522-46c6-8a7f-e5741ec93c1c">
  
모든 파티션과 해당 파티션에 데이터가 얼마나 존재하는지 확인할 수 있는 쿼리입니다.  
TABLE_ROWS는 SQL 옵티마이저의 최적화를 위한 추정값이기 때문에 항상 정확하지 않습니다.  


**예시**

```sql
INSERT INTO users(email, name, age, reg_at) 
VALUES('gngsn@gmail.com', 'gngsn', 25, '2022-12-23 12:02:28');
```

이런 값을 추가하면 

<img width="619" alt="1" src="https://github.com/299unknown/diary/assets/151738362/3f7b45e4-ada5-4f29-b564-09da50d2a786">

`p_maxvalue`쪽으로 자동으로 파티셔닝되게 됩니다.

**DROP**
```sql
ALTER TABLE users DROP PARTITION P_202207;
```

파티션을 제거하고 싶다면 아래와 같은 쿼리를 입력하면 됩니다.  
그럼, 기존 P_202206 파티션에 저장된 데이터들은 파티션과 함께 삭제됩니다.  

**SELECT**

```sql
SELECT * FROM users PARTITION (P_202207);        
SELECT * FROM users PARTITION (P_maxvalue);
```
이런식으로 파티션 하나만 조회할수도있습니다.  
`EXPLAN`으로 확인해보면 특정 파티션만 검색한것을 확인할 수 있습니다.  

**병합**
```sql
ALTER TABLE test 
REORGANIZE PARTITION p2,p3 INTO (
PARTITION p23 VALUES LESS THAN (2020)
);
```


**성능고려주의점**
파티션을 사용하면 성능이 오히려 떨어질 수 있습니다.
데이터 검색을 먼저 구분해보자면, WHERE 절의 조건으로 아래와 같은 사항을 따져볼 수 있습니다.
* 검색할 파티션을 찾을 수 있는지
* 인덱스를 효율적으로 사용할 수 있는지 (Index Range Scan)

* 파티션 선택 O, 인텍스 효율 O
	* 이때는 파티션의 개수와 관계없이 검색을 위해 꼭 필요한 파티션의 Index Range Scan하는 경우를 의미합니다. 두 조건이 모두 가능할 때 쿼리가 가장 효율적으로 처리될 수 있습니다.

* 파티션 선택 X, 인덱스 효율 O
	* 이 경우 우선 테이블의 모든 파티션을 대상으로 검색해야 하지만, 각 파티션에 Index Range Scan을 사용할 수 있습니다.
	* 최종적으로 테이블에 존재하는 모든 파티션의 개수만큼 Index Range Scan 검색 하게 됩니다. 
	* 이 작업은 파티션 개수만큼의 테이블에 대해 인덱스 레인지 스캔을 한 다음, 결과를 병합해서 가져오는 것과 같습니다.

* 파티션 선택 O, 인덱스 효율 X
	* 인덱스는 이용할 수 없어서 대상 파티션에 대해 Full Table Scan 하기 때문에, 각 파티션의 레코드 건수가 많다면 상당히 느리게 처리됩니다.

* 파티션 선택 X, 인텍스 효율 X
	* 테이블의 모든 파티션을 검색해야 하고 각 파티션에서도 Full Table Scan을 수행해야 합니다. 최악의 경우 입니다.


`파티션 선택 O, 인덱스 효율 X`, `파티션 선택 X, 인텍스 효율 X`의 경우 파티션을 지양하는것이 좋습니다.  


#### 특징

**INSERT**
![1](https://github.com/299unknown/diary/assets/151738362/55932079-8019-4bfe-9b11-2bb62fdb0764)
* Partition을 통해 데이터를 조회할 때는 파티션 키로 정한 데이터를 기준으로 파티션이 결정한 후, 파티션이 결정되면 나머지 과정은 파티션되지 않은 일반 테이블과 동일하게 처리됩니다.
* 최대한 파티션 키를 통해 데이터를 찾을 수 있게끔 조건을 걸어주는 것을 권장드립니다. 

**UPDATE**
![1](https://github.com/299unknown/diary/assets/151738362/209015b6-56b9-4c48-829b-4722586b6622)
데이터를 변경할 때는 파티션 키를 변경하는지 아닌지로 구분해서 설명할 수 있습니다.

* 파티션 키 외의 데이터 수정 : 먼저, 파티션 키 외의 데이터가 수정될 때에는 파티션이 적용되지 않은 일반 테이블과 마찬가지로 칼럼 값만 변경하면 됩니다.
* 파티션 키 칼럼이 변경될 때 : 아래와 같이 기존의 레코드가 저장된 파티션에서 해당 레코드를 삭제합니다. 그리고 변경되는 파티션 키 칼럼의 표현식을 평가한 후,그 결과를 이용해 레코드를 이동시킬 새로운 파티션을 결정해서 레코드를 새로 저장합니다.

**Index**
![1](https://github.com/299unknown/diary/assets/151738362/d233bfb7-b1a2-47ec-a673-423362c0c81b)
모든 인덱스는 파티션 단위로 생성됩니다.  
MySQL의 파티션 테이블에서 인덱스는 전부 로컬 인덱스로, 테이블 단위의 글로벌한 인덱스는 지원하지 않습니다.  

파티션되지 않은 테이블에서는 인덱스를 순서대로 읽으면 그 칼럼으로 정렬된 결과를 바로 얻을 수 있지만, 파티션된 테이블은 인덱스가 분리되어 있기 때문에 다르게 동작합니다.  

일반 테이블의 인덱스 스캔처럼 결과를 바로 반환하는 것이 아니라, 여러 파티션에 대해 인덱스 스캔을 수행할 때 각 파티션으로부터  
조건에 일치하는 레코드를 정렬된 순서대로 읽으면서 우선순위 큐(Priority Queue)에 임시로 저장하여 가져옵니다.  
  
각 파티션 에서 읽은 데이터가 이미 정렬되어 있기 파티션에 접근한 후 그 순서대로만 가져오며, 내부적으로 큐 처리를 합니다.  

#### 성능 확인

최적화 단계에서 필요한 파티션만 골라내고 불필요한 것 들은 실행 계획에서 배제, 옵티마이저가 다수의 파티션 중 일부만 읽어도 된다고 판단되면 불필요한 파티션에는 전혀 접근하지 않습니다.  

```sql
mysql> EXPLAIN SELECT * FROM users WHERE reg_at > '20220801' AND reg_at > '20220801';
+----+------------+------------+-------+---------+---------+------+--------------------------+
| id | table      | partitions | type  | key     | key_len | rows | Extra                    |
+----+------------+------------+-------+---------+---------+------+--------------------------+
|  1 | tb_article | P_202208      | index | PRIMARY | 9       |    1 | Using where; Using index |
+----+------------+------------+-------+---------+---------+------+--------------------------+
```
#### 참고자료
https://gmlwjd9405.github.io/2018/09/24/db-partitioning.html  
https://gngsn.tistory.com/203  
https://gngsn.tistory.com/204?category=851218  

---
## 20240404
### DB seeding 초기 데이터

#### 인트로 

DB를 사용하다보면 초기 데이터가 필요한 경우가 있습니다. 테이블의 경우 ORM등의 기술이나 프로젝트의 엔티티등을 통해서 초기생성 가능하지만 초기 데이터 까지 추기하기 위해서는 seeding을 해주어야 합니다.

#### 사용

javascript 진영에서는 prisma ORM을 통해서 가능하다고 확인 했지만 지금 프로젝트에 맞는 Spring의 경우 보통 어떻게 진행하는지 알아보았습니다.

일반적으로 프로젝트를 docker를 사용한 환경에서 사용하게 되는데 이때 docker-compose 파일 작성을 통해 미리 입력한 sql문들을 mysql이 시작될때 입력할 수 있습니다.

```docker

version: "3.8"
services:
  mysql:
    container_name: mysql_local
    image: mysql:8.0.30
    volumes:
      - ./db/conf.d:/etc/mysql/conf.d
      - ./db/initdb.d:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=pass_local
      - MYSQL_USER=pass_local_user
      - MYSQL_PASSWORD=passlocal123
      - MYSQL_ROOT_PASSWORD=passlocal123
      - TZ=Asia/Seoul
```

**초기화 설정**
처음 인스턴스를 생성할 때 추가적인 데이터베이스 초기화를 위해 실행할 .sh 또는 .sql 파일이 있다면, 컨테이너의 /docker-entrypoint-initdb.d 디렉토리에 해당 파일이 들어있는 디렉토리를 마운트 시켜주어야 한다. (파일명의 알파벳 순서로 실행)  


```sql
# ./db/initdb.d/create_table.sql

CREATE TABLE `users`
(
    `user_id`  int         NOT NULL AUTO_INCREMENT COMMENT '유저 식별값',
    `username` varchar(30) NOT NULL COMMENT '유저 로그인 아이디',
    `password` varchar(50) NOT NULL COMMENT '유저 패스워드',
    `created_at`   timestamp   NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '생성 일시',
    `modified_at`  timestamp            DEFAULT NULL COMMENT '수정 일시',
    PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='유저';
```
이런식의 테이블생성 쿼리도 미리 직성할 수 있고

```sql
# ./db/initdb.d/insert_data.sql

INSERT INTO `user` (username, password, create_at)
VALUES ('user1', 'password1', '2022-08-01 00:00:00'),
       ('user2', 'password2', '2022-08-01 00:00:00'),
       ('user3', 'password3', '2022-08-01 00:00:00');

...
```
seed 데이터를 넣을 수 있는 쿼리도 작성할 수 있습니다.


#### 참고자료
https://seungtaek-overflow.tistory.com/22  
https://yoo-dev.tistory.com/14  

---

## 20240305
### 프로그래머스 올바른괄호 큐/스택 2단계

'(' 또는 ')' 로만 이루어진 문자열 s가 주어졌을 때, 문자열 s가 올바른 괄호이면 true를 return 하고, 올바르지 않은 괄호이면 false를 return 하는 solution 함수를 완성해 주세요.  

"()()" true  
")()("false  

```java
import java.util.*;
class Solution {
    boolean solution(String s) {
        Stack<Character> stack = new Stack<>();
            for (int i = 0; i < s.length(); i++) {
                if (s.charAt(i) == '(') {
                    stack.push('(');
                } else if (s.charAt(i) == ')') {
                    if (stack.isEmpty()) {
                        return false;
                    }
                    stack.pop();
                }
            }
            return stack.isEmpty();
    }
}
```
스택에 넣는데 처음에 "("부터 넣으면서 ")"가 나오면 뺸다  
이과정에서 스트링이 끝나기전에 스택이 비거나 "("가 남으면 실패  

---

## 20240406
### 프로그래머스 체육복 그리드 1단계

점심시간에 도둑이 들어, 일부 학생이 체육복을 도난당했습니다. 다행히 여벌 체육복이 있는 학생이 이들에게 체육복을 빌려주려 합니다. 학생들의 번호는 체격 순으로 매겨져 있어, 바로 앞번호의 학생이나 바로 뒷번호의 학생에게만 체육복을 빌려줄 수 있습니다. 예를 들어, 4번 학생은 3번 학생이나 5번 학생에게만 체육복을 빌려줄 수 있습니다. 체육복이 없으면 수업을 들을 수 없기 때문에 체육복을 적절히 빌려 최대한 많은 학생이 체육수업을 들어야 합니다.  전체 학생의 수 n, 체육복을 도난당한 학생들의 번호가 담긴 배열 lost, 여벌의 체육복을 가져온 학생들의 번호가 담긴 배열 reserve가 매개변수로 주어질 때, 체육수업을 들을 수 있는 학생의 최댓값을 return 하도록 solution 함수를 작성해주세요.  

제한사항
전체 학생의 수는 2명 이상 30명 이하입니다.  
체육복을 도난당한 학생의 수는 1명 이상 n명 이하이고 중복되는 번호는 없습니다.  
여벌의 체육복을 가져온 학생의 수는 1명 이상 n명 이하이고 중복되는 번호는 없습니다.  
여벌 체육복이 있는 학생만 다른 학생에게 체육복을 빌려줄 수 있습니다.  
여벌 체육복을 가져온 학생이 체육복을 도난당했을 수 있습니다. 이때 이 학생은 체육복을 하나만 도난당했다고 가정하며, 남은 체육복이 하나이기에 다른 학생에게는 체육복을 빌려줄 수 없습니다.  

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        int n = 5;
        int[] lost = {2, 4};
        int[] reserve = {1,2,5};
        Solution solution = new Solution();
        System.out.println("Hello World");

        int arr3 = solution.solution(n, lost, reserve);
        System.out.println(arr3);

    }

    static class Solution {
        public int solution(int n, int[] lost, int[] reserve) {
            ArrayList<Integer> arr = new ArrayList<>();
            Arrays.sort(lost);
            Arrays.sort(reserve);
            for (int i : lost) {
                arr.add(i);
            }
            ArrayList<Integer> arr2 = new ArrayList<>();
            for (int i : reserve) {
                arr2.add(i);
            }
            int answer = n - arr.size();

            for (int i : reserve ) {
                if (arr.contains(i)) {
                    answer++;
                    arr.remove(Integer.valueOf(i));
                    arr2.remove(Integer.valueOf(i));

                }
            }
            for (int i : arr2) {
                if (arr.contains(i - 1)) {
                    answer++;
                    arr.remove(Integer.valueOf(i-1));
                    arr2.remove(Integer.valueOf(i-1));
                }
                else if (arr.contains(i + 1)) {
                    answer++;
                    arr.remove(Integer.valueOf(i+1));
                    arr2.remove(Integer.valueOf(i+1));
                }
            }
            return answer;
        }
    }
}
```
하 이 문제 어이없는게 테스트케이스중에 정렬이 안된것이 있다고 했다. 아무리봐도 친절하게 순서데로 나오는거같은데 진짜 너무한다. `Arrays.sort(lost);`이거 안해줘서 몇시간 고민함  

---

## 20240408  
### 스프링 AOP 실전 예제

실전에서 사용하는 AOP적용법을 간단하게 알아보자 

EX 예외터지면 해당함수를 일정 횟수 반복하는 AOP

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Retry {
 int value() default 3;
}
```
1. 붙여줄 어노테이션을 만들어준다.

```java
@Slf4j
@Aspect
public class RetryAspect {
 
 @Around("@annotation(retry)")
 public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws
Throwable {
 log.info("[retry] {} retry={}", joinPoint.getSignature(), retry);
 int maxRetry = retry.value();
 Exception exceptionHolder = null;
 for (int retryCount = 1; retryCount <= maxRetry; retryCount++) {
 try {
 log.info("[retry] try count={}/{}", retryCount, maxRetry);
 return joinPoint.proceed();
 } catch (Exception e) {
 exceptionHolder = e;
 }
 }
 throw exceptionHolder;
 }
}
```
2. 해당 어노테이션이 붙은 코드를 일정횟수 반복하게 하는 `@Aspect`를 만들어준다.

```java
@Repository
public class ExamRepository {
 @Trace
 @Retry(value = 4)
 public String save(String itemId) {
 //...
 }
}
```
3. 사용할 함수에 붙여준다.

```java
@SpringBootTest
@Import({TraceAspect.class, RetryAspect.class}) //TraceAspect.class이건 또다른 예시임
public class ExamTest {
}
```
4. Bean등록을 까먹지않는다.

5. 사용해주면 끝

이런식으로 어노테이션형식으로 `@Aspect`만 만들어준후 가져다 붙이고 Bean으로 등록만하면 끝이다.  

---

## 20240410
### 알고리즘 프로그래머스 조이스틱 그리디 2단계  
조이스틱으로 알파벳 이름을 완성하세요. 맨 처음엔 A로만 이루어져 있습니다. ex) 완성해야 하는 이름이 세 글자면 AAA, 네 글자면 AAAA  조이스틱을 각 방향으로 움직이면 아래와 같습니다.  
위 - 다음 알파벳  
아래 - 이전 알파벳 (A에서 아래쪽으로 이동하면 Z로)  
왼쪽 - 커서를 왼쪽으로 이동 (첫 번째 위치에서 왼쪽으로 이동하면 마지막 문자에 커서)  
오른쪽 - 커서를 오른쪽으로 이동 (마지막 위치에서 오른쪽으로 이동하면 첫 번째 문자에 커서)  

만들고자 하는 이름 name이 매개변수로 주어질 때, 이름에 대해 조이스틱 조작 횟수의 최솟값을 return 하도록 solution 함수를 만드세요.  

ex "JEROEN" 56  

```java
import java.util.*;
class Solution {
    public int solution(String name) {
        int answer = 0;
        int length = name.length();

        int index; // 다음 값들을 확인할 때 사용
        int move = length - 1; // 좌우 움직임 수를 체크

        for(int i = 0; i < name.length(); i++){
            answer += Math.min(name.charAt(i) - 'A', 'Z' - name.charAt(i) + 1);

            index = i + 1;
            // 연속되는 A 갯수 확인
            while(index < length && name.charAt(index) == 'A'){
                index++;
            }

            // 순서대로 가는 것과, 뒤로 돌아가는 것 중 이동수가 적은 것을 선택
            move = Math.min(move, i * 2 + length - index);
            // 2022년 이전 테스트 케이스만 확인하면 여기까지해도 정답처리가 되기 때문에, 이전 정답들에는 여기까지만 정리되어 있지만,
            // BBBBAAAAAAAB 와 같이, 처음부터 뒷부분을 먼저 입력하는 것이 더 빠른 경우까지 고려하려면 아래의 코드가 필요합니다.
            move = Math.min(move, (length - index) * 2 + i);
        }
        return answer + move;
    }
}
```
위아래 계산은 어렵지않은데 앞으로갈지 뒤로갈지, 앞으로 갔다가 다시 되돌아와서 뒤로갈지 반대인지 이거 계산하는게 많이 어려웠다.  
그래서 결국 답을 봤는데 처음 위아래 영어선택 계산과 더불어서 그 위치에서 `while`로 앞으로 중복되는 `A`의 개수를 찾고 그다음 그 위치에서 최적의 좌우 이동 계산하는 연산이  
복합적으로 들어가는걸 생각하기 어려웠다.  

2중 for문 이나 while생각하면 맨날 배열 2중으로 돌기만 생각했는데 좀더 넓게 생각해야겠다.  

---

## 20240411
### 스프링 AOP 실무 주의사항 문제1  

스프링 AOP의 첫번째 문제는 함수내부호출의 경우 이다.  

```java
public class CallServiceV0 {
 public void external() {
 log.info("call external");
 internal(); //내부 메서드 호출(this.internal())
 }
 public void internal() {
 log.info("call internal");
 }
}


@Aspect
public class CallLogAspect {
 @Before("execution(* hello.aop.internalcall..*.*(..))")
 public void doLog(JoinPoint joinPoint) {
 log.info("aop={}", joinPoint.getSignature());
 }
}
```
`external`, `internal`모두 AOP로 프록시 등록해서 스프링 빈에는 둘의 프록시 객체가 등록되게된다.  
하지만 `external`에서 함수 내부에서 자신의 클래스의 `internal`을 호출하게되면 스프링빈의 프록시 `internal`이 아니라 날것의 `this.internal`이 호출되기 때문에  
`internal`에는 AOP가 적용이 되지않는 문제가 있다.  

만약 AspectJ를 직접 사용하면 `public void internal()`함수에 코드안에 직접 코드를 쑤셔박아 넣는 방식을 사용할 수 있기때문에 이런 문제가 안생길 수 있지만 너무 복잡하기에 다른 해결책을 사용한다.  

그해결책은 다음시간에  

---
## 20240412
### 프록시 AOP 실무주의사항 문제1 해결법

#### 가장 권장하는 해결법

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class CallServiceV3 {
 private final InternalService internalService;
 public void external() {
 log.info("call external");
 internalService.internal(); //외부 메서드 호출
 }
}



@Slf4j
@Component
public class InternalService {
 public void internal() {
 log.info("call internal");
 }

```
그냥 아까 내부함수 호출하는쪽으로 새로운 클래스로 뽑아서 새로 만든다. 이것이 가장 권장하는방법이다.  
또는 클라이언트 쪽 한칸 뒷단계에서 external호출 -> internal호출 2줄로 작성할 수도 있긴하다.  

#### 권장하지 않는방법

1.

수정자 주입으로 자기자신을 다시 받아서 그 받은거를 `callServiceV1.internal(); `사용하는 방법이 있는데 자기자신이기때문에 생성자주입은 안되고 수정자로 사용해야한다.  
스프링2.6부터 어떤방식이든 순환참고 금지하는것으로 기본설정되어있다.


2
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class CallServiceV2 {
// private final ApplicationContext applicationContext;
 private final ObjectProvider<CallServiceV2> callServiceProvider;
 public void external() {
 log.info("call external");
// CallServiceV2 callServiceV2 = 
applicationContext.getBean(CallServiceV2.class);
 CallServiceV2 callServiceV2 = callServiceProvider.getObject();
 callServiceV2.internal(); //외부 메서드 호출
 }
```
스프링 빈 컨테이너에서 직접 꺼내쓰는 코드 방법이 있다. 너무 강제적으로 하는거같아서 좋지않다.  

---
## 20240413
### 스프링 퍼시스턴스 개발 고민

#### 인트로
스프링에는 JDBC -> 하이버네이트 -> JPA,QueryDSL 등의 DB 연결과 쿼리사용의 기술들이 있습니다. 자바의 객체지향적은 부분과 RDB를 매칭하기위해 많은 노력과 기술들이 나왔는데요.  
이런 많은 기술들을 어떻게 사용하는것이 올바른것인지 고민하게 되었습니다.  

#### 배경
기본적으로 JPA, QueryDSL 사용하게 되지만 어느선까지 기능을 이용하며 어떤 코드로 작성할것인가가 주요점입니다. 배경지식으로 영속성을 관리해주는 JPA부분만 조금 확인하고 넘어가겠습니다.  

영속성 컨텍스트란?  
ORM은 객체와 데이터베이스 테이블의 매핑을 통해 엔티티 클래스 객체 안에 포함된 정보를 테이블에 저장하는 기술이다.  
JPA에서는 테이블과 매핑되는 엔티티 객체 정보를 영속성 컨텍스트를 통해 애플리케이션 내에서 오래 지속되도록 보관한다.  
 
영속성 컨텍스트는 JPA를 이해하는데 가장 중요한 용어이다.  

영속성 컨텍스트는 논리적인 개념  
눈에 보이지 않음  
엔티티 매니저를 통해 영속성 컨텍스트에 접근  

 
엔티티의 생명주기  
![1](https://github.com/witwint/TIL/assets/108222981/2b5feffe-4b72-4682-acde-e7d10680b056)

* 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
* 영속(managed) : 영속성 컨텍스트에 관리되는 상태
* 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
* 삭제(remove) : 삭제된 상태

**비영속**

객체를 생성한 상태
```java
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");
```

**영속**
```java
EntityManager em = EntityManagerFactory.createEntityManager();
em.getTransaction().begin();

Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");

// 객체를 영속성 컨텍스트에 저장(영속)
em.persist(member);
```

**준영속**

```java
// member 엔티티를 영속성 컨텍스트에서 분리(준영속)
em.detach(member);
```

**삭제**

```java
// 객체를 삭제한 상태(삭제)
em.remove(member);
```

**영속성 상태의 장점**

1차 캐시
동일성(identity) 보장
트랜잭션을 지원하는 쓰기 지연
변경 감지
지연 로딩


영속성 컨텍스트(Persistence Context)를 그림으로 표현하면 다음과 같이 나타낼 수 있다.  
![2](https://github.com/witwint/TIL/assets/108222981/1fcbdd54-1dad-4cd1-85ae-ad99d7e1e939)

영속성 컨텍스트에는 1차 캐시 영역과 쓰기 지연 SQL 저장소 영역이 있다.  
JPA API 중에서 엔티티 정보를 영속성 컨텍스트에 저장하는 API를 사용하면, 영속성 컨텍스트의 1차 캐시에 엔티티 정보가 저장된다.  

```java
// 엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");

// 객체를 영속성 컨텍스트에 저장(영속)
em.persist(member);
```

**엔티티 등록 - 쓰기 지연**
엔티티 매니저는 데이터 변경 시 반드시 트랜잭션을 시작해야 한다.
```java
EntityManager em = EntityManagerFactory.createEntityManager();
EntityTransaction tx = em.getTransaction(); // 트랜잭션

// 트랜잭션 시작
tx.begin();

// 비영속
Member member = new Member();
member.setId("member1");
member.setUsername("홍길동");

// 영속
em.persist(member);

// 엔티티 등록
tx.commit();
```

* em.persist(member); : member 엔티티를 영속 컨텍스트에 저장하지만, 데이터베이스에는 반영되지 않는다.
* tx.commit(); : 트랜잭션을 커밋하는 순간 데이터베이스에 INSERT SQL을 보내 저장하게 된다.
* persist()를 실행할 때, 영속 컨텍스트의 1차 캐시에는 member 엔티티가 저장되고, 쓰기 지연 SQL 저장소에는 member 엔티티의 INSERT SQL 쿼리문이 저장된다.
* txcommit()을 실행하는 순간 쓰기 지연 SQL 저장소에 저장된 INSERT SQL 쿼리를 보내 데이터베이스에 저장하는 것이다.

따라서, 여러 개의 엔티티를 생성하고 persist를 하더라도, commit()을 하기 전에는 데이터베이스에 저장되지 않는다. 이를 쓰기 지연이라 하며, 영속 컨텍스트의 장점이다.

**엔티티 수정 - 변경 감지**
```java
EntityManager em = EntityManagerFactory.createEntityManager();
EntityTransaction tx = em.getTransaction(); // 트랜잭션

// 트랜잭션 시작
tx.begin();

// member 조회
Member member = em.find(Member.class, "member");
member.setUsername("hello");
member.setAge("20");

// 엔티티 등록
tx.commit();
```

* 엔티티의 수정은 set메서드를 통해서 변경한 뒤, 별다른 로직 없이 트랜잭션 커밋을 하는 순간에 업데이트된다.
* 이것이 가능한 이유는 바로 변경 감지(Dirty Checking) 기능을 제공하기 때문이다.
* 영속 컨텍스트의 1차 캐시에는 member의 초기 데이터가 저장되어 있을 것이다.
* 이후 set 메서드를 통해 데이터를 변경한다.
* 트랜잭션 커밋 시 flush()가 발생하면서 1차 캐시에서 엔티티와 스냅샷을 비교하여 변경에 대한 감지를 한다.
* 이후 SQL UPDATE 쿼리를 생성하여 쓰기 지연 SQL 저장소에서 쿼리를 보낸다.
* 이로써 DB에 저장된 데이터를 수정하게 된다.

**엔티티 삭제**
```java
Member member = em.find(Member.class, "member");

em.remove(member); // 엔티티 삭제
```

* 엔티티 삭제는 remove() 메서드를 통해 데이터를 삭제할 수 있다.
* 영속성 컨텍스트와 데이터베이스에서 모두 제거된다.

**플러시 - flush()**
트랜잭션 커밋을 실행하면 변경 내용을 데이터베이스에 반영하게 된다.
트랜잭션 커밋이 일어날 때 플러시도 함께 발생하여 데이터베이스에 반영할 수 있는 것이다.
즉, 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것이다.

**플러시 발생 시**

* 변경 감지(dirty checking)
* 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
* 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송(등록, 수정, 삭제 쿼리)

**영속성 컨텍스트를 플러시 하는 방법**

* em.flush() - 직접 호출(테스트에 사용)
* tx.commit() - 트랜잭션 커밋을 통한 자동 호출
* JPQL 쿼리 실행 - 플러시 자동 호출

**플러시에 대한 오해**

* 플러시는 영속성 컨텍스트를 비우지 않는다.
* 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 역할이다.
* 플러시의 개념은 트랜잭션이라는 작업 단위에 중요 → 커밋 직전에만 동기화하면 된다.

#### 엔티티에서 One To Many 단점

**Article과 Image 엔티티**
```java
 @Entity
    public class Article {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column
        private String content;

        @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true) 
        private List<Image> images = new ArrayList<>();

        public void addImage(final Image image) {
            images.add(image);
        }
    }
```
```java
 @Entity
    public class Image {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column
        private String url;

    }
```
@OneToMany 단방향에서 따로 조인 설정을 넣어주지 않으면 단방향 @JoinTable이 적용됩니다.

조인테이블 사용시 DB 예시
![1](https://github.com/witwint/TIL/assets/108222981/61b34c00-d0a6-43a8-9861-7d4105eb6ad8)


**저장로직, 결과**
```java
Article article = new Article("foo");

    article.addImage(new Image("foo 1"));
    article.addImage(new Image("foo 2"));
    article.addImage(new Image("foo 3"));
    article.addImage(new Image("foo 4"));

    articleRepository.saveAndFlush(article);
```
```sql
  Hibernate: insert into article (id, content) values (null, ?)

    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)

    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
```
각 테이블을 저장후에 사이의 테이블의 값이 추가로 들어갑니다. 이 경우 1:N 관계 보다는 N:N 연관 처럼 보이며 매우 효율적이지 않습니다. 또 한 세 개의 테이블이 사용되므로 필요한 것보다 더 많은 공간을 사용하고 있습니다.  

**삭제시 문제점**
```java
Image image = imageRepository.findById(1L).get();
    imageRepository.delete(image);
```
```sql
 PUBLIC.ARTICLE_IMAGES FOREIGN KEY(IMAGES_ID) REFERENCES PUBLIC.IMAGE(ID) (1)"; SQL statement:
    delete from image where id=? [23503-199]
```
에러 발생! article_images(중간 테이블) 테이블에서 Image의 id를 외래키로 가지고 있기 때문에 제거가 불가능합니다.  
Image를 삭제하는 방법은 Article의 Image List에서 remove 해줘야 합니다.  

```java
 Image image = imageRepository.findById(1L).get();
    article.getImages().remove(image);
    testEntityManager.flush();
```
```sql
  Hibernate: delete from article_images where article_id=?
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: insert into article_images (article_id, images_id) values (?, ?)
    Hibernate: delete from image where id=?
```

article_images 테이블 에서 article_id를 통해 모두 지운다.  
지우려는 image를 제외한 나머지 image들을 article_images에 다시 저장한다. ->????????  
지우려는 image를 테이블에서 삭제한다.  

이유는 단방향 연결이기때문에 `article.getImages().remove(image);`이런 코드를 서용한후 하이버네이트가 `article`입장에서 `image`를 전혀 모르기때문에 일단 나의 `article`에 해당하는 중간 테이블의 레코드를 전부다 지우고 나서 다시 추가하는 방식으로 쿼리를 날리게 됩니다.  

**@JoinColumn을 사용한 단방향 @OneToMany**
```java
public class Article{
    ....

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name="article_id")
    private List<Image> images = new ArrayList<>();

    ....
```
```sql
 Hibernate: insert into article (id, content) values (null, ?)

    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)
    Hibernate: insert into image (id, url) values (null, ?)

    Hibernate: update image set article_id=? where id=?
    Hibernate: update image set article_id=? where id=?
    Hibernate: update image set article_id=? where id=?
    Hibernate: update image set article_id=? where id=?
```
@JoinColumn을 사용하면 image를 DB에 저장할 때, article_id를 모르기 때문에 먼저 저장한 후에 update문을 통해서 article_id를 한 번 더 실행합니다.  

@JoinColumn사용시 DB  
![1](https://github.com/witwint/TIL/assets/108222981/1054891a-c2e1-4264-adaf-0d060836bcce)

**삭제**
```sql
    Hibernate: update image set article_id=null where article_id=? and id=?
    Hibernate: delete from image where id=?
```

**OneToMany 양방향**
```java
@Entity
public class Article{

    @OneToMany(mappedBy = "article",cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Image> images = new ArrayList<>();

    public void addImage(final Image image) {
        images.add(image);
        image.setArticle(this);
    }

    public void removeImage(final Image image){
        images.remove(image);
        image.setArticle(null);
    }
}

@Entity
public class Image {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "article_id")
    private Article article;

      ...
}
```

**저장**
```sql
Hibernate: insert into article (id, content) values (null, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
```

**삭제**
```sql
Hibernate: delete from image where id=?
```

딱 한 번씩, 간단하게 실행됩니다.  
양방향을 하면 이렇게 편하고 사용함에 있어서도 편한데 왜 양방향을 사용하지 않고 @OneToMany 단방향을 생각했을까요?  
객체는 가급적이면 단방향으로 해주는 게 좋습니다. 양방향으로 하면 신경써줘야 할 부분이 많죠.  

의존성? A가 변경될 때 B도 함께 변경될 수 있다.  
즉, 양방향은 관리가 어렵고 논리적으로 서로가 계속 변경합니다.  
(A 변경 -> B 변경 -> A 변경...)  


#### 양방향 연관관계 단점  
데이터베이스에서는 외래 키(FK)를 이용해서 양방향으로 연관관계를 가질 수 있다. 아래의 두 가지 SQL문이 데이터베이스의 테이블은 외래 키(FK) 하나로도 양방향으로 동작할 수 있다는 예시이다.  
```sql
select * from Member m join Team t on m.team_id = t.team_id
select * from Team t join Member m on m.team_id = t.team_id
```

하지만 객체는 그렇지 않다! 단방향 참조만이 가능하다. 그래서, 객체에서도 단방향 연관관계 2개(회원 -> 팀, 팀 -> 회원)를 만들어 양방향 연관관계 를 구현하는 것이다.  

다만 구현에 주의할 점이 몇가지 있다.

기본 엔티티
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
}

...

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

1. 양방향 등록 실수
```java
Team team = new Team("teamA");
em.persist(team);

Member member = new Member("member1");

// 역방향만 연관관계 설정
team.getMembers().add(member);

em.persist(member);
```
이렇게 되면, 어떻게 될까?? 당연히 member 테이블의 team에 대한 외래 키(FK)는 null이 된다. 왜? 연관관계의 주인(Member)에서는 어떠한 작업도 해주지 않았기에 연관관계 설정이 된지 전혀 모르는 것이다.  

2. (순수JPA)영속성 컨텍스트 초기화 안함
```java
  @Test
    public void testEntity() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);

        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);

        // 초기화
//        em.flush();
//        em.clear();

        List<Team> teams = em.createQuery("select t from Team t", Team.class)
                .getResultList();

        for (Team team : teams) {
            System.out.println("team = " + team.getName());
            for (Member member : team.getMembers()) {
                System.out.println(" > member = " + member);
            }
        }

    }
```
엔티티를 영속화시키고 영속성 컨텍스트를 비우지 않았다면, Team엔티티에 대한 정보가 영속성 컨텍스트에 계속 남아있으므로 영속성 컨텍스트 안의 Team 객체에 접근해서 List<Member> 변수로 연관관계를 가지는 Member객체들을 탐색한다.

3. 지연로딩과 즉시로딩(N+1)문제

**즉시로딩에서 N+1**
1. EAGER(즉시 로딩)인 경우1. JPQL에서 만든 SQL을 통해 데이터를 조회
2. 이후 JPA에서 Fetch 전략을 가지고 해당 데이터의 연관 관계인 하위 엔티티들을 추가 조회
3. 2번 과정으로 N + 1 문제 발생

**지연로딩에서의 N+1**
1. JPQL에서 만든 SQL을 통해 데이터를 조회
2. JPA에서 Fetch 전략을 가지지만, 지연 로딩이기 때문에 추가 조회는 하지 않음
3. 하지만, 하위 엔티티를 가지고 작업하게 되면 추가 조회가 발생하기 때문에 결국 N + 1 문제 발생


지연로딩상황

```java
// ========[페치 조인]=========
Team teamA = new Team();
teamA.setName("팀A");
em.persist(teamA);

Team teamB = new Team();
teamB.setName("팀B");
em.persist(teamB);

Member member1 = new Member();
member1.setUsername("회원1");
member1.setTeam(teamA);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("회원2");
member2.setTeam(teamA);
em.persist(member2);

Member member3 = new Member();
member3.setUsername("회원3");
member3.setTeam(teamB);
em.persist(member3);

em.flush();
em.clear();

String query = "select m from Member m";
List<Member> result = em.createQuery(query, Member.class)
        .getResultList();

for (Member member : result) {
    System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
}
```

```sql
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.MEMBER_ID as member_i1_5_,
            member0_.age as age2_5_,
            member0_.TEAM_ID as team_id4_5_,
            member0_.username as username3_5_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.TEAM_ID as team_id1_11_0_,
        team0_.createdBy as createdb2_11_0_,
        team0_.createdDate as createdd3_11_0_,
        team0_.lastModifiedBy as lastmodi4_11_0_,
        team0_.lastModifiedDate as lastmodi5_11_0_,
        team0_.name as name6_11_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
member = 회원1, 팀A
member = 회원2, 팀A
Hibernate: 
    select
        team0_.TEAM_ID as team_id1_11_0_,
        team0_.createdBy as createdb2_11_0_,
        team0_.createdDate as createdd3_11_0_,
        team0_.lastModifiedBy as lastmodi4_11_0_,
        team0_.lastModifiedDate as lastmodi5_11_0_,
        team0_.name as name6_11_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
member = 회원3, 팀B
```
우선 지연로딩 방식으로 구현되어 있기 때문에, Member 엔티티를 조회하더라도 Team 엔티티는 프록시로 조회하게 된다. 그리고 Team 엔티티의 필드에 접근할 때 실제로 SQL문이 나가서 DB에 접근하게 된다.  


* 회원1이 속한 팀인 팀A를 조회하면서 영속성 컨텍스트에 팀A를 저장해둔다.
* 회원2가 속한 팀의 이름에 접근할 때, 영속성 컨텍스트에 있으므로 DB가 아닌 1차 캐시에서 조회된다.
* 회원3이 속한 팀인 팀B는 영속성 컨텍스트에 존재하지 않아서, DB로 쿼리문을 날리게 된다.

결론적으로, 쿼리가 총 3번 나가게 되었다. 이렇게 되면 관련된 엔티티의 데이터 개수만큼 쿼리가 나가서 의도치 않은 성능 저하를 야기할 수 있다.
이 문제가 바로 N+1 문제 이다. N+1 문제를 해결하기 위해 페치 조인을 사용해야 한다.  

```java
// ..
// Member, Team 세팅
// ..

em.flush();
em.clear();

String query = "select m from Member m join fetch m.team";
List<Member> result = em.createQuery(query, Member.class)
        .getResultList();

for (Member member : result) {
    System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
}


tx.commit();
```

```sql
Hibernate: 
    /* select
        m 
    from
        Member m 
    join
        fetch m.team */ select
            member0_.MEMBER_ID as member_i1_5_0_,
            team1_.TEAM_ID as team_id1_11_1_,
            member0_.age as age2_5_0_,
            member0_.TEAM_ID as team_id4_5_0_,
            member0_.username as username3_5_0_,
            team1_.createdBy as createdb2_11_1_,
            team1_.createdDate as createdd3_11_1_,
            team1_.lastModifiedBy as lastmodi4_11_1_,
            team1_.lastModifiedDate as lastmodi5_11_1_,
            team1_.name as name6_11_1_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.TEAM_ID=team1_.TEAM_ID
member = 회원1, 팀A
member = 회원2, 팀A
member = 회원3, 팀B
```

Member 엔티티들을 모두 조회하면서 한방쿼리로 연관된 엔티티인 Team 엔티티까지 조회하게 됐다. 즉, 페치 조인(Fetch join)을 이용해N+1 문제를 해결했다.   

**컬렉션 페치 조인**

일대다 관계에서 컬렉션 페치 조인을 하게 되면 어떻게 될까? 

```java
String query = "select t from Team t join fetch t.members";
List<Team> result = em.createQuery(query, Team.class)
        .getResultList();

for (Team team : result) {
    System.out.println("team = " + team.getName() + "|" + team.getMembers().size());
}


tx.commit();
```
```sql
Hibernate: 
    /* select
        t 
    from
        Team t 
    join
        fetch t.members */ select
            team0_.TEAM_ID as team_id1_11_0_,
            members1_.MEMBER_ID as member_i1_5_1_,
            team0_.createdBy as createdb2_11_0_,
            team0_.createdDate as createdd3_11_0_,
            team0_.lastModifiedBy as lastmodi4_11_0_,
            team0_.lastModifiedDate as lastmodi5_11_0_,
            team0_.name as name6_11_0_,
            members1_.age as age2_5_1_,
            members1_.TEAM_ID as team_id4_5_1_,
            members1_.username as username3_5_1_,
            members1_.TEAM_ID as team_id4_5_0__,
            members1_.MEMBER_ID as member_i1_5_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.TEAM_ID=members1_.TEAM_ID
team = 팀A|2
team = 팀A|2
team = 팀B|1
```
문제가 하나 있다! 왜 팀A가 2번 조회되지??  
이것이 컬렉션 페치 조인에서 주의해야할 점이다! 일대다 조인은 뻥튀기(?)되는 문제가 발생할 수 있다. 즉, 같은 데이터가 중복 조회되는 문제가 있으니 주의해야한다.  

> 일대다 조인에서의 중복 조회
> 아래의 그림처럼 조회되는 것이다. 실제 팀A 데이터는 하나지만, 조회할 때 Member 엔티티와 조인되므로 중복 조회되는 문제가 발생하는 것이다. 실제 DB에서의 조인 실행 결과는 "[TEAM JOIN MEMBER]"와 같다.
> 즉 Member 의 데이터가 다르게 조회됨으로써 다른 로우가 되지만, JPA에서 전체 필드가 아닌 부분적으로 조회한 결과는 같은 결과를 가지는 로우가 2개가 돼서, 중복 조회되는 것이다.
> <img width="580" alt="1" src="https://github.com/witwint/TIL/assets/108222981/b38f23d7-ee84-403e-8975-b0f1487c8c66">

**페치 조인과 DISTINCT**

중복조회되는 문제를 해결하려면, DISTINCT 명령어를 활용해서 중복 로우를 제거해주면 된다.   
<img width="508" alt="1" src="https://github.com/witwint/TIL/assets/108222981/f30de9eb-bf23-4e8d-a119-a4f93aa1ed12">

하지만 위의 그림처럼 SQL을 실행했을 때, 서로가 다른 결과라고 하면 DISTINCT 명령어를 써도 제거되지 않는다.  
즉, SQL에 DISTINCT 를 추가해도, 데이터가 다르므로 SQL 결과에서 중복 조회를 제거하는 데에 실패한다.  

그래서, JPA에서는 DISTINCT가 DB에서 뿐만 아니라, 애플리케이션 레벨에서도 중복 제거를 시도한다.  
즉, 같은 식별자를 가진 Team 엔티티를 삭제한다.   

```java
String query = "select distinct t from Team t join fetch t.members";
List<Team> result = em.createQuery(query, Team.class)
        .getResultList();

for (Team team : result) {
    System.out.println("team = " + team.getName() + "|" + team.getMembers().size());
}
```
```sql
Hibernate: 
    /* select
        distinct t 
    from
        Team t 
    join
        fetch t.members */ select
            distinct team0_.TEAM_ID as team_id1_11_0_,
            members1_.MEMBER_ID as member_i1_5_1_,
            team0_.createdBy as createdb2_11_0_,
            team0_.createdDate as createdd3_11_0_,
            team0_.lastModifiedBy as lastmodi4_11_0_,
            team0_.lastModifiedDate as lastmodi5_11_0_,
            team0_.name as name6_11_0_,
            members1_.age as age2_5_1_,
            members1_.TEAM_ID as team_id4_5_1_,
            members1_.username as username3_5_1_,
            members1_.TEAM_ID as team_id4_5_0__,
            members1_.MEMBER_ID as member_i1_5_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.TEAM_ID=members1_.TEAM_ID
team = 팀A|2
team = 팀B|1
```
정리하면, JPQL에서의 DISTINCT 명령어는 다음의 2가지 기능을 가진다.

* SQL에 DISTINCT 추가
* 애플리케이션에서 엔티티 중복 제거

**페치 조인과 일반 조인의 차이**

일반 조인을 실행할 때에는, 연관된 엔티티를 함께 조회하지 않는다. 하지만, 페치 조인을 실행하면 연관된 엔티티도 조회하게 된다.  

무슨 말이냐하면, 페치 조인을 하게되면 연관된 엔티티의 로우에 대한 정보도 가져오게 되지만, 일반 조인을 하게 되면 select 절에 명시한 테이블 혹은 로우에 대한 정보만 가져오게 된다. 아래의 예시는 페치 조인과 일반 조인의 sql문이다. 차이를 확인해보시기 바랍니다.  

```sql
[일반 조인]
select m from Member m join m.team

[SQL문]
select m.id, m.name, m.email from Member m
inner join Team t on m.team_id = t.id

[페치조인]
select m from Member m join fetch m.team

[SQL문]
select m.id, m.name, m.email, t.id, t.name from Member m
inner join Team t on m.team_id = t.id
```

select 절에서 가져오는 데이터가 다르다! 그렇기에 페치 조인을 활용하게 되면 지연 로딩으로 연관관계가 설정되어 있어도, 일반 조인과는 달리 다대일 관계 혹은 일대일 관계의 객체가 초기화될 수 있는 것이다!  

**페치 조인과 JPQL**

JPQL은 결과를 반환할 때 연관관계를 고려하지 않는다. 단지 Select 절에 지정된 엔티티만 조회할 뿐이다. 바로 위의 예제에서 일반 조인으로 실행하게 되면 Team 엔티티만 조회하게 되고, Member 엔티티는 조회하지 않는다.  

다만, Fetch join 을 할 때에만 연관된 엔티티도 함께 조회한다(즉시 로딩). 페치 조인은 객체 그래프를 SQL 한 번에 조회하는 개념이다.   

**둘 이상의 컬렉션은 페치 조인할 수 없다**

Team 엔티티에 컬렉션 타입의 변수가 하나 더 있다고 가정할 때, 둘 이상의 컬렉션에 대해서도 페치 조인을 하게 되면 일대다대다 관계가 되므로 문제가 생길 수 있다.  
컬렉션에 대해 페치 조인은 딱 하나만 지정할 수 있다.  

**컬렉션을 페치 조인하면 페이징API(setFirstResult, setMaxResults) 를 사용할 수 없다**
일대일, 다대일과 같은 단일 값 연관관계에서는 페치 조인해도 페이징이 가능하다.  
일대다 관계에서는 중복 조회되는 문제(aka 뻥튀기?)가 있어서 이걸 페이징 처리하게 되면 의도한 결과가 나오지 않을 수 있다.  

* 중복 조회되는 문제를 DB단에서만 처리하는 것이 아니라, 어플리케이션 단에서도 처리하기 때문이다.
* 중복 조회된 결과가 2개일 때 페이지 사이즈가 1이라면, 중복 조회되는 문제가 발생한다.

#### OneToOne의 단점

OneToOne예시
```java

@Entity(name = "Team")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "team_id")
    private Long id;

    private String teamName;
}


@Entity(name = "Member")
@Getter
@NoArgsConstructor
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "member_id")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

** OneToOne 은 즉시로딩인가 지연로딩인가**

* @OneToOne 은 즉시 로딩이다.
* FetchType을 LAZY로 변경해서 쓰기를 추천한다
* 단방향 @OneToOne 관계에서는 지연로딩 적용에 문제가 없다
* 양방향 @OneToOne 관계에서는 FetchType을 Lazy로 설정하더라도 Eager로 동작하는 경우가 있다.
* 정확하게 말하면 @OneToOne 양방향 연관 관계에서 연관 관계의 주인이 아닌 쪽 엔티티를 조회할 때, Lazy로 동작할 수 없다.

간략한 이유 : 아에 반대쪽의 존재를 알 수 있는 근거가 하나도 없어서 null로 표기가 되면서 지연로딩의 프록시가 null을 받게되는데 이는 오류가나게된다. `@OneToMany`는 Many 를 컬렉션으로 관리하기 때문에 null 을 표현할 방법이 있다(size) 즉 size = 0 이런값이라도 있다.  

**@OneToOne 지연로딩 문제 해결책**


* ptional=false 설정을 통해 무조건 Lazy Loading 을 시킨다. 이를 통해 반드시 연관 시 값이 존재함이 보장되어야 한다.
	* PrimaryKeyJoin 의 경우에는 optional=false 일 경우에 데이터 저장 순서가 꼬여버린다. 정상적인 optional=false가 작동하려면 ForeignKey Join을 해야한다.
* fetch join 으로 같이 가져와 해결 가능
* 억지로 엔티티에 FK 롤 집어 넣어 해결
* @ElementCollection 을 통해 컬렉션을 사용하고 unique 조건으로 데이터가 오직 1개만 들어가게 만든다.
* OneToOne -> OneToMany + ManyToOne 분리 방법(좋은 방법은 아닌거 같다)


#### QueryDSL
Spring Data Jpa를 써보신 분들은 아시겠지만, 기본으로 제공해주는 @Query로는 다양한 조회 기능을 사용하기에 한계가 있습니다.   
그래서 이 문제를 해결하기 위해 정적 타입을 지원하는 조회 프레임워크를 사용하는데요.  
Querydsl은 Jooq와 함게 가장 유명한 조회 프레임워크입니다.  


**동적쿼리에서의 장점예시**
**JPQL**
```java
  @Query("select c from Consultation c join fetch c.hospital join fetch "
        + "c.patient where c.idfConsultation = :id")
  Optional<Consultation> findByIdWithHospital(@Param("id") long id);
```

**QueryDSL**
```java
@Override
public Optional<Consultation> findByIdWithHospital(long id) {
  return Optional.ofNullable(
      queryFactory
          .selectFrom(consultation)
          .join(consultation.hospital).fetchJoin()
          .where(consultation.idfConsultation.eq(id))
          .fetchOne()
  );
}
```

간단 사용 예시
각종 빌드설정을 해준후(상세한 내용은 참조블로그 참고)
```
@Repository
public class AcademyRepositorySupport extends QuerydslRepositorySupport {
    private final JPAQueryFactory queryFactory;

    public AcademyRepositorySupport(JPAQueryFactory queryFactory) {
        super(Academy.class);
        this.queryFactory = queryFactory;
    }

    public List<Academy> findByName(String name) {
        return queryFactory
                .selectFrom(academy)
                .where(academy.name.eq(name))
                .fetch();
    }

}
```
이런식으로 `return queryFactory.selectFrom(academy).where(academy.name.eq(name)).fetch();`등의 sql과 유사한 내용을 자바코드로 사용할 수 있게됩니다.  
* 문자가 아닌 코드로 쿼리를 작성할 수 있어 컴파일 시점에 문법 오류를 확인할 수 있다.
* 인텔리제이와 같은 IDE의 자동 완성 기능의 도움을 받을 수 있다.
* 복잡한 쿼리나 동적 쿼리 작성이 편리하다.
* 쿼리 작성 시 제약 조건 등을 메서드 추출을 통해 재사용할 수 있다.
* JPQL 문법과 유사한 형태로 작성할 수 있어 쉽게 적응할 수 있다.

더불어서 보통 `QueryDSL`을 사용하면 

```java
@Autowired
    private AcademyRepository academyRepository; //기존 data JPA

    @Autowired
    private AcademyRepositorySupport academyRepositorySupport; //QueryDSL용 클래스
```
이렇게 두개를 사용해야하는데 

<img width="1401" alt="1" src="https://github.com/witwint/TIL/assets/108222981/3f6fb934-527f-45d1-a64d-c86dfa5dd0ff">
이러한 방식으로 하나의 의존관계 주입으로도 둘다 사용할 수 있게 Spring Data Jpa가 지원해줍니다. (상세한 내용은 참조블로그 참고)  

더불어서 `@QueryProjection`등 DB연결후 도메인객체를 뽑을때 바로 엔티티 자체가 아니라 설정 DTO로 변환해서 바로 가져올 수 있는 기능도 있습니다.

```java
@Data
@NoArgsConstructor
public class MemberDTO {
	
    private String username;
    private int age;
    
    @QueryProjection
    public MemberDTO(String username, int age) {
    	
        this.username = username;
        this.age = age;
    }
}


List<MemberDto> memberDtos = queryFactory

	.select(new MemberDto(member.username, member.age))
        .from(member)
        .fetch();
```
**장점**
* 다른 방식들과 달리, 이 방법은 컴파일시점에 필드 타입체크 등이 가능하기 때문에 안정적으로 코드를 작성할 수 있다.  

**단점**
* DTO의 경우 서비스, API 계층 등 여러 곳에서 사용될 수 있는데, DTO가 queryDsl에 의존하게 되면서 의존관계가 꼬일 수 있기 때문에, 상황에 따라 알맞게 사용해야 한다.

우리프로젝트의 경우 리턴Respose

**QueryDSL의 다양한 기능**
https://velog.io/@bagt/QueryDsl-DTO-Projection  


#### 그래서 어떻할 것이냐?

JPA와 QueryDSL을 알아보면서 유의할점을 컴펙트하게 모아보겠습니다.  
* One To Many 에서는 사용할거면 양방향 매핑을 해주는것이 좋습니다.
* One To Many 양방향 매핑은 @JoinColumn을 사용하는것이 좋습니다.
* 양방향 매핑은 순환참조를 주의해야합니다.
* 양방향 매핑에서 영속성 컨텍스트와 실제 DB의 불일치를 주의해야합니다.
* 주로 지연로딩을 사용하더라도 상황에따라 페치 조인을 사용하며 쿼리 효율을 높여줘야합니다.
* 페치조인을 사용할때 실제 조인과 어떻게 다른지 알아야합니다.
* JAP와, QueryDSL의 페치조인사용법도 익혀야합니다.
* OneToOne의 특성 지연로딩에서의 프록시관계로인한 한계를 이해해야합니다.
* 일단 JPQL과 QueryDSL의 사용법을 알아야합니다...
* ...
* ...
추가요소 기타 등등.....

개발자는 편하게 사용하고 싶지만 모든게 다 SQL 쿼리의 효율때문에 일어난 일 입니다. 편하기 사용하기 위함인데 점점 추상화 되면서 근본적인 sql이 어떻게 날라가는지 예상하기 어렵습니다.  
물론 그렇다고 무조건 JDBC JdbcTempalte mybatis 옳냐 하면 사실 이런것들이 불편하기에 나온것이 JPA입니다.  
그래서 정말 JDBC JdbcTempalte mybatis등 SQL중심으로 어떤 추상화 레벨에서 사용할것이냐? JPA, data JPA, QuertDSL 등을 어느 수준의 범위까지 쓸것이냐? 이런것들 모두 팀과 개인의 선택인거 같습니다.  










https://dublin-java.tistory.com/51  
https://velog.io/@ddangle/%EC%96%91%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84%EC%9D%98-%ED%97%88%EC%A0%90  
https://velog.io/@ddangle/%ED%8E%98%EC%B9%98-%EC%A1%B0%EC%9D%B8Fetch-join  
https://dev-coco.tistory.com/165  
https://velog.io/@yhlee9753/OneToOne-%EA%B4%80%EA%B3%84%EB%8A%94-%EA%B3%BC%EC%97%B0-%EC%A7%80%EC%97%B0%EB%A1%9C%EB%94%A9%EC%9D%B4-%EB%90%98%EB%8A%94%EA%B0%80  
https://jojoldu.tistory.com/372  
https://ittrue.tistory.com/292  
https://green-bin.tistory.com/24  
https://coding-business.tistory.com/100  
https://www.reddit.com/r/java/comments/w4abyg/is_there_a_reason_to_not_use_spring_data_jpa_and/?rdt=43738  

---
## 20240414
### owasp 22년 10대 취약점

#### 새로운 상위 10대 취약점 목록에 대한 주요 세부 정보

OWASP는 CWE의 근본 원인에 초점을 맞춰 새로운 리스트를 만들었습니다. 업데이트된 리스트는 회사가 언어/프레임워크에 적용할 수 있는 CWE에 집중할 수 있기에 교육에 유용합니다. 리스트에 포함된 10개 중 8개 카테고리는 수집된 데이터를 기반으로 만들어졌습니다. 나머지 두 카테고리는 상위 10개 커뮤니티 조사를 기반으로 구성되었습니다.

2022년 OWASP 상위 10대 취약점은 명칭 변경, 범위 지정, 그리고 통합 진행의 결과입니다. 해당 취약점을 하나씩 살펴보고 이러한 취약점을 해결하기 위한 해결책을 알아보겠습니다.

**1. 잘못된 접근 제어**

잘못된 접근 제어는 공격자가 사용자 계정에 대한 접근권한을 획득하며 발생하는 문제입니다. 공격자는 사용자 혹은 관리자로 시스템에서 활동하며 승인되지 않은 데이터 및 민감한 파일에 대한 권한을 획득합니다. 잘못된 접근 제어 결함은 해커가 사용자 권한 설정을 변경하는데 도움을 줄 수 있습니다. 관리자 패널, 웹사이트 컨트롤 패널, FTP / SFTP / SSH를 통한 접근, 그리고 데이터베이스 접근 등이 잘못된 접근 제어의 예시입니다.

이러한 취약점은 다음과 같은 방법으로 해결할 수 있습니다:

* 교차 사이트 위조 또는 안전하지 않은 민감한 데이터 저장을 탐색하는 인터랙티브 애플리케이션 보안 테스트 솔루션 구현
* IAST 활동 보완을 위한 침투 테스트 수행
* 비활성 계정 삭제
* 정기적인 검사 및 접근 제어 테스트
* 적절한 세션 관리 방법 사용

**2. 암호화 실패**

암호화 실패는 저장되거나 전송된 데이터가 어떠한 방식으로든 손상될 때 발생합니다.

암호화 실패의 결과로 신용카드 사기 또는 신원 도용이 종종 발생합니다. 암호화 실패는 데이터가 일반 텍스트나 오래된 알고리즘을 사용하여 전송될 때 발생합니다. 안전하지 않은 키 관리 및 로테이션 기술도 암호화 실패의 원인이 될 수 있습니다.

이 취약점을 해결하기 위한 방법은 아래와 같습니다:

* 데이터 수집 문서의 자동완성 기능 해제
* 데이터 노출 영역 줄이기
* 데이터가 전송 중이거나 저장되어 있을 때 데이터 암호화 사용
* 최고 수준의 암호화 사용
* 데이터 수집 양식에 대한 캐싱 비활성화
* 암호 저장 시 필수 해시 함수 활용

**3. 인젝션**

인젝션 취약점은 SQL, OS, NoSQL 혹은 LDAP 인젝션을 통해 해로운 데이터를 인터프리터에 삽입하는 것을 말합니다. 인젝션 공격은 인터프리터를 속여 애플리케이션이 의도하지 않은 명령을 생성하거나 기존에 설계되지 않은 동작을 하도록 합니다. 파라미터를 입력으로 받은 애플리케이션은 인젝션 공격에 취약합니다. 인젝션 공격을 방지하기 위해 다음 접근 방식을 사용할 수 있습니다.

* CI/CD 파이프라인에 SAST 및 IAST 도구 포함
* 공격에 노출된 불필요한 명령 수행 방지를 위해 데이터에서 명령을 분리
* 매개변수화된 쿼리 사용
* 제거기 대신 안전한 API 사용
* 서버사이드 검증 및 침입 탐지 시스템을 사용하여 의심스러운 클라이언트사이드 동작 식별

**4. 불안전한 디자인**

불안전한 디자인은 잘못된 제어 설계와 관련된 모든 결함을 나타냅니다. 이 카테고리는 위협 모델링, 보안 디자인 패턴 및 참조 아키텍처를 다룹니다.

불안전한 디자인의 해결책:

* 안전한 개발 라이프사이클 사용
* 즉시 사용 가능한 안전 디자인 패턴 라이브러리 생성
* 애플리케이션의 각 수준에 타당성 검사 통합
* 중요 인증, 접근 제어, 비즈니스 로직 및 키 흐름에 대한 위협 모델링 구축
* 용자 및 서비스 리소스 사용 제한

**5. 보안 설정 오류**

보안 설정 오류는 상위 10개 취약점 중 가장 흔한 취약점입니다. 안전하지 않은 기본 설정 승인, 불완전한 설정, 민감한 정보를 담고 있는 장황한 에러 메세지, 그리고 잘못 설정된 HTTP 리더기는 보안 설정 오류의 원인이 될 수 있습니다.

보안 설정 오류를 위한 해결책:

* 조직의 보안 정책에 부합하는 템플릿 사용
* 위험 감소를 위해 세분화된 애플리케이션 아키텍처 사용
* 사용하지 않는 기능 및 서비스 제거
* 클라우드 리소스, 서버 및 애플리케이션을 지속적으로 모니터링하여 설정 오류를 탐지

**6. 취약하고 오래된 구성요소**

오픈 소스 구성요소에는 애플리케이션 보안에 심각한 위협이 되는 취약점이 포함될 수 있습니다. 취약한 구성요소는 종종 데이터 유출의 원인이 됩니다.

취약하고 오래된 구성요소로 인한 위험을 최소화하기 위한 해결책:

* 회사 프레임워크의 일부인 구성요소는 설정 관리 아래에 두기
* 스캐너는 모니터링해야 하는 모든 구성요소를 식별할 수 있도록 하기
* 패치에 관련한 운영 위험을 줄이기 위해 패치 관리 업무 흐름을 자동화
* 위협 인텔리전스 데이터가 풍부한 취약성 데이터베이스에 대한 스캔 수행

**7. 식별 및 인증 실패**

공격자는 애플리케이션이 세션 관리 혹은 사용자 인증과 관련된 기능을 잘못 실행할 때 암호, 세션 토큰 또는 보안 키를 손상시킵니다. 이로 인해 사용자 ID가 도용될 수 있으며 식별 및 인증 실패는 같은 네트워크 내의 다른 자산을 위험에 빠뜨릴 수 있습니다.

식별 및 인증 실패에 대한 해결책:

* 다단계 인증 사용
* 관리자 권한이 있는 사용자는 기본 비밀번호 사용하지 않기
* 모든 로그인 시도 실패 검토
* 안전한 세션 관리자를 사용하고 URL에 세션 ID를 넣지 않기

**8. 소프트웨어 및 데이터 무결성 실패**

소프트웨어 및 데이터 무결성 실패는 무결성 위반으로부터 코드와 인프라스트럭처를 보호할 수 없을 때 발생합니다. 악성 코드 및 무단 접근은 이런 취약점의 위험 요소입니다. 신뢰할 수 없는 소스의 플러그인, 라이브러리, 혹은 모듈을 포함하는 프로그램은 무결성 실패에 취약합니다. 자동 업데이트 기능을 사용하면 필요한 무결성 검사 없이 업데이트가 됩니다.

소프트웨어 및 데이터 무결성 실패에 대한 해결책:

* 프로그램이 변조되지 않도록 디지털 서명 구현
* 코드 및 설정 수정에 대한 검토 절차 구현
* 라이브러리와 종속성이 신뢰할 수 있는 저장소를 사용하고 있는지 확인
* CI/CD 파이프라인에 설정 및 접근 제어가 포함되어 있는지 확인
* 무결성 검사 없이 암호화되지 않은 데이터를 신뢰할 수 없는 클라이언트에 전달하지 않기

**9. 보안 기록 및 모니터링 실패**

보안 기록 및 모니터링 실패는 애플리케이션이 공격에 취약하게 만듭니다. 로그인, 그리고 실패한 로그인이 기록되거나 모니터링되지 않는다면 취약한 애플리케이션이 될 수 있습니다.

보안 기록 및 모니터링 실패에 대한 해결책:

* 침투 테스트를 통해 테스트 로그를 연구하고, 가능한 결함을 감지
* 로그 관리 솔루션이 쉽게 관리 가능한 포맷으로 로그 생성
* 높은 가치의 트랜잭션이 변조 방지를 위한 검사추적을 보유하고 있는지 확인
* 의심스러운 활동을 감지하기 위해 알림과 모니터링 매커니즘 구현
* 모든 로그 데이터를 올바르게 인코딩

**10. 서버사이드 요청 위조**

이는 애플리케이션이 사용자 공급 URL의 유효성을 확인하지 않고 원격 리소스를 가져오면서 나오는 결과입니다. 최근 아키텍처가 복잡해지고, 클라우드 서비스 사용이 증가하면서 서버 사이드 요청 위조가 발생하고 있습니다

서버사이드 요청 위조를 위한 해결책:

* 방화벽 정책을 ‘기본적으로 거부’로 설정
* 애플리케이션에 따라 방화벽 규칙에 대한 소유권 및 라이프사이클 설정
* 방화벽에서 허용되거나 차단된 모든 네트워크 흐름 기록
* 클라이언트 공급 입력 데이터 삭제
* 일관된 URL 사용

#### 참고자료
https://www.appsealing.com/kr/2022-owasp-10%EB%8C%80-%EC%B7%A8%EC%95%BD%EC%A0%90/  

---
## 20240415
### 프록시 AOP 실무주의사항 문제2와 해결법

#### 프록시 기술과 한계 - 타입 캐스팅
JDK 동적 프록시 한계
인터페이스 기반으로 프록시를 생성하는 JDK 동적 프록시는 구체 클래스로 타입 캐스팅이 불가능한 한계가 있다. 어떤 한계인지 코드를 통해서 알아보자  
```java
@Slf4j
@SpringBootTest(properties = {"spring.aop.proxy-target-class=false"}) //JDK 동적
프록시, DI 예외 발생
//@SpringBootTest(properties = {"spring.aop.proxy-target-class=true"}) //CGLIB 프
록시, 성공
@Import(ProxyDIAspect.class)
public class ProxyDITest {
 @Autowired MemberService memberService; //JDK 동적 프록시 OK, CGLIB OK
 @Autowired MemberServiceImpl memberServiceImpl; //JDK 동적 프록시 X, CGLIB OK
 @Test
 void go() {
 log.info("memberService class={}", memberService.getClass());
 log.info("memberServiceImpl class={}", memberServiceImpl.getClass());
 memberServiceImpl.hello("hello");
 }
}
```
`@Autowired MemberServiceImpl memberServiceImpl;`이부분에서 JDK 동적 프록시를 사용하면 상위 인터페이스 기반으로 만들어진 프록시 이기에 실제 클래스`MemberServiceImpl`타입으로 받을 수 없다.  

물론 만약 인터페이스가 존재한다면 다형성을 위해서도 대부분의 작업을 ` @Autowired MemberService memberService;`이런식으로 인터페이스로 받긴해서 문제가 없긴한데  
간혹 실제클래스를 받아야하거나 테스트코드등에서 너무 불편한경우가 있다.  

그럼 CGLIB만 쓰면 되지 않을까??

#### CGLIB 단점(이였던것)

CGLIB 구체 클래스 기반 프록시 문제점
* 대상 클래스에 기본 생성자 필수
* 생성자 2번 호출 문제 (실제 진짜 내가 코드 쓴 객체 생성될때 + 프록시만들면서 부모생성자 (super)로 한번더 )
* final 키워드 클래스, 메서드 사용 불가 (사실 실제 서버환경에서는 AOP적용 대상에 final안씀)

이런 문제가 있었지만  
대상 클래스에 기본 생성자 필수 - 스프링 4.0부터 objenesis 라는 특별한 라이브러리를 사용해서 기본 생성자 없이 객체 생성이 가능하다.  
생성자 2번 호출 문제 - 위 해결법으로 동시에 이것도 해결  

이런방식으로 CGLIB를 개선하며 스프링부트 2.0부터는 아에 무조건 CGLIB가 기본 프록시방법으로 고정되었다.  

---

## 20240416
### 프로그래머스 게임맵 최단거리 BFSDFS 2단계

ROR 게임은 두 팀으로 나누어서 진행하며, 상대 팀 진영을 먼저 파괴하면 이기는 게임입니다. 따라서, 각 팀은 상대 팀 진영에 최대한 빨리 도착하는 것이 유리합니다.  지금부터 당신은 한 팀의 팀원이 되어 게임을 진행하려고 합니다. 다음은 5 x 5 크기의 맵에, 당신의 캐릭터가 (행: 1, 열: 1) 위치에 있고, 상대 팀 진영은 (행: 5, 열: 5) 위치에 있는 경우의 예시입니다.  

위 그림에서 검은색 부분은 벽으로 막혀있어 갈 수 없는 길이며, 흰색 부분은 갈 수 있는 길입니다. 캐릭터가 움직일 때는 동, 서, 남, 북 방향으로 한 칸씩 이동하며, 게임 맵을 벗어난 길은 갈 수 없습니다.  
아래 예시는 캐릭터가 상대 팀 진영으로 가는 두 가지 방법을 나타내고 있습니다.  
위 예시에서는 첫 번째 방법보다 더 빠르게 상대팀 진영에 도착하는 방법은 없으므로, 이 방법이 상대 팀 진영으로 가는 가장 빠른 방법입니다.  만약, 상대 팀이 자신의 팀 진영 주위에 벽을 세워두었다면 상대 팀 진영에 도착하지 못할 수도 있습니다. 예를 들어, 다음과 같은 경우에 당신의 캐릭터는 상대 팀 진영에 도착할 수 없습니다.  
게임 맵의 상태 maps가 매개변수로 주어질 때, 캐릭터가 상대 팀 진영에 도착하기 위해서 지나가야 하는 칸의 개수의 최솟값을 return 하도록 solution 함수를 완성해주세요. 단, 상대 팀 진영에 도착할 수 없을 때는 -1을 return 해주세요.  

제한사항

* maps는 n x m 크기의 게임 맵의 상태가 들어있는 2차원 배열로, n과 m은 각각 1 이상 100 이하의 자연수입니다.
* n과 m은 서로 같을 수도, 다를 수도 있지만, n과 m이 모두 1인 경우는 입력으로 주어지지 않습니다.
* maps는 0과 1로만 이루어져 있으며, 0은 벽이 있는 자리, 1은 벽이 없는 자리를 나타냅니다.
* 처음에 캐릭터는 게임 맵의 좌측 상단인 (1, 1) 위치에 있으며, 상대방 진영은 게임 맵의 우측 하단인 (n, m) 위치에 있습니다.


예시 {{1,0,1,1,1},{1,0,1,0,1},{1,0,1,1,1},{1,1,1,0,1},{0,0,0,0,1}}; = 11  

코드
```java
import java.util.*;

class Solution {
    public int solution(int[][] maps) {
        int answer = 0;
        int n = maps.length;
        int m = maps[0].length;
        
        // 이동 가능한 방향 (상,하,우,좌)
        int[][] directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0, 0, 1}); // 시작 위치 (0, 0)에서 출발하므로 큐에 추가

        while (!queue.isEmpty()) {
            int[] currentPosition = queue.poll();
            int x = currentPosition[0];
            int y = currentPosition[1];
            int distance = currentPosition[2];

       	    // 목적지에 도달하면 최단 거리를 반환후 종료
            // 도착 가능한 여러 방안이 있더라도, 최단 거리 방안이 제일 먼저 도착
            if (x == n - 1 && y == m - 1) { 
                answer = distance;
                break;
            }

            for (int[] dir : directions) {
                int newX = x + dir[0];
                int newY = y + dir[1];

                // 이동 가능한 위치이면 큐에 추가하고 해당 위치를 벽으로 표시 (방문 처리)
                if (newX >= 0 && newX < n && 
                    newY >= 0 && newY < m && 
                    maps[newX][newY] == 1) {
                    queue.offer(new int[]{newX, newY, distance + 1});
                    maps[newX][newY] = 0;
                }
            }
        }
        
        if (answer == 0) {
            // 목적지에 도달할 수 없는 경우 -1 반환
            return -1;
        }
        
        return answer;
    }
}
```
저번에도 풀다 말은거같은데 이번에 그냥 답도 보면서 전체적인 구조를 학습했다. `int[][] directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};`이런식으로 이동방향을 기입해두고  
돌려서 쓰는거도 전혀 몰랐는데 좀 전체적인 구조를 이해하고 비슷한 문제를 풀어봐야할거같다.  

---
## 20240417
### 루시카토 OX 자료구조 10문제  

* 이진탐색 트리(BST)의 평균 시간 복잡도는 O(logn)이며 최악의 시간 복잡도는 O(n)이다
답 o
![1](https://github.com/witwint/TIL/assets/108222981/2900d9be-7dee-4130-afcc-1767f537368d)
![1](https://github.com/witwint/TIL/assets/108222981/40a3d58b-b174-4cd3-9a41-adc4feb3427d)
자주쓰는 시간 복잡도 평균과 최악의 시간 복잡도를 고려하여 사용해야 합니다.

* 자바에서 HashSet이 HashMap보다 성능이 좋다.
답 x
![1](https://github.com/witwint/TIL/assets/108222981/6f39c929-c2ee-4f89-a7e9-87ffcd280544)
HashSet 내부에서 HashMap을 선언하여 사용하기에 차이가 없다.

* 자바 ArrayList의 contains 시간복잡도는 O(n)이다.
답o
![1](https://github.com/witwint/TIL/assets/108222981/6e2e9356-f358-4382-a8c9-91ad9c54d171)
![1](https://github.com/witwint/TIL/assets/108222981/7d1d42e9-0b35-4613-a511-bb71eef8680e)
실제로 그냥 인덱스를 돌려서 값이 같은걸 찾고 리턴해준다.

* 자바에서 HashMap과 달리 HashTable의 특징은 Thread-safe한것이다.
답 o
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    public synchronized int size() { }

    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) { }

    public synchronized V put(K key, V value) { }
}
```
클래스를 확인해보면 `synchronized`통해 Thread-safe하게 사용할 수 있습니다. 다만 병목현상때문에 현제는 거의 사용하지 않습니다.  
자바에서 지원하느 Synchronized 키워드는 여러개의 스레드가 한개의 자원을 사용하고자 할 때, 현재 데이터를 사용하고 있는 해당 스레드를 제외하고 나머지 스레드들은 데이터에 접근 할 수 없도록 막는 개념입니다.  

* 자바의 ConcurrentHashMap은 검색 속도 향상을 위해 나온 클래스 이다.
답 X
Hashtable 클래스의 단점을 보완하면서 Multi-Thread 환경에서 사용할 수 있도록 나온 클래스가 바로 ConcurrentHashMap 입니다.
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    public V get(Object key) {}

    public boolean containsKey(Object key) { }

    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) ==
.
.
.
```
synchronized 키워드가 메소드 전체에 붙어 있지 않습니다. get() 메소드에는 아예 synchronized가 존재하지 않고, put() 메소드에는 중간에 synchronized 키워드가 존재하는 것을 볼 수 있습니다. 이것을 좀 더 정리해보면 ConcurrentHashMap은 읽기 작업에는 여러 쓰레드가 동시에 읽을 수 있지만, 쓰기 작업에는 특정 세그먼트 or 버킷에 대한 Lock을 사용한다는 것입니다.  
(https://velog.io/@kimunche/ConcurrentHashMap) 참고자료
![1](https://github.com/witwint/TIL/assets/108222981/29410bcc-f317-49b8-8506-ec865824aed3)
![2](https://github.com/witwint/TIL/assets/108222981/4041ba56-067e-4a05-9359-8377ca8d4378)

* 자바에서 map을 사용할 때 입력 순서를 보장할 수 있는 방법은 없다.
답 x
`LinkedHashMap`이 있습니다. LinkedHashMap은 HashMap을 상속하여 만들어진 클래스이고 따라서 HashMap의 특징을 가지고 습니다. 다른 점이라고 한다면, Node 객체를 Entry 객체로 감싸서 저장된 키의 순서를 보존하고 있다는 점입니다. 동기화가 되어있지 않기 때문에 멀티스레드 환경에서 사용할 때는 주의가 필요합니다.  

* 레드 블랙트리는 자기 균형 이진 탐색 트리이다.
답 o
레드블랙트리는 최대힙 최소힙이런 트리가 정리를 하면서 데이터를 정리헤 저장하는거처럼 좌우 방향을 기준으로 자신의 왼쪽 서브 트리에는 현재 노드보다 값이 작은 것, 오른쪽 서브 트리에는 값이 큰 것들만 가질 수 있게 만든 트리입니다.
![`](https://github.com/witwint/TIL/assets/108222981/1db141a6-cff5-4493-aee2-6210108d4721)
자세한 내용 (https://jwdeveloper.tistory.com/280)

* 자바의 HashMap은 해쉬 충돌이 생겼을때 개방 주소법(Open Addressing)을 사용한다.
답 x

* 자바의 HashMap버킷 크기는 고정이다.
답 x

* 자바의 HashMap은 해시 버킷 체이닝에 LinkedList만 사용한다.
답 x
자바의 HashMap은 체이닝 방법을 사용해서 해쉬 충돌을 해결한다.
특이점이 2가지 있는데 우선 버킷의 경우 해시 버킷 개수의 기본 값은 16이고, 데이터의 개수가 임계점에 이를 때마다 해시 버킷 개수의 크기를 두 배씩 증가시킨다. 그리고 임계점은 지금 버킷 개수의 3/4이다.
```java
	

//8개 이상의 키-값 쌍이 모이면 트리로 변경
static final int TREEIFY_THRESHOLD = 8;

//6개 이하로 떨어지면 링크드 리스트로 변경
static final int UNTREEIFY_THRESHOLD = 6;

```
해쉬 충돌 체이닝의 경우 버킷에 체이닝 된 데이터의 수에 따라 링크드 리스트와 트리를 변경해가며 사용한다. 트리는 레드블랙 트리를 사용하고 링크드 리스트와 트리를 변경하는 기준은 위와 같이 설정되어 있다.  



1. HashMap은 해싱함수를 통해 인덱스를 산출한다.

2. HashMap은 인덱스를 통한 접근으로 시간 복잡도 O(1)의 빠른 성능을 자랑한다.

3. key는 무한하지만 인덱스는 한정되어 있어 충돌은 불가피하다.

4. 충돌을 줄이기 위해 HashMap은 버킷의 사이즈를 조절한다.

5. 충돌이 일어날 시, 충돌 수가 적으면 LinkedList 방식으로 충돌된 객체들을 관리하다가, 임계점을 넘으면 Red-Black Tree 방식으로 객체들을 저장한다.

6. 시간 복잡도는 Linked List가 O(n), Red-Black Tree가 O(log n)이다.
![1](https://github.com/witwint/TIL/assets/108222981/6cc7d837-5a28-4dd4-b78a-754ddb6f88c1)

![1](https://github.com/witwint/TIL/assets/108222981/0dbc316f-e1c5-45d7-b473-703e7efa003b)

---
## 20240418
### 스프링 카프카  

#### 인트로
기존 데이터 시스템의 구조는 각 애플리케이션과 데이터베이스가 end-to-end로 직접 연결되어 있었습니다. 이러한 구조는 간단하지만 각각의 데이터 파이프라인이 분리되어 있어, 요구사항이 증가함에 따라 시스템의 복잡도를 높이는 결과를 가져왔고, 크게 아래와 같은 문제점들이 발생했습니다.  

* 시스템 복잡도의 증가
	* 중앙화된 데이터 전송 영역이 없어, 데이터의 흐름을 파악하기 어렵고, 시스템 관리가 복잡함.
	* 시스템의 일부분에 문제가 발생하면, 연결된 모든 애플리케이션들을 확인해야 함.

* 데이터 일관성 유지의 어려움
	* 데이터가 여러 시스템과 데이터베이스에 분산되어 있는 경우, 한 시스템에서 변경된 데이터가 다른 시스템에 즉시 반영되지 않아 데이터의 일관성을 유지하기 어려움.

* 데이터 실시간 처리의 어려움
	* 전통적인 메시지 큐 시스템이나 데이터베이스는 대부분 배치 처리 방식을 사용함.
	* 이는 데이터를 실시간으로 처리하는 것이 어렵다는 것을 의미.

* 확장성 제한
	* 대부분의 전통적인 메시지 큐 시스템은 한정된 리소스 내에서 작동하므로, 대량의 데이터를 처리하는 데 제한이 있음.
	* 데이터의 양이 증가하면서 시스템을 확장해야 하는 상황에서 이런 제한이 큰 문제가 될 수 있음.

아파치 카프카(Apache Kafka)는 이런 문제점들을 해결하기 위해 링크드인에서 개발되었고, 현재는 Apache Software Foundation의 오픈 소스 프로젝트로 유지 관리되는 분산 스트리밍 플랫폼입니다.  

#### Kafka의 기본 구조

![1](https://github.com/witwint/TIL/assets/108222981/0b66e294-5b90-42f9-ac1b-1b9e851d47cc)

* 프로듀서(Producer)
프로듀서는 Kafka에 메시지를 발행하는 역할을 하는 컴포넌트입니다.  
프로듀서는 다양한 데이터 소스로부터 데이터를 가져와 Kafka의 특정 토픽에 메시지를 발행합니다.  

* 브로커(Broker)
브로커는 Kafka의 핵심 서버 컴포넌트로, 프로듀서로부터 메시지를 받아서 저장하고, 컨슈머에게 메시지를 전달하는 역할을 합니다.  
Kafka 클러스터는 여러 브로커들로 구성되며, 각 브로커는 하나 이상의 토픽의 메시지를 저장하고 관리합니다.  

* 토픽(Topic)
토픽은 Kafka에서 데이터를 분류하는 단위입니다.  
프로듀서는 메시지를 특정 토픽에 발행하고, 컨슈머는 토픽을 구독하여 메시지를 소비합니다. 토픽은 여러 파티션으로 나뉘어질 수 있고, 이를 통해 데이터를 병렬로 처리할 수 있습니다. 각 파티션은 순서가 보장된 메시지 스트림을 제공하며, 브로커가 클러스터 내에서 파티션을 분산하여 저장합니다.  

* 컨슈머(Consumer)
컨슈머는 Kafka의 특정 토픽을 구독하고, 해당 토픽의 메시지를 소비하는 역할을 하는 컴포넌트입니다.  
컨슈머는 하나 이상의 토픽을 구독할 수 있으며, 토픽의 파티션을 동시에 소비할 수 있습니다.

#### 예제

* 1 스프링디펜던시
![스크린샷 2024-04-12 200930](https://github.com/witwint/TIL/assets/108222981/cfe83a36-2c62-4243-93c6-195912ad267d)
스프링 카프카 디펜던시 등록  
```java
 // kafka
    implementation 'org.springframework.kafka:spring-kafka'
```

* 2 카프카서버
![스크린샷 2024-04-12 201024](https://github.com/witwint/TIL/assets/108222981/ea9a5996-3530-4106-81ae-32ec0b2b40a9)
카프카란 것은 우리가 스프링과 외부의 DB (mysql)같은것을 연결해서 사용하듯이 외부의 카프카 서비스를 사용하는것 예시에서는 ms에서 제공하는 카프카 서비스를 이용하는 예시  
서버를 하나 파서 도커로 카프카이미지 돌려서 손쉽게 구동할수도있음  

* 3 yml설정
```yml
spring:
  kafka:
    producer:
      bootstrap-servers: developeryhub.servicebus.windows.net:9093     # 브로커 주소
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  # 키값 직렬화방법
      value-serializer: org.apache.kafka.common.serialization.StringSerializer #벨류값 직렬화방법

    properties:  #연결서버 인증방법 상황에 맞게 설정해줘야함 ms제공 카프카에서는 이런식으로 하라고 되어있음
      sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://developeryhub.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=7MeGYZsuZ6hVYxtKv9wW+KW0o7qUJokz88rBw=";
      sasl.mechanism: PLAIN
      security.protocol: SASL_SSL
      
```

* 4 스프링 프로듀서 코드

  
컨트롤러
```java
@RestController
@RequestMapping("/kafka")
public class TestController {

	@Autowired
	TestService service;
	
	@GetMapping("/insert")
	public String insert() {
				
		return service.insert();
	} 
}
```

서비스
```java
@Slf4j
@Service
public class TestService {

	@Autowired
	private KafkaTemplate<String, String> kafkaTemplate;
	
	public String insert( ) {
		ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send("hub1", LocalDateTime.now().toString());
		future.addCallback(successCallback -> {
			log.info("[producer] successCallback. offset: " + successCallback.getRecordMetadata().offset() + "partition: " + successCallback.getRecordMetadata().partition());
			}, 
		errorCallback -> {
			log.error("[producer] errorCallback. msg: " + errorCallback.getMessage());
			}
		);
		return "{}";
	}
}
```
기본적으로 yml파일에 카프카 설정이 있으면 카프카 사용한다고 생각하고 가장 기본의 `KafkaTemplate` 빈으로 등록해줘서 바로사용가능  `<String, String>`은 아까 설정에서 스트링 직렬화 사용한다고 했기때문에 그것으로 등록  

`kafkaTemplate.send("hub1", LocalDateTime.now().toString());` 에서 `"hub1"` 카프카 서버에 등록해둔 토픽(그룹) `LocalDateTime.now().toString()`보내고싶은 데이터 넣어줌  
![스크린샷 2024-04-12 202339](https://github.com/witwint/TIL/assets/108222981/4310af5d-a74a-4b60-9468-93a2bed62f68)

`ListenableFuture<SendResult<String, String>> future` 여기서 `kafkaTemplate.send`는 비동기로 동작하기에 일단 값을 받아두고 나중에 해결하기위한 타입  

`future.addCallback` 결과가 어떤지에 따라서 화살표함수 동작을 작성해줄곳 첫번째 매개변수 = 성공, 두번째 매개변수 = 실패 성공의 경우 `successCallback`에서 이런저런 값 꺼낼 수 있음  

`localhost:8080/kafka/insert/` 여러번 요청 보내보면   
![스크린샷 2024-04-12 203146](https://github.com/witwint/TIL/assets/108222981/40b1ac5d-05a6-4dda-90b8-db56ff138e0a)
결과 뽑을 수 있음  
`partition`은 카프카에 해당 토픽에 대한 파티션 개수를 내가 설정해줘서(예시에서는 2개) 같은 요청을 보낼때 해당 토픽에 요청이 골고루 들어가는것을 확인가능 (0,1)  
`offset`은 해당 토픽의 파티션에 오프셋 값은 '파티션 내에서' 고유하고, 순차적으로 표기됩니다. 테이블의 pk (id) 개념과 비슷하다고 볼 수 있습니다. 예시에서는 몇번 시도를 미리 해둬서 조금 높은숫자부터 각파티션에서 독립적으로 1씩 증가하는걸 볼 수 있습니다.  

* 5 컨슈머 설정

```yml
spring:
  kafka:
    producer:
      bootstrap-servers: developeryhub.servicebus.windows.net:9093     # 브로커 주소
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    properties:
      sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://developeryhub.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=7MeGYZsuZ6hVYxtKv9wW+KW0o7qUJokz88rBw=";
      sasl.mechanism: PLAIN
      security.protocol: SASL_SSL


# 같은 프로젝트에서 컨슈머 설정할거라yml에 컨슈머만 추가 프로듀서쪽이랑 의미는 같음
    consumer:
      bootstrap-servers: developery-kafka.servicebus.windows.net:9093    # 브로커 주소
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```
yml 컨슈머 추가  

```java
@Slf4j
@Service
public class KafkaConsumer {

	@KafkaListener(topics = "hub1", groupId = "myGroup1")
    public void consume(String message) throws IOException {
		log.info("[consumer] Consumed message : {}", message);
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		log.info("end");
    }
}
```
컨슈머 서비스 클래스 추가  
`@KafkaListener(topics = "hub1", groupId = "myGroup1")`이부분이 중요한데 `topics = "hub1"`카프카의 어떠한 토픽을 읽을지, `groupId = "myGroup1"`컨슈머 그룹이름설정  
이렇게 하면 카프카 `hub1` 토픽에 메시지가 들어오면 자동으로 아래 함수가 실행되는것!  

프로듀서쪽에서 `LocalDateTime.now().toString())`스트링 값으로 보내줬기때문에 `public void consume(String message)`함수의 매개변수타입 스트링으로  

저 함수안에서 받은 메시지로 작업할거 작업하고 DB에 넣고 하는것들 다 해주면 됨 예시로 `sleep`넣어줌  

![스크린샷 2024-04-12 204447](https://github.com/witwint/TIL/assets/108222981/2a4a1b0a-bedf-405c-be09-aa0c285bbe1e)

`localhost:8080/kafka/insert/` 여러번 요청 보내보면 위와같은 결과 확인 가능  
다만 지금 상황에서는 문제가 있는데 4번의 요청을 연타해서 보내면 `public void consume`함수는 `sleep`때문에 첫번째 요청-> 5초작업 ->두번째 요청 ->5초작업 ...
싱글 스레드인 지금 상황에서는 마지막 줄까지 출력되는데 20초가 걸리는 상황  

* 6 다중처리법

방법 1 컨슈머스프링서버 여러개 띄우기  
스프링부트를 여러개 띄우면 각서버가 토픽의 파티션개수 (예시에서는 2개)에 맞게 분할해서 붙기때문에 지금 예시에서는 서버2개를 띄우면 토픽의 각 파티션에 각서버거 붙어서 한번에 2개씩 일 처리 가능하다 (실무에서는 더 많은 파티션 사용함)  

방법 2 스레드 늘리기  
물론 파티션 10개 사용한다고 서버를 10개를 돌릴수는 없기때문에 하나의 스프링부트에서 카프카 리스너의 스레드를 여러개 돌릴수있도록 설정함  

스레드를 늘리기위해서는 카프카 config로 빈으로 수동설정해서 등록해줘야함
```java
@Configuration
@AllArgsConstructor
public class KafkaConfig {

//우리가 사용할 카프카 메시지 키벨류값
	final ConsumerFactory<String, Object> consumerFactory;		// 자동 주입됨.
	
	@Bean
	public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
		ConcurrentKafkaListenerContainerFactory<String, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();

		factory.setConsumerFactory(consumerFactory);
		factory.setConcurrency(2);		// 이곳의 숫자만큼 consumer thread가 구동됨. 
		
		ContainerProperties properties = factory.getContainerProperties();
		properties.setAckMode(AckMode.MANUAL);		// 실무에서는 auto commit 대신 매뉴얼commit을 이용.
		
		return factory;
	}	
}
```
주석참조
`properties.setAckMode(AckMode.MANUAL);	`기본값은 auto로 카프카쪽에서 컨슈머 쪽으로 메시지 보내면 그냥 끝 다음꺼 보낼 수 있으면 보냄  
`AckMode.MANUAL`로 하면 컨슈머쪽에서 리턴값 넘겨줘야지 다음작업함  

* 7 컨슈머 수정
```java
@Slf4j
@Service
public class KafkaConsumer {

	@KafkaListener(topics = "hub1", groupId = "myGroup1")
    public void consume(@Payload String message,
    		@Header(KafkaHeaders.ACKNOWLEDGMENT) Acknowledgment acknowledgment,
    		@Header(name = KafkaHeaders.RECEIVED_MESSAGE_KEY, required = false) String key,
    		@Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
    		@Header(KafkaHeaders.OFFSET) long offset,
            @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
            @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long ts
    		) {
		log.info("[consumer] Consumed key: {}, partition: {}, offset: {}, topic: {}, ts: {}, message : {}",
				key, partition, offset, topic, ts, message);
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		acknowledgment.acknowledge();  // commit
		log.info("end");
    }
}
```
`@Header(KafkaHeaders.ACKNOWLEDGMENT) Acknowledgment acknowledgment,`이런식으로 추가적인 값들을 받을 수 있고 아까 `AckMode.MANUAL`로 수동설정해줬기때문에 `Acknowledgment acknowledgment`이건 꼭 받고나서 다 끝나면 `acknowledgment.acknowledge();  // commit`해줘야함 안해주면 카프카 큐애서 안빠짐  

![스크린샷 2024-04-12 210331](https://github.com/witwint/TIL/assets/108222981/5a435bae-81b2-4bff-8b9d-648cbcde3de3)
처음에 스프링 부트 구동 시키면 설정한데로 스레드 1개는 토픽의 파티션 1번으로 다른 1개를 토픽의 파이션2번으로 붙은 설정 확인할 수 있음  

![스크린샷 2024-04-12 210549](https://github.com/witwint/TIL/assets/108222981/083f664b-c9aa-4d6d-ae84-f93da2650289)
이번에 요청을 2연타 해보면 2개가 별개로 동작하는걸 확인할 수 있음  

* 8 오류가 발생하면?

만약 오류가 중간에 발생하면 어떻게 될까  
```java
@Slf4j
@Service
public class KafkaConsumer {
..
.
.
if (true) throw now RuntimeException("myexce") //강제 오류 추가
.
.
```

이 코드를 넣고 한번의 요청을 보내면  

![스크린샷 2024-04-12 210730](https://github.com/witwint/TIL/assets/108222981/0d0ab928-57d0-408a-b52e-f0786c8f16eb)
x10
위의 오류가 10번뜨게된다 이유는 카프카에서는 수동설정일때 `acknowledgment.acknowledge();`가 돌아오지않으면 내부적으로 10번의 재시도를 하는것이 기본값이다.  
이를 해결해보자  

* 8 오류해결 재시도 커스텀과 실패 메시지 바로 버리지않고 실패토픽으로 이동후 거기서 다시 재시도 구현

KafkaConfig 수정
```java
@Slf4j
@Configuration
@AllArgsConstructor
public class KafkaConfig {
	
	final ConsumerFactory<String, Object> consumerFactory;		// 자동 주입됨.
	
	final KafkaTemplate<String, Object> kafkaTemplate;
	
	@Bean
	public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
		ConcurrentKafkaListenerContainerFactory<String, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();

		factory.setConsumerFactory(consumerFactory);
		factory.setConcurrency(2);		// 이곳의 숫자만큼 consumer thread가 구동됨. 
		
		// deprecated 인데 예제들이 대부분 이거라서... 
		@SuppressWarnings("deprecation")
		final DeadLetterPublishingRecoverer recoverer = new DeadLetterPublishingRecoverer(kafkaTemplate,
		        (record, exception) -> {	
		        	// 에러 핸들러에서 지정한 재시도만큼 모두 최종 실패, 이곳이 실행됨. 이곳에서 어느 토픽으로 보낼지를 리턴해야함.
		        	log.info("run recoverer. original record: {}, exception: {}", record, exception);
		        	// DLQ 용도의 토픽을 만들어서 리턴함. 보통 .dlt 를 suffix 로 하고, record.partition 즉 동일 파티션으로 지정하나 
		        	// 통일성있게 규칙을 만들어서 따르면 됨.
		            return new TopicPartition(record.topic() + ".dlt", 0);	            
		        });		
		
		factory.setRetryTemplate(retryTemplate());
		
		// 아래 setRecoveryCallback 은 안해도 잘 동작함. 
		// retryTemplate 이 모두 실패시 콜백을 받아서 뭔가 처리하고 싶을때 구현하면 됨. 
		factory.setRecoveryCallback((context -> {
			if(context.getLastThrowable().getCause() instanceof RecoverableDataAccessException){
	            log.info("여기는 호출이 되지 않더군..");
			} 
			else{
				log.error("retryTemplate의 최대 재시도횟수까지 해도 실패한 경우 여기가 호출됨. setRecoveryCallback. context: " + context);
	    	
				// SeekToCurrentErrorHandler 을 쓰지 않을 경우, 아래 recoverer.accept() 코드 필요
				recoverer.accept((ConsumerRecord<?, ?>) context.getAttribute("record"),
						(Exception)context.getLastThrowable());
				
				// exception을 throw 해야지 최종적으로 commit 이 수행되어 해당 offset 을 무시하고 다음 offset을 시도함. 
				// 에러 핸들러가 동작하기 위해서는 exception throw 해야함. 그래야 DLQ로 메시지 넘어감. 
				throw new RuntimeException(context.getLastThrowable().getMessage());
			}
			return null;
		}));
		
		
		// retry template 를 반복하는 횟수. 1인 경우 (1)+1=2회,  3인 경우 (1)+3 = 4회, 
		// 1000L은 retry template 간 interval. 
//		ErrorHandler errorHandler = new SeekToCurrentErrorHandler(recoverer, new FixedBackOff(1000L, 1L));
//		factory.setErrorHandler(errorHandler);
		
		// SeekToCurrentErrorHandler 쓰지 않는다면 아래처럼 일반적인 에러 핸들러를 만들고 로깅만 하면 됨.
		factory.setErrorHandler((exception, record) -> {			
			log.info("errorHandler. exception: {}, record: {}", exception, record);
		});
	        
		ContainerProperties properties = factory.getContainerProperties();
		properties.setAckMode(AckMode.MANUAL);		// 실무에서는 auto commit 대신 매뉴얼 commit을 이용.
		
		return factory;
	}
	
	private RetryTemplate retryTemplate() {
		RetryTemplate retryTemplate = new RetryTemplate();
		 
        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(500L);  // 500ms 쉬었다가 ...
        retryTemplate.setBackOffPolicy(fixedBackOffPolicy);
 
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(5);			// 최대 5번까지 재시도
        retryTemplate.setRetryPolicy(retryPolicy);
 
        return retryTemplate;
    }
}
```
실무에서는 실패한 요청 바로 재시도해도 안될가능성이 높기때문에 몇초기다리는 옵션 사용  
1. RetryTemplate함수
2. factory.setRetryTemplate(retryTemplate()); 넣어주면 적용완료
3. 이상태로만 사용하면 바로 적용안됨, 실패했을때 어떻게 할것인가 exception handler등 이거 다해줘야적용가능
4. final KafkaTemplate<String, Object> kafkaTemplate; 카프카 프로듀서 만들었을때 그것
5. @SuppressWarnings("deprecation") recoverer 설정 추가해줌 재시도까지 다하고나서도 실패했을때 어떻게 할것인지
지금 deprecated인데 최신은 옆쪽 블로그 참조 (https://sejoung.github.io/2022/02/2022-02-10-dead_letter_queue/#spring-kafka-dead-letter-queue-%EC%84%A4%EC%A0%95)
6. factory.setRecoveryCallback으로 완전히 실패시 동작할 함수 recoverer 만든것(.dlt 토픽을 전송) 실행
7. factory.setErrorHandler 최종 에러헨들러 마지막에 동작

이동된 토픽에서 다시 요청받을 컨슈머

```java
@KafkaListener(topics = "hub1.dlt", groupId = "myGroup1")
    public void dltConsume(@Payload String message,
    		@Header(KafkaHeaders.ACKNOWLEDGMENT) Acknowledgment acknowledgment,
    		@Header(name = KafkaHeaders.RECEIVED_MESSAGE_KEY, required = false) String key,
    		@Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
    		@Header(KafkaHeaders.OFFSET) long offset,
            @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
            @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long ts
    		) {
		log.info("[dltConsume] dltConsumed key: {}, partition: {}, offset: {}, topic: {}, ts: {}, message : {}",
				key, partition, offset, topic, ts, message);
		
		
		acknowledgment.acknowledge();  // commit
		log.info("end");
    }
```

![1](https://github.com/witwint/TIL/assets/108222981/ebd89800-aa56-4bc2-8e53-41f99f477a61)
카프카에서 .dlt로 새로 만들어줍니다.  

`localhost:8080/kafka/insert/`로 기존꺼 오류뜨는 코드로 요청
```cil
23:11:14.736     : [producer] successCallback. partition: 0,  offset: 50
23:11:14.742     : [consumer] Consumed key: k:0, partition: 0, offset: 50, topic: hub1, ts: 1638799872909, message : 2021-12-06T23:11:11.183551900
23:11:15.253     : [consumer] Consumed key: k:0, partition: 0, offset: 50, topic: hub1, ts: 1638799872909, message : 2021-12-06T23:11:11.183551900
23:11:15.764     : [consumer] Consumed key: k:0, partition: 0, offset: 50, topic: hub1, ts: 1638799872909, message : 2021-12-06T23:11:11.183551900
23:11:16.275     : [consumer] Consumed key: k:0, partition: 0, offset: 50, topic: hub1, ts: 1638799872909, message : 2021-12-06T23:11:11.183551900
23:11:16.790     : [consumer] Consumed key: k:0, partition: 0, offset: 50, topic: hub1, ts: 1638799872909, message : 2021-12-06T23:11:11.183551900
23:11:16.792     : retryTemplate의 최대 재시도횟수까지 해도 실패한 경우 여기가 호출됨. setRecoveryCallback. context: [RetryContext: count=5, lastException=org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void com.developery.azure.KafkaConsumer.consume(java.lang.String,org.springframework.kafka.support.Acknowledgment,java.lang.String,int,long,java.lang.String,long)' threw exception; nested exception is java.lang.RuntimeException: myException, exhausted=false]
23:11:16.802     : run recoverer. original record: ConsumerRecord(topic = hub1, partition = 0, leaderEpoch = null, offset = 50, CreateTime = 1638799872909, serialized key size = 3, serialized value size = 29, headers = RecordHeaders(headers = [], isReadOnly = false), key = k:0, value = 2021-12-06T23:11:11.183551900), exception: {}

23:11:17.002     : errorHandler. exception: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener failed; nested exception is java.lang.RuntimeException: Listener method 'public void com.developery.azure.KafkaConsumer.consume(java.lang.String,org.springframework.kafka.support.Acknowledgment,java.lang.String,int,long,java.lang.String,long)' threw exception; nested exception is java.lang.RuntimeException: myException, record: ConsumerRecord(topic = hub1, partition = 0, leaderEpoch = null, offset = 50, CreateTime = 1638799872909, serialized key size = 3, serialized value size = 29, headers = RecordHeaders(headers = [], isReadOnly = false), key = k:0, value = 2021-12-06T23:11:11.183551900)
23:11:17.225     : [dltConsume] dltConsumed key: k:0, partition: 0, offset: 15, topic: hub1.dlt, ts: 1638799877000, message : 2021-12-06T23:11:11.183551900
23:11:17.226     : end
```

---
## 20240419
### 프로그래머스 주식투자 스택큐 2단계
문제 설명 초 단위로 기록된 주식가격이 담긴 배열 prices가 매개변수로 주어질 때, 가격이 떨어지지 않은 기간은 몇 초인지를 return 하도록 solution 함수를 완성하세요.  
제한사항 prices의 각 가격은 1 이상 10,000 이하인 자연수입니다. prices의 길이는 2 이상 100,000 이하입니다.

입력 {1, 2, 3, 2, 3} 출력 {4, 3, 1, 1, 0}

```java
public class Main {
    public static void main(String[] args) {
        int[] pricesMain = {1, 2, 3, 2, 3};

        Solution solution = new Solution();
        //System.out.println("Hello World");

        int[] arr3 = solution.solution(pricesMain);

        for (int i : arr3) {
            System.out.println("i = " + i);
        }
    }

    static class Solution {
        public int[] solution(int[] prices) {
            int[] answer = new int[prices.length];
            for (int i = 0; i < prices.length; i++) {
                if (i == prices.length - 1) {
                    answer[i] = 0;
                    break;
                }
                for (int j = i+1; j < prices.length; j++) {
                    if (prices[i] > prices[j]) {
                        answer[i] = j - i;
                        break;
                    }
                    else {
                        answer[i] = prices.length - i -1;
                    }
                }
            }
            return answer;
        }
    }
}
```
일단 나는 이중for문으로 그냥 풀었다. 이렇게 해도 효율성까지 통과는 했다.  
이걸 어떻게 스택큐를 활용하냐 감이 안와서 찾아봤는데 (https://velog.io/@imok-_/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%EC%A3%BC%EC%8B%9D%EA%B0%80%EA%B2%A9-java-Stack-%ED%99%9C%EC%9A%A9)  
그냥 이해만 해두자.. 오히려 시간도 더 걸리더라  

---

## 20240420
### 스프링 톰캣 직접 사용해보기

톰캣 실행법 다운받아서 -> 권한주고 -> 실행명령어  
기본적으로 8080포트 사용  

#### 날것의 자바 War로 톰캣에 올리기  

```java
plugins {
 id 'java'
 id 'war'
}
group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'
repositories {
 mavenCentral()
}
dependencies {
 //서블릿
 implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'
}
tasks.named('test') {
 useJUnitPlatform()
}
```
날것의 자바 gradle설정 war로 빌드하겠다는 설정과 최소한의 서블릿 라이브러리  

예시로 `/src/main`에 webapp이라는 폴더생성후 index.html파일 생성

```html
<html>
<body>index html</body>
</html>
```
간단 예시

```java
@WebServlet(urlPatterns = "/test")
public class TestServlet extends HttpServlet {
 @Override
 protected void service(HttpServletRequest req, HttpServletResponse resp) 
throws IOException {
 System.out.println("TestServlet.service");
 resp.getWriter().println("test");
 }
}
```
서블릿도 간단하게 등록  

`./gradlew build`로 빌드해보면 WAR파일생김
이거 압축 풀어보면 

* WEB-INF
	* classes
		* hello/servlet/TestServlet.class
	* lib
		* jakarta.servlet-api-6.0.0.jar
* index.html

이런 구조의 파일 생겼던것  

* JAR 소개
자바는 여러 클래스와 리소스를 묶어서 JAR (Java Archive)라고 하는 압축 파일을 만들 수 있다.  
이 파일은 JVM 위에서 직접 실행되거나 또는 다른 곳에서 사용하는 라이브러리로 제공된다.  
직접 실행하는 경우 main() 메서드가 필요하고, MANIFEST.MF 파일에 실행할 메인 메서드가 있는 클래스를 지정해두어야 한다.  
실행 예) java -jar abc.jar  
Jar는 쉽게 이야기해서 클래스와 관련 리소스를 압축한 단순한 파일이다. 필요한 경우 이 파일을 직접 실행할 수도 있고, 다른 곳에서 라이브러리로 사용할 수도 있다.  

* WAR 소개
WAR(Web Application Archive)라는 이름에서 알 수 있듯 WAR 파일은 웹 애플리케이션 서버(WAS)에 배포할 때사용하는 파일이다.  
JAR 파일이 JVM 위에서 실행된다면, WAR는 웹 애플리케이션 서버 위에서 실행된다.  
웹 애플리케이션 서버 위에서 실행되고, HTML 같은 정적 리소스와 클래스 파일을 모두 함께 포함하기 때문에 JAR와비교해서 구조가 더 복잡하다.  
그리고 WAR 구조를 지켜야 한다.  

* WAR 구조
* WEB-INF
	* classes : 실행 클래스 모음
	* lib : 라이브러리 모음
	* web.xml : 웹 서버 배치 설정 파일(생략 가능)
* index.html : 정적 리소스
WEB-INF 폴더 하위는 자바 클래스와 라이브러리, 그리고 설정 정보가 들어가는 곳이다.  
WEB-INF 를 제외한 나머지 영역은 HTML, CSS 같은 정적 리소스가 사용되는 영역이다.  

#### 톰캣 폴더에 WAR 배포방법
톰캣폴더/webapps쪽에 원래 있던 기본 톰캣app을 전부 삭제하고 빌드된 war 파일 복붙  
war파일명을 `ROOT.war`로 변경후에 다시 톰캣을 실행시키면 톰캣이 얘를 까서 알아서 실행해준다.  

로컬 IDE 인텔리제이에서 실행시키기 위해서는 내부 간단 설정을 해주면 인텔리제이 내부의 톰캣실행 딱 누르면 알아서 빌드되면서 실행된다. 설명참조  

---
## 20240422
### 프로그래머스 완전 탐색 카펫 2단계

Leo는 카펫을 사러 갔다가 아래 그림과 같이 중앙에는 노란색으로 칠해져 있고 테두리 1줄은 갈색으로 칠해져 있는 격자 모양 카펫을 봤습니다.  
Leo는 집으로 돌아와서 아까 본 카펫의 노란색과 갈색으로 색칠된 격자의 개수는 기억했지만, 전체 카펫의 크기는 기억하지 못했습니다.  Leo가 본 카펫에서 갈색 격자의 수 brown, 노란색 격자의 수 yellow가 매개변수로 주어질 때 카펫의 가로, 세로 크기를 순서대로 배열에 담아 return 하도록 solution 함수를 작성해주세요.  

* 제한사항
갈색 격자의 수 brown은 8 이상 5,000 이하인 자연수입니다.
노란색 격자의 수 yellow는 1 이상 2,000,000 이하인 자연수입니다.  
카펫의 가로 길이는 세로 길이와 같거나, 세로 길이보다 깁니다.

예시 10,2 -> 4,3 / 24,24 -> 8,6

```java
import java.util.*;

class Solution {
    public int[] solution(int brown, int yellow) {
            int[] answer = new int[2];
            int total = brown + yellow;
            for (int row = 3; row < brown; row++) {
                if (total % row != 0) {
                    continue;
                }
                int cal = total / row;
                if (row * cal == total) {
                    if ((row - 2) * (cal - 2) == yellow) {
                        answer[0] = row;
                        answer[1] = cal;
                        break;
                    }

                }
            }
            Arrays.sort(answer);
            for (int i = 0; i < answer.length / 2; i++) {
                int temp = answer[i];
                answer[i] = answer[answer.length - 1 - i];
                answer[answer.length - 1 - i] = temp;
            }


            return answer;
        }
}
```
처음에는 소인수분해를 해야하나 어찌해야하나 했는데 그냥 밖의 갈색을 기준으로 가로 길이를 루프돌면서 연산했다.  

---
## 20240423
### 서블릿 컨테이너 초기화  

WAS를 실행하는 시점에 필요한 초기화 작업들이 있다. 서비스에 필요한 필터와 서블릿을 등록하고, 여기에 스프링을 사용한다면 스프링 컨테이너를 만들고, 서블릿과 스프링을 연결하는 디스페처 서블릿도 등록해야 한다.  
WAS가 제공하는 초기화 기능을 사용하면, WAS 실행 시점에 이러한 초기화 과정을 진행할 수 있다.  

#### 서블릿 컨테이너  

서블릿 컨테이너는 실행 시점에 초기화 메서드인 onStartup() 을 호출해준다. 여기서 애플리케이션에 필요한 기능 들을 초기화 하거나 등록할 수 있다  
```java
public interface ServletContainerInitializer {
 public void onStartup(Set<Class<?>> c, ServletContext ctx) throws
ServletException; 
}
```
하지만 여기에 추가작업이 하나 더필요한데 `resources/META-INF/services/jakarta.servlet.ServletContainerInitializer` 이런 파일 만들어서 스프링 야믈파일 설정하듯이  
`hello.container.MyContainerInitV1`이 클레스 페키지 주소까지 써줘야한다. 파일명, 오타 나면 안된다.  

#### 서블릿 컨테이너 등록 2
자바 코드로 직접 등록하는 방식이 있는데  
```java
public interface AppInit {
 void onStartup(ServletContext servletContext);
}
```
아무렇게나 인터페이스 만들어두고  
```java
public class AppInitV1Servlet implements AppInit {
 @Override
 public void onStartup(ServletContext servletContext) {
 System.out.println("AppInitV1Servlet.onStartup");
 //순수 서블릿 코드 등록
 ServletRegistration.Dynamic helloServlet =
 servletContext.addServlet("helloServlet", new HelloServlet());
 helloServlet.addMapping("/hello-servlet");
 }
}
```
그거로 적당히 구현한다음  

```java
@HandlesTypes(AppInit.class)
public class MyContainerInitV2 implements ServletContainerInitializer {
 @Override
 public void onStartup(Set<Class<?>> c, ServletContext ctx) throws
ServletException {
 System.out.println("MyContainerInitV2.onStartup");
 System.out.println("MyContainerInitV2 c = " + c);
 System.out.println("MyContainerInitV2 container = " + ctx);
 for (Class<?> appInitClass : c) {
 try {
 //new AppInitV1Servlet()과 같은 코드
 AppInit appInit = (AppInit) 
appInitClass.getDeclaredConstructor().newInstance();
 appInit.onStartup(ctx);
 } catch (Exception e) {
 throw new RuntimeException(e);
 }
 }
 }
}
```
컨테이너 코드쪽에서 `@HandlesTypes(AppInit.class)`인터페이스 넣어주면 `Set<Class<?>> c` 매개변수에 구현체들 class로 들어온다. 이거를 for돌리면서직업 인스턴스생성해서 필요한 함수들 실행해주면 되는것  
이렇게 복잡하게 쓰는 이유는 그 첫번째 방법의 파일만들어서 야믈처럼 등록해주고 이런거도 번거롭고 더쉬운방법인 `@WebServlet(urlPatterns = "/test")` 이런 어노테이션 이용방법은 저 주소를 자바 코드로 if 쓰면서 내가 원할때 바꿀 수가없기때문이다.  

---
## 20240424
### 스프링 컨테이너 등록  

스프링 라이브러리를 그레이들에 넣어주고  

```java
@RestController
public class HelloController {
 @GetMapping("/hello-spring")
 public String hello() {
 System.out.println("HelloController.hello");
 return "hello spring!";
 }
}
```
익숙한 컨트롤러를 만들어준다.  

```java
@Configuration
public class HelloConfig {
 @Bean
 public HelloController helloController() {
 return new HelloController();
 }
}
```
위의 컨트롤러를 수동 빈등록으로 넣어준다. 컴포넌트스캔은 그냥 안써봄  

```java
public class AppInitV2Spring implements AppInit {
 @Override
 public void onStartup(ServletContext servletContext) {
 System.out.println("AppInitV2Spring.onStartup");
 //스프링 컨테이너 생성
 AnnotationConfigWebApplicationContext appContext = new
AnnotationConfigWebApplicationContext();
 appContext.register(HelloConfig.class);
 //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
 DispatcherServlet dispatcher = new DispatcherServlet(appContext);
 //디스패처 서블릿을 서블릿 컨테이너에 등록 (이름 주의! dispatcherV2)
 ServletRegistration.Dynamic servlet =
 servletContext.addServlet("dispatcherV2", dispatcher);
 // /spring/* 요청이 디스패처 서블릿을 통하도록 설정
 servlet.addMapping("/spring/*");
 }
}
```
`AppInit`의 구현체는 모두 자동으로 서블릿컨테이너에서 실행되니깐 하나더 만들어주면서 스프링 컨테이너(빈관리)생성 -> 스프링MVC디스패처서블릭생성 -> 스프링컨테이너, 디스패처서블릿연결 -> 이 디스패처서블릿을 서블릿컨테이너에 등록  
이렇게 하면 지금 예시에서는 `/spring`로 들어온 요청은 `/spring`이거 이후에 적힌 주소로 우리가 알던 컨트롤러등에서 요청받음  

#### 스프링 MVC 서블릿 컨테이너 초기화 지원  
서블릿컨테이너를 초기화하기위해 지금까지 한짓이  
`ServletContainerInitializer` 인터페이스구현 -> 어플리케이션초기화 `@HandlesTypes` -> `/META-INF/services/jakarta.servlet.ServletContainerInitializer`경로등록  
이런 번거로운 작업 필요했다.  

하지만 스프링이 제공하는 특별한 인터페이스를 구현하면 위과정이 필요없다.  
```java
public class AppInitV3SpringMvc implements WebApplicationInitializer {
 @Override
 public void onStartup(ServletContext servletContext) throws ServletException
{
 System.out.println("AppInitV3SpringMvc.onStartup");
 //스프링 컨테이너 생성
 AnnotationConfigWebApplicationContext appContext = new
AnnotationConfigWebApplicationContext();
 appContext.register(HelloConfig.class);
 //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
 DispatcherServlet dispatcher = new DispatcherServlet(appContext);
 //디스패처 서블릿을 서블릿 컨테이너에 등록 (이름 주의! dispatcherV3)
 ServletRegistration.Dynamic servlet =
 servletContext.addServlet("dispatcherV3", dispatcher);
 //모든 요청이 디스패처 서블릿을 통하도록 설정
 servlet.addMapping("/");
 }
}
```
그냥 `WebApplicationInitializer`를 구현한거뿐인데 이게 가능한 이유는 별거없다. 그레이들로 추가된 스프링 MVC를 까보면  
`WebApplicationInitializer` 이 인터페이스를 기반으로 `서블릿컨테이너를 초기화하기위해 지금까지 한짓` 이걸 그냥 미리해둔 코드가 있다.  
`WebApplicationInitializer` 이거 이름으로 어플리캐이션 초기화`@HandlesTypes` 자동으로 되어있고 경로등록도 그냥 미리 해둔것이다. 처음 서블릿 컨테이너 등록한거처럼  

스프링 부트를 사용하지않고 스프링을 서버에 올리기위해서는 이런 과정이 필요했다.  

---
## 20240425
### OX문제 자료구조 시간복잡도

#### 문제

1. 우리는 입력값과 연산 수행 시간의 상관관계를 나타내는 척도를 시간 복잡도라고 한다. (o/x)

2. 해당 코드의 시간복잡도는 O(n^3)이다.(o/x)
```java
for(int i=0; i<n; i++) {

    for(int j=0; j<n; j++) {

        for(int k=0; k<n; k++) {

            System.out.println(n * j + k);

        }

    }

}
```

3. 해당 코드의 시간복잡도는 O(n^2)이다. (o/x)
```java
for(int i=0; i<n; i++) {

    for(int j=0; j<m; j++) {

        System.out.println(n * m);

    }

}
```

4. 해당 코드의 시간복잡도는 O(n^2)이다. (o/x)
```java
public static int pibo(int n) {

    if (n == 0) return 0;
    else if (n == 1) return 1;
    return pibo(n - 1) + pibo(n - 2);

}
```

5. 해당 코드의 시간복잡도는 O(n^2)이다. (o/x)
```java
if(n%2 == 0) {
    System.out.println("짝수");
} else {
    System.out.println("홀수");
}

for(int i=0; i<n; i++) {
    System.out.println(n);

}
```

#### 답

1. 우리는 입력값과 연산 수행 시간의 상관관계를 나타내는 척도를 시간 복잡도라고 한다. 답 O
[](https://github.com/witwint/TIL/assets/108222981/7a2bb6e2-0515-4249-b021-49fa2c8f7d45)

2. 해당 코드의 시간복잡도는 O(n^3)이다. 답 O
```java
for(int i=0; i<n; i++) {

    for(int j=0; j<n; j++) {

        for(int k=0; k<n; k++) {

            System.out.println(n * j + k);

        }

    }

}
```
반복문이 3번 겹치기 때문에 시간복잡도가 n^3이 나온다.

3. 해당 코드의 시간복잡도는 O(n^2)이다. 답 X
```java
for(int i=0; i<n; i++) {

    for(int j=0; j<m; j++) {

        System.out.println(n * m);

    }

}
```
이중 포문 같지만 엄연히 다르다. n번 반복하는 반복문을 두 번 중첩시킨 것이 아니라 n번 반복문과 m번 반복문을 중첩시켰다. 막말로 n이 999999999999999인데 m은 1일 수도 있기 때문에 같은 n을 for에서 쓰는것과 달리 O(nm)이라고 나타내 주어야 한다.

4. 해당 코드의 시간복잡도는 O(n^2)이다. 답 X
```java
public static int pibo(int n) {

    if (n == 0) return 0;
    else if (n == 1) return 1;
    return pibo(n - 1) + pibo(n - 2);

}
```
위 코드는 피보나치수열을 구현한 것이다. pibo(n)을 호출하면 pibo(n-1)과 pibo(n-2)가 호출된다. 그리고 이 호출 작업은 총 n번 반복되기 때문에 이를 Big-O 표기법으로 나타내면 O(2ⁿ)이 된다.  
![](https://github.com/witwint/TIL/assets/108222981/dc066e43-38b1-4f96-82aa-4ace31932b02)  


5. 해당 코드의 시간복잡도는 O(n^2)이다. 답O
```java
if(n%2 == 0) {
    System.out.println("짝수");
} else {
    System.out.println("홀수");
}

for(int i=0; i<n; i++) {
    System.out.println(n);

}
```
위 코드는 O(n²)와 O(n)이 합쳐져 O(n²+n)이 될 것 같지만 유감스럽게도 O(n²)이 된다. n²에 비해서 n은 영향력이 낮으므로 이를 무시하고 표기하는 것이 규칙이다. 앞서 상수항과 영향력이 낮은 항을 무시하는 이유는 Big-O 표기법이 실제 소요시간을 나타내기 위한 방법이 아니라 자료의 수가 증가함에 따라 소요시간이 얼마나 증가하는지를 나타내기 위한 방법이기 때문이다.  

O(?) <- 이 괄호 안에 들어가는 식을 계산해서 1억당 1초 정도로 생각할 수 있다고 한다. 예를 들어 n이 1억인데 시간 복잡도가 O(n)이면 소요시간이 1초정도 걸린다는 뜻이다. n이 1억, m이 10, 시간 복잡도가 O(nm)이면 소요시간이 10초가 된다는 것이다.

---
## 20240428
## 20240319
### 프로그래머스 피로도 dfs 2단계
XX게임에는 피로도 시스템(0 이상의 정수로 표현합니다)이 있으며, 일정 피로도를 사용해서 던전을 탐험할 수 있습니다. 이때, 각 던전마다 탐험을 시작하기 위해 필요한 "최소 필요 피로도"와 던전 탐험을 마쳤을 때 소모되는 "소모 피로도"가 있습니다. "최소 필요 피로도"는 해당 던전을 탐험하기 위해 가지고 있어야 하는 최소한의 피로도를 나타내며, "소모 피로도"는 던전을 탐험한 후 소모되는 피로도를 나타냅니다. 예를 들어 "최소 필요 피로도"가 80, "소모 피로도"가 20인 던전을 탐험하기 위해서는 유저의 현재 남은 피로도는 80 이상 이어야 하며, 던전을 탐험한 후에는 피로도 20이 소모됩니다.  이 게임에는 하루에 한 번씩 탐험할 수 있는 던전이 여러개 있는데, 한 유저가 오늘 이 던전들을 최대한 많이 탐험하려 합니다. 유저의 현재 피로도 k와 각 던전별 "최소 필요 피로도", "소모 피로도"가 담긴 2차원 배열 dungeons 가 매개변수로 주어질 때, 유저가 탐험할수 있는 최대 던전 수를 return 하도록 solution 함수를 완성해주세요.  

예시
```java
int k = 80;
int[][] dungeons = {{80,20},{50,40},{30,10}};

답 : 3
```

코드
```java
public class Main {
    public static void main(String[] args) {
        int k = 80;
        int[][] dungeons = {{80,20},{50,40},{30,10}};


        Solution solution = new Solution();
        //System.out.println("Hello World");

        int arr3 = solution.solution(k, dungeons);

        System.out.println("arr3 = " + arr3);
    }

    static class Solution {
        private int answer = 0;
        private boolean[] visited;
        public int solution(int k, int[][] dungeons) {
            visited = new boolean[dungeons.length];
            dfs(0, k, dungeons);
            return answer;
        }

        private void dfs(int depth, int k, int[][] dungeons) {
            for (int i = 0; i < dungeons.length; i++) {
                if (visited[i] || k < dungeons[i][0]) {
                    continue;
                }
                visited[i] = true;
                dfs(depth + 1, k - dungeons[i][1], dungeons);
                visited[i] = false;
            }
            answer = Math.max(depth, answer);
        }

    }
}
```
이문제 어제도 풀었는데 dfs가 쉽지않아서 오늘 다시풀었다. 사실 암기해서 풀은느낌이긴하다..  

---
## 20240429
### 스프링 내장톰캣

#### 내장톰캣 기능 제공  
기존의 `war`파일은 무조건 서버위에서만 실행되던 파일이였다 그런데 톰캣도 자바인데 라이브러리로 내장으로 가질수는없을까? 해서 나온것이 내장톰캣  

```java
dependencies {
 //스프링 MVC 추가
 implementation 'org.springframework:spring-webmvc:6.0.4'
 //내장 톰캣 추가
 implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
}
.
.
.
//일반 Jar 생성
task buildJar(type: Jar) {
 manifest {
 attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
 }
 with jar
}
```
이정도 디펜던시를 사용하면 쓸 수 있다. jar생성 task도 추가  

이전의
* hello.servlet.HelloServlet
* hello.spring.HelloController
* hello.spring.HelloConfig
코드가 있다고할때
`main`함수에 이정도만 써주면 일단 톰캣을 사용할 수 있다.  
```java
public class EmbedTomcatServletMain {
 public static void main(String[] args) throws LifecycleException {
 System.out.println("EmbedTomcatServletMain.main");
 //톰캣 설정
 Tomcat tomcat = new Tomcat();
 Connector connector = new Connector();
 connector.setPort(8080);
 tomcat.setConnector(connector);
 //서블릿 등록
 Context context = tomcat.addContext("", "/");
 tomcat.addServlet("", "helloServlet", new HelloServlet());
 context.addServletMappingDecoded("/hello-servlet", "helloServlet");
 tomcat.start();
 }
}
```
톰캣생성 -> 포트연결 -> 톰캣서블릿등록 -> 경로등록 -> 스타트  
이러고 `http://localhost:8080/hello-servlet`로 요청보내면 지정된 컨트롤러가 실행된다.  

이걸기반으로 스프링을 등록하는거도 똑같다.

```java
public class EmbedTomcatSpringMain {
 public static void main(String[] args) throws LifecycleException {
 System.out.println("EmbedTomcatSpringMain.main");
 //톰캣 설정
 Tomcat tomcat = new Tomcat();
 Connector connector = new Connector();
 connector.setPort(8080);
 tomcat.setConnector(connector);
 //스프링 컨테이너 생성
 AnnotationConfigWebApplicationContext appContext = new
AnnotationConfigWebApplicationContext();
 appContext.register(HelloConfig.class);
 //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
 DispatcherServlet dispatcher = new DispatcherServlet(appContext);
 //디스패처 서블릿 등록
 Context context = tomcat.addContext("", "/");
 tomcat.addServlet("", "dispatcher", dispatcher);
 context.addServletMappingDecoded("/", "dispatcher");
 tomcat.start();
 }
}
```
위랑 비슷하게 스프링 컨테이너를 실행하고 내장톰캣에 디스패처 서블릿을 등록해줬다. 그러고 실행  
`http://localhost:8080/hello-spring` 똑같이 요청해주면 응답이 나온다.  

사실 진짜 톰캣을 딥하게 직접 사용하려면 여러 예외처리와 이것저것 해줘야할게 많은데 배워야할게 많으니 이정도만 이해하고 넘어가도된다.  

#### 빌드와 배포
우리는 이제 순수 자바의 main 메소드만 실행하면 되기때문에 jar로 빌드해주면된다.  
그리고 jar는 `META-INF/MANIFEST.MF`파일안에 main() 메서드의 클래스를 지정해줘야하는데
```java
task buildJar(type: Jar) {
 manifest {
 attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
 }
 with jar
}
```
task에 그부분을 같이 넣어줬다.  

`./gradlew clean buildJar`하고 `build/libs/embed-0.0.1-SNAPSHOT.jar`위치에 `java -jar embed-0.0.1-SNAPSHOT.jar`이거 실행해주면???

안된다;;;;

왜냐하면 `embed-0.0.1-SNAPSHOT.jar`를 까보면 우리가 라이브러리 등록한 MVC, 톰캣 라이브러리가 아에없다.  
기본적인 jar는 war와 다르게 lib폴더에 라이브러리 jar를 포함할 수 없고, jar안에 다른 jar를 포함하지 못하도록 규정되어있다.  

그래서 나온방법  

```java
task buildFatJar(type: Jar) {
 manifest {
 attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
 }
 duplicatesStrategy = DuplicatesStrategy.WARN
 from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : 
zipTree(it) } }
 with jar
}
```
task에 FatJar추가 -> 내가 등록한 라이브러리의 jar도 모두 압축해제해서 class파일로 전부까서 기존의 내 코드와 합쳐 jar만들기  

이렇게하면 `java -jar embed-0.0.1-SNAPSHOT.jar`실행했을때 `INFO: Starting ProtocolHandler ["http-nio-8080"]`서버가 띄워진다.  

다만  이방법도 단점이 있는데 `META-INF/services/jakarta.servlet.ServletContainerInitializer`이런 기초적인 서블릿등록파일이 여러 라이브러리에 동시에있을 수 있다.  
이경우 한쪽이 버려지거나 하는데 이러면 버려진쪽은 아에 실행이 안된다.  

그렇지만 이런문제들까지 스프링부트가 해결해준다. 다음에 알아보기  

---
## 20240430
### kafka Connect

#### Kafka Connect란?

먼저 Kafka는 Producer와 Consumer를 통해 데이터 파이프라인을 만들 수 있다. 예를 들어 A서버의 DB에 저장한 데이터를 Kafka Producer/Consumer를 통해 B서버의 DB로도 보낼 수 있다. 이러한 파이프라인이 여러개면 매번 반복적으로 파이프라인을 구성해야줘야한다. KafkConnect는 이러한 반복적인 파이프라인 구성을 쉽고 간편하게 만들 수 있게 만들어진 Apache Kafka 프로젝트중 하나다.  

![1](https://github.com/witwint/TIL/assets/108222981/cf4a3707-0c21-44cc-be90-effd421b1fee)

위 사진을 보면 Kafka Connect를 이용해 왼쪽의 DB의 데이터를 Connect와 Source Connector를 사용해 Kafka Broker로 보내고 Connect와 Sink Connector를 사용해 Kafka에 담긴 데이터를 DB에 저장하는 것을 알 수 있다.  
여기서 중요한 건 Connect와 Connector의 차이와 Source Connector와 Sink Connector이다.  

```
    Connect: Connector를 동작하게 하는 프로세서(서버)
    Connector:  Data Source(DB)의 데이터를 처리하는 소스가 들어있는 jar파일

    Source Connector: data source에 담긴 데이터를 topic에 담는 역할(Producer)을 하는 connector
    Sink Connector: topic에 담긴 데이터를 특정 data source로 보내는 역할(Consumer 역할)을 하는 connector
```

또한 Connect는 단일 모드(Standalone)와 분산 모드(Distributed)로 이루어져있다.  

```
    단일 모드(Standalone): 하나의 Connect만 사용하는 모드

    분산 모드(Distributed): 여러개의 Connect를 한개의 클러스트로 묶어서 사용하는 모드.
      -> 특정 Connect가 장애가 발생해도 나머지 Connect가 대신 처리하도록 함
```

Kafka Connect는 REST API를 사용해서 Connector를 등록 및 사용할 수 있다. 이제 예제를 한번 해보자.  

#### Kafka Connect 예제

이번 예제에서 mysql table에 데이터를 insert하면 다른 table에 데이터가 그대로 저장되는 예제를 해본다.  
먼저 선작업으로 DB에 Table을 하나 만들자.  

```sql
CREATE SCHEMA test;

CREATE TABLE test.users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20)
)
```

#### 기본설정
1. Kafka 설치
2. (설치후 해당 디렉토리 이동후 아래처럼 server.properties 파일을 열어) ` vi config/server.properties`
3. listneres=PLAINTEXT://:9092 주석을 풀고 localhost를 입력해준다.
![1](https://github.com/witwint/TIL/assets/108222981/6d12f407-4725-4158-91ea-e549793aec7c)
4. kafka 디렉토리에서 아래의 명령어를 통해 zookeeper server와 kafka broker를 실행해준다.
```
$ ./bin/zookeeper-server-start.sh ./config/zookeeper.properties
$ ./bin/kafka-server-start.sh ./config/server.properties
```
5. Kafka Connect 설치
6. 설치후 kafka-connect 디렉토리에서 아래 명령어로 kafka-connect 실행(zookeeper와 kafka broker가 실행되어있어야함) `$ ./bin/connect-distributed ./etc/kafka/connect-distributed.properties`
7. 실행하고 나서 kafka 디렉토리에서 아래 명령어를 실행하여 topic 리스트를 확인하면 `$ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list`
8. 다음과 같은 topic이 생성된 걸 확인할 수 있다.
![1](https://github.com/witwint/TIL/assets/108222981/06f7a4fc-4aff-4ed3-b817-71499350a3fa)
9. Connector 설치
10. 설치를 하면 아래의 디렉토리가 설치된다.
![1](https://github.com/witwint/TIL/assets/108222981/607c5b1d-a3aa-444e-87c0-0fe0710e6f59)
11. Kafka Connect 디렉토리에서 `$ vi etc/kafka/connect-distributed.properties`
12. 아래처럼 plugin.path={kafka connect jdbc plugin 경로}/lib을 입력해준다.
![1](https://github.com/witwint/TIL/assets/108222981/3a07475f-a703-4190-8f4a-f1417c2dd10e)
13. Mysql Connector 설치 (설치후 mysql-connector-java-버전.jar파일을 {kafka-connector디렉토리}/share/java/kafka에 복사한다.)
![1](https://github.com/witwint/TIL/assets/108222981/92ec855b-5ca4-40b9-9c57-640def4aeb01)

#### Source Connector
1. Connect의 REST API를 통해 Source Connector와 Sink Connector를 생성해보자.
2. Source Connector
```
{
    "name": "my-source-connect",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
        "connection.url": "jdbc:mysql://localhost:3306/test",
        "connection.user":"root",
        "connection.password":"비밀번호",
        "mode":"incrementing",
        "incrementing.column.name" : "id",
        "table.whitelist" : "users",
        "topic.prefix" : "example_topic_",
        "tasks.max" : "1",
    }
}
cUrl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
```
각 속성의 의미는 다음과 같다.

* name : source connector 이름(JdbcSourceConnector를 사용)
* config.connector.class : 커넥터 종류(JdbcSourceConnector 사용)
* config.connection.url : jdbc이므로 DB의 정보 입력
* config.connection.user : DB 유저 정보
* config.connection.password : DB 패스워드
* config.mode : "테이블에 데이터가 추가됐을 때 데이터를 polling 하는 방식"(bulk, incrementing, timestamp, timestamp+incrementing)
* config.incrementing.column.name : incrementing mode일 때 자동 증가 column 이름
* config.table.whitelist : 데이터를 변경을 감지할 table 이름
* config.topic.prefix : kafka 토픽에 저장될 이름 형식 지정 위 같은경우 whitelist를 뒤에 붙여 example_topic_users에 데이터가 들어감
* tasks.max : 커넥터에 대한 작업자 수(본문 인용.. 자세한 설명을 찾지 못함)

3. 실행 후 아래 요청을 통해 생성된 Connectors List를 확인할 수 있다. `cUrl -X GET -d @- http://localhost:8083/connectors`
![1](https://github.com/witwint/TIL/assets/108222981/39fa4df4-ab14-48b3-869a-07070f47a8be)
4. 이제 test.users table에 데이터를 insert해보자.
![1](https://github.com/witwint/TIL/assets/108222981/04e23cce-5896-4e74-99ca-ae1e2a1e84be)
그리고 kafka server 디렉토리에서 아래 명령어를 통해 topic 리스트를 확인해보면 example_topic_users 토픽이 생성된 것을 확인할 수 있다.(DB에 데이터를 삽입함으로써 Source Connector가 DB데이터를 topic에 push한 것)
`$ ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list`
![1](https://github.com/witwint/TIL/assets/108222981/30050950-678e-40de-9a97-9d914838be7a)
토픽리스트

#### Sink Connector
1. 이제 Source Connector를 통해 topic에 넣은 데이터를 Sink하기 위해 Sink Connector를 생성해보자 Source Connector와 동일하게 아래처럼 API통해 생성한다.
```
{
    "name": "my-pksink-connect",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "connection.url": "jdbc:mysql://localhost:3306/test",
        "connection.user":"root",
        "connection.password":"비밀번호",
        "auto.create":"true",
        "auto.evolve":"true",
        "delete.enabled":"false",
        "tasks.max":"1",
        "topics":"example_topic_users"
    }
}

cUrl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
```
sourceConnector와 겹치는 속성을 제외한 속성은 다음과 같은 뜻을 가진다  

* auto.create : 데이터를 넣을 테이블이 누락되었을 경우 자동 테이블 생성 여부
* auto.evolve : 특정 데이터의 열이 누락된 경우 대상 테이블에 ALTER 구문을 날려 자동으로 테이블 구조를 바꾸는지 여부 (하지만 데이터 타입 변경, 컬럼 제거, 키본 키 제약 조건 추가등은 시도되지 않는다)
* delete.enabled : 삭제 모드 여부

더 자세한 속성은 해당 링크에서 확인할 수 있다.  
https://docs.confluent.io/kafka-connect-jdbc/current/sink-connector/sink_config_options.html  

생성후 users table에 데이터를 insert하면  
![1](https://github.com/witwint/TIL/assets/108222981/175aec27-25c7-4557-b2fd-54e219c15675)
example_topic_users table이 생성된 것을 볼 수 있고  

![img1 daumcdn](https://github.com/witwint/TIL/assets/108222981/6e3188ca-6ef9-4008-acc9-3dd4d73f5ee1)  
![1](https://github.com/witwint/TIL/assets/108222981/9b99abfd-e263-45f9-9380-7308045be22d)  
서로 테이블의 내용이 같은 것을 확인할 수 있다.  

#### 참조
https://cjw-awdsd.tistory.com/53  

---

## 20240501
### 스프링부트 클래스 만들기와 부트 내부확인

#### 스프링 부트 클래스 만들기  
이전의 그냥 main함수안에서 톰캣실행 서블릿등등 만들고 연결하고실행하는코드 한번 모아서 함수하나로 정리  

```java
public class MySpringApplication {
 public static void run(Class configClass, String[] args) {
 System.out.println("MySpringBootApplication.run args=" + List.of(args));
 //톰캣 설정
 Tomcat tomcat = new Tomcat();
 Connector connector = new Connector();
 connector.setPort(8080);
 tomcat.setConnector(connector);
 //스프링 컨테이너 생성
 AnnotationConfigWebApplicationContext appContext = new
AnnotationConfigWebApplicationContext();
 appContext.register(configClass);
 //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
 DispatcherServlet dispatcher = new DispatcherServlet(appContext);
 //디스패처 서블릿 등록
 Context context = tomcat.addContext("", "/");
 tomcat.addServlet("", "dispatcher", dispatcher);
 context.addServletMappingDecoded("/", "dispatcher");
 try {
 tomcat.start();
 } catch (LifecycleException e) {
 throw new RuntimeException(e);
 }
 }
}
```

부트에 필요한 커스텀 어노테이션 제작(예시로는 컴포넌트스캔용으로하나만 추가기능 등록된것)
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ComponentScan
public @interface MySpringBootApplication {
}
```

그럼 실제 사용 main함수 ,클래스에서는

```java
@MySpringBootApplication
public class MySpringBootMain {
 public static void main(String[] args) {
 System.out.println("MySpringBootMain.main");
 MySpringApplication.run(MySpringBootMain.class, args);
 }
}
```
우리가 보던 스프링부트와 같은 모양  

#### 부트 내부 살펴보기

```java
@SpringBootApplication
public class BootApplication {
public static void main(String[] args) {
SpringApplication.run(BootApplication.class, args);
}
}
```
실제 부트 main함수는 이렇게 생겼는데 저 run함수 까보면 결국 스프링컨테이너 생성, was에 서블릿들 연결해서 실행 하는 코드 똑같이 들어있음 부가적인것 있긴하지만  

아무튼 이런식으로 만든 부트 빌드해보면 `./gradlew clean build`

* boot-0.0.1-SNAPSHOT.jar
	* META-INF
		* MANIFEST.MF
	* org/springframework/boot/loader
		* JarLauncher.class : 스프링 부트 main() 실행 클래스
	* BOOT-INF
		* classes : 우리가 개발한 class 파일과 리소스 파일
			* hello/boot/BootApplication.class
			* hello/boot/controller/HelloController.class
			* …
		* lib : 외부 라이브러리
			* spring-webmvc-6.0.4.jar
			* tomcat-embed-core-10.1.5.jar
			* ...
		* classpath.idx : 외부 라이브러리 경로
		* layers.idx : 스프링 부트 구조 경로

잘보면 이전에 시도해봤던 Fat jar랑은 좀다름 jar안에 jar안된다고 하는데 들어있고 하는데 살펴보면

`META-INF/MANIFEST.MF`
```
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: hello.boot.BootApplication
Spring-Boot-Version: 3.0.2
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Build-Jdk-Spec: 17
```
`Main-Class`에 우리가 등록한 부트 main함수가 아님 이유는 부트가 기존의 jar를 좀더 손봐서 `org.springframework.boot.loader.JarLauncher`를 먼저 실행하고  
`org.springframework.boot.loader.JarLauncher`이녀석이 내부의 jar를 풀어서 사용할 수 있게 세팅을 한번더 한다음 그다음에 `Start-Class: hello.boot.BootApplication` 내가 등록한 애를 실행 할 수 있도록 좀더 커스텀한것임  
`war`구조를 기반으로 비슷하게 만들었음  


참고로 `boot-0.0.1-SNAPSHOT-plain.jar`는 라이브러리 빠진 순수 우리가 작성한 코드만 들어있는 jar인데 쓸일거의없음  

이런 부분은 전체적으로 이해만 하고 넘어가자 배울것이 너무 많다.  

---

## 20240502
### 스프링 부트 스타터와 라이브러리 관리

#### 직접관리시절

```java
dependencies {
 //1. 라이브러리 직접 지정
 //스프링 웹 MVC
 implementation 'org.springframework:spring-webmvc:6.0.4'
 //내장 톰캣
 implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
 //JSON 처리
 implementation 'com.fasterxml.jackson.core:jackson-databind:2.14.1'
 //스프링 부트 관련
 implementation 'org.springframework.boot:spring-boot:3.0.2'
 implementation 'org.springframework.boot:spring-boot-autoconfigure:3.0.2'
 //LOG 관련
 implementation 'ch.qos.logback:logback-classic:1.4.5'
 implementation 'org.apache.logging.log4j:log4j-to-slf4j:2.19.0'
 implementation 'org.slf4j:jul-to-slf4j:2.0.6'
 //YML 관련
 implementation 'org.yaml:snakeyaml:1.33'
}
```
그냥 모든 라이브러리, 버전 다 넣고 호환성 확인도 내가

#### 버전관리 도움

```java
plugins {
 id 'org.springframework.boot' version '3.0.2'
 id 'io.spring.dependency-management' version '1.1.0' //추가
 id 'java'
}

dependencies {
 //2. 스프링 부트 라이브러리 버전 관리
 //스프링 웹, MVC
 implementation 'org.springframework:spring-webmvc'
 //내장 톰캣
 implementation 'org.apache.tomcat.embed:tomcat-embed-core'
 //JSON 처리
 implementation 'com.fasterxml.jackson.core:jackson-databind'
 //스프링 부트 관련
 implementation 'org.springframework.boot:spring-boot'
 implementation 'org.springframework.boot:spring-boot-autoconfigure'
 //LOG 관련
 implementation 'ch.qos.logback:logback-classic'
 implementation 'org.apache.logging.log4j:log4j-to-slf4j'
 implementation 'org.slf4j:jul-to-slf4j'
 //YML 관련
 implementation 'org.yaml:snakeyaml'
}

```
` id 'io.spring.dependency-management' version '1.1.0' //추가` 이거 넣으면 버전 미리 스프링에서 확인한거 알아서 넣어줌  
별다른게 아니라 깃헙 스프링공식쪽에 그레들로 대중적인거 전부다 싹다 호환되는 버전 정리해서 써놓은것임 그거 기반으로 설정해줌  

#### 스프링 부트 스타터

```java
dependencies {
 //3. 스프링 부트 스타터
 implementation 'org.springframework.boot:spring-boot-starter-web'
}
```
위쪽에 라이브러리들 web으로 사용할때 대부분 필요한 라이브러리 인데 스프링부트에서 그걸모아서 상위의 라이브러리로 관리해줌 그래서 이거 한줄만 넣어주면  
mvc, 로그, 직렬화, 톰캣 등등 필요한거 한번에 끌어다 쓸수있게 모아준것  

여담으로 내 맘데로 버전 하나만 바꾸고싶고하면 `ext['tomcat.version'] = '10.1.4`이런식으로 한줄 추가해주면 예시에서는 톰캣의 버전정보만 조금 다르게 설정도 가능  

---
## 20240503
### 프로그래머스 완전탐색 모음사전 2단계
사전에 알파벳 모음 'A', 'E', 'I', 'O', 'U'만을 사용하여 만들 수 있는, 길이 5 이하의 모든 단어가 수록되어 있습니다. 사전에서 첫 번째 단어는 "A"이고, 그다음은 "AA"이며, 마지막 단어는 "UUUUU"입니다. 단어 하나 word가 매개변수로 주어질 때, 이 단어가 사전에서 몇 번째 단어인지 return 하도록 solution 함수를 완성해주세요.  

예시 "I" = 1563

코드
```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        String word = "AAAE";


        Solution solution = new Solution();
        //System.out.println("Hello World");

        int arr3 = solution.solution(word);

        System.out.println("arr3 = " + arr3);
    }

    static class Solution {
        private int answer = 0;
        private ArrayList<String> totalString = new ArrayList<>();

        private final ArrayList<String> useWord = new ArrayList<>(List.of("A","E","I","O","U"));
        public int solution(String word) {

            for (String s : useWord) {
                dfs(0, s);
            }

            for(int i=0; i<totalString.size(); i++){
                if(totalString.get(i).equals(word)){
                    answer = i + 1;
                    break;
                }
            }

            return answer;


        }

        private void dfs(int depth, String string) {

            if (depth >= 5) return;

            if (!totalString.contains(string)) {
                totalString.add(string);
            }
            for (String s : useWord) {
                dfs(depth + 1, string + s);
            }
        }

    }
}
```
처음에 그냥 if문 반복으로 풀긴했었는데 이게 나중에 보고나니 저 단어생성의 순서가 DFS로 돌릴때와 똑같다고한다.  
이걸 생각하기 어려웠고 DFS구현이 쉽지않았다.  

---
## 20240504
### 스프링부트 자동구성 방식

#### 설명

```java
dependencies {
 implementation 'org.springframework.boot:spring-boot-starter-jdbc'
 implementation 'org.springframework.boot:spring-boot-starter-web'
 compileOnly 'org.projectlombok:lombok'
 runtimeOnly 'com.h2database:h2'
 annotationProcessor 'org.projectlombok:lombok'
 testImplementation 'org.springframework.boot:spring-boot-starter-test'
 //테스트에서 lombok 사용
 testCompileOnly 'org.projectlombok:lombok'
 testAnnotationProcessor 'org.projectlombok:lombok'
}
```
JDBC를 사용하는 프로젝트를 한다고 가정해보자

```java
@Slf4j
@Configuration
public class DbConfig {
 @Bean
 public DataSource dataSource() {
 log.info("dataSource 빈 등록");
 HikariDataSource dataSource = new HikariDataSource();
 dataSource.setDriverClassName("org.h2.Driver");
 dataSource.setJdbcUrl("jdbc:h2:mem:test");
 dataSource.setUsername("sa");
 dataSource.setPassword("");
 return dataSource;
 }
 @Bean
 public TransactionManager transactionManager() {
 log.info("transactionManager 빈 등록");
 return new JdbcTransactionManager(dataSource());
 }
 @Bean
 public JdbcTemplate jdbcTemplate() {
 log.info("jdbcTemplate 빈 등록");
 return new JdbcTemplate(dataSource());
 }
}
```
옛날같으면 무조건 설정파일에서 내가 사용할 모든 빈들 이런식으로 수동등록해야한다. -> 아무튼 이러면 JdbcTemplate사용시 정상동작  

그런데 저 설정파일을 제거해보자 `@Configuration`이거 주석처리  

이렇게 해도 모든 코드는 정상동작하고 테스트 코드상에서도 실제로 빈이 등록되어있다 어떻게??

#### 자동등록

`implementation 'org.springframework.boot:spring-boot-starter-jdbc'`이런 `spring-boot-starter`로 되어있는 라이브러리를 땡기면  
해당 라이브러리 내부에 사용하는 라이브러리들을 자동 빈등록해주는 설정파일이 아에 들어있다.

JDBC쪽 라이브러리 하나 까보면
```java
//자동등록 내부에Configuration 있음 after 이클래스생성이후 동작하도록
@AutoConfiguration(after = DataSourceAutoConfiguration.class)
//if문 같은것 저런 클래스 있어야동작하도록(빈등록순서)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(JdbcProperties.class)
//설정추가
@Import({ DatabaseInitializationDependencyConfigurer.class, 
JdbcTemplateConfiguration.class,
NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
}
```
이런식으로 1차적으로 설정등록 코드가 있고 하나더 들어가서 `@Import`의 `JdbcTemplateConfiguration.class`확인  

```java
@Configuration(proxyBeanMethods = false)
//JdbcOperations 빈이 없을 때 동작 = 사용자가 새로 JdbcTemplate빈 등록했을때는 동작하지말라는 뜻
@ConditionalOnMissingBean(JdbcOperations.class) //JdbcOperations는 JdbcTemplate의 인터페이스 이걸로 사용자가 등록했는지 알수있
class JdbcTemplateConfiguration {
@Bean
@Primary
JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) 
{
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
JdbcProperties.Template template = properties.getTemplate();
jdbcTemplate.setFetchSize(template.getFetchSize());
jdbcTemplate.setMaxRows(template.getMaxRows());
if (template.getQueryTimeout() != null) {
jdbcTemplate.setQueryTimeout((int) 
template.getQueryTimeout().getSeconds());
}
return jdbcTemplate;
}
}
```

#### 여담

Auto Configuration = 자동설정 = 자동등록

자동설정 : 컴퓨터 용어에서는 환경 설정, 설정이라는 뜻(빈들을 자동으로 등록해서 스프링이 동작하는 환경을 자동으로 설정)  
자동등록 : 컴퓨터라고 하면 CPU, 메모리등을 배치해야 컴퓨터가 동작한다. 이렇게 배치하는 것을 구성 자동 구성은 스프링 실행에 필요한 빈들을 자동으로 배
치해주는 것  

Auto Configuration은 자동 구성이라는 단어를 주로 사용(자동 구성이 어떻게 동작하는지 내부 원리 이해)  
Configuration이 단독으로 사용될 때는 설정이라는 단어를 사용(특정 조건에 맞을 때 설정이 동작하도록 한다)  

---

### 루시카토 OX CS 자료구조 정렬종류  

#### 문제

* 선택정렬의 시간복잡도는 최선 평균 최악 모두 O(n^2)이다. O/X

* 삽입정렬은 최선의경우에는 O(n)의 시간복잡도를 가지게된다. O/X

* 버블정렬을 수행할때 공간복잡도는 O(n^2)이다. O/X

* 퀵정렬은 이미 정렬된 데이터에서 매우 효율적이다. O/X

* 병합정렬은 LikedList를 사용해서 구현하면 더 효율적이다. O/X

* 힙 정렬은 최악의 경우에는 시간복잡도가 O(n^2)이 나올 수 있다. O/X

---

#### 답

* 선택정렬의 시간복잡도는 최선 평균 최악 모두 O(n^2)이다. O

선택정렬  
![](https://github.com/justindevcode/TIL/assets/108222981/488cb547-bf1f-4197-9f1a-fda0e438f3fd)  
`선택 정렬은 주어진 배열에서 가장 작은 값을 찾아 첫 번째 원소와 교환한 후, 그 다음으로 작은 값을 두 번째 원소와 교환하는 방식을 반복하여 정렬을 수행한다.`  
시간복잡도  
선택 정렬의 시간 복잡도는 O(n^2)이며 최선, 평균, 최악의 경우 모두 동일하다.  
모두 정렬된 배열이어도 배열의 길이만큼 배열의 처음부터 끝까지 순회해야 하기 때문이다.  

공간복잡도  
주어진 배열 안에서 교환(swap)을 통해, 정렬이 수행되므로 O(1)이다.  

---

* 삽입정렬은 최선의경우에는 O(n)의 시간복잡도를 가지게된다. O

삽입정렬  
![1](https://github.com/justindevcode/TIL/assets/108222981/7600f4a8-e931-4619-8c59-91c36678f5be)  
`삽입 정렬은 배열을 정렬된 부분과 정렬되지 않은 부분으로 나누고, 정렬되지 않은 부분의 첫 번째 원소를 정렬된 부분에 적절한 위치에 삽입하는 방식으로 정렬을 수행한다.`  
시간복잡도  
선택 정렬의 시간 복잡도는 O(n^2)이며 최악과 평균은 동일하다. 하지만, 모두 정렬이 되어 있는 경우, 한 번씩 밖에 비교를 안하므로 최선의 경우 O(n)의 시간 복잡도를 가지게 된다.  

공간복잡도  
주어진 배열 안에서 교환(swap)을 통해, 정렬이 수행되므로 O(1)이다.  

---

* 버블정렬을 수행할때 공간복잡도는 O(n^2)이다. X

버블정렬  
![1](https://github.com/justindevcode/TIL/assets/108222981/b9d0da74-900b-4809-870b-c60467a7d3b3)  
`버블 정렬은 인접한 두 원소를 비교하고 필요에 따라 위치를 교환하는 방식으로 정렬을 수행한다.`  
시간복잡도  
버블 정렬은 정렬이 돼있던 안돼있던, 2개의 원소를 비교하기 때문에 최선, 평균, 최악의 경우 모두 시간 복잡도가 O(n^2)으로 동일하다.  

공간복잡도  
주어진 배열 안에서 교환(swap)을 통해, 정렬이 수행되므로 O(1)이다.  

---

* 퀵정렬은 이미 정렬된 데이터에서 매우 효율적이다. X

퀵정렬  
![1](https://github.com/justindevcode/TIL/assets/108222981/970ed6a1-4c45-4a19-a15b-6f35e94df522)  
`퀵 정렬은 분할 정복 방식을 사용하여 배열을 피벗(pivot)을 기준으로 두 부분으로 나눈 후 각 부분을 재귀적으로 정렬한다.`  
시간복잡도  
퀵 정렬의 시간 복잡도는 O(n log n)이며 최선, 평균은 동일하고 최악의 경우 O(n^2)이다.  
삽입 정렬과는 반대로 퀵 정렬은 이미 정렬된 데이터라면 매우 비효율적으로 작용한다.  

공간복잡도  
퀵 정렬은 정렬을 위해 평균적으로 O(log n)만큼의 memory를 필요로한다. 이는 재귀적 호출로 발생하는 것이며, 최악의 경우 O(n)의 공간복잡도를 보인다.  

---

* 병합정렬은 LikedList를 사용해서 구현하면 더 효율적이다. O

병합정렬  
`병합 정렬은 분할 정복(Divide and Conquer) 방식을 사용하여 배열을 반으로 나눈 후 각 부분을 재귀적으로 정렬하고, 나중에 병합하여 정렬된 배열을 생성한다.`  
![1](https://github.com/justindevcode/TIL/assets/108222981/99f8bfc9-8cba-4d10-887a-407cc4b84ddf)  
시간복잡도  
병합 정렬의 시간 복잡도는 O(n log n)이며, 최선, 평균, 최악의 경우 모두 동일하다.  
데이터를 정확히 반으로 나누고 정렬하기 때문에 항상 일정한 시간 복잡도를 유지하므로 퀵 정렬의 한계점을 보완할 수 있는 장점  

공간복잡도  
LikedList를 사용시 O(1)  
그외 O(n)  
병합 정렬은 순차적인 비교로 정렬을 진행하므로, LinkedList 정렬이 필요할 때 사용하면 효율적이다.  
만약 배열을 linked list(연결 리스트)로 구현하면 인덱스만 변경되므로 데이터의 이동 및 복사를 하지 않아도 된다.  

```
@ 배열 혹은 리스트로 구현하는 경우

1. 쪼갠다(재귀적으로)
- 나눠지는 집합별에 대해 각각의 startIndex, endIndex를 찾는다.(실제 쪼개지는 것이 아님)

 

2. 정렬하면서 합친다.
- 두 집합을 비교하여 정렬된 상태로 임시 배열에 저장한 뒤, 원본 배열에 이를 다시 옮겨준다.
- 하나만 예시를 들면 위 그림에서 "크기가 8인 정렬"이라고 쓰여있는 부분에서 아래의 과정이 일어난다.
   [2, 10, 30, 69] [8, 16, 22, 31] 두 집합을 정렬된 상태로 임시 배열b에 [2, 8, 10, 16, 22, 30, 31, 69] 저장하고
   이를 다시 원본 배열a에 할당해줘야함. a[i] = b[i]
   이 과정은 2개의 집합이 1개의 집합으로 합쳐질 때 항상 이뤄지므로 오버헤드가 상당하다.

 

@ 연결 리스트로 구현하는 경우

1. 쪼갠다(재귀적으로)

- 각 단계에서 링크드 리스트를 2개의 링크드 리스트로 분리시켜준다.  
   head -> Node1 -> Node2 -> Node3 -> Node4 이면 
   head -> Node1 -> Node2, head2 -> Node3 -> Node4 으로.

2. 정렬하면서 합친다.
- 노드의 Next를 변경해주는 것을 통해 정렬하면서 합친다.
- 합치는 과정은 head가 2개였던 것을 1개로 합쳐주며 이뤄짐.
```

---

* 힙 정렬은 최악의 경우에는 시간복잡도가 O(n^2)이 나올 수 있다. X

힙 정렬  
![1](https://github.com/justindevcode/TIL/assets/108222981/caf91cef-6d98-48d4-b765-48ef57bdff28)  
`힙 정렬은 힙 자료구조를 사용하여 정렬을 수행하는 방식으로, 주어진 배열을 최대 힙 또는 최소 힙으로 구성한 후 힙에서 원소를 하나씩 꺼내 정렬한다.`  
시간복잡도  
힙 정렬의 시간 복잡도는 O(n log n)이며, 최선, 평균, 최악의 경우 모두 동일하다.  
완전 이진트리를 사용한다.  

공간복잡도  
O(1)  

---

![1](https://github.com/justindevcode/TIL/assets/108222981/624a08bc-7946-4d4a-9291-2bc15dd8e6a7)


#### 참조
https://velog.io/@ajm0718/%EC%A0%95%EB%A0%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%97%90-%EB%8C%80%ED%95%9C-%EB%B3%B5%EC%9E%A1%EB%8F%84  

---
## 20240507
### 아파치 카프카 2

지난 시간과 다르게 좀더 새부적인 내용을 알아보자  

#### Kafka Cluster  
카프카는 `Kafka Cluster`안에 `Broker`가 여러개 존재한다고 생각하면된다. 이는 데이터 유실방지 때문인데 `Broker`가 하나의 카프카 서버인데 이를 여러개 띄워 서로의 복사본을 가지고 있는 하나의 `Kafka Cluster`가 된다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/2c1df81c-6a62-433d-acb9-0360deb6425b)  

클러스터 내에서 파티션 각각이 서버에 분산된 형태로 나누어 처리될 수 있다고 표현한 이유는 위 그림이 담고 있다. 기본적으로 토픽 내의 파티션이 추가 생성되면 카프카는 등록된 브로커 서버에 분산된 형태로 새 파티션을 배치(?) 시켜주는 개념이다.  
1개 파티션은 리더와 팔로워가 존재한다. 카프카에서는 파티션마다 replica 를 설정해서 이를 실현하며, 사실 이 특징은 브로커 노드의 장애나 fail 에 대비하는 것에 목표를 두었다.  
`replication factor` 가 3이라면, 클러스터 내에 최소 3대의 브로커 서버 등록이 보장되어야 하며 1개의 리더 노드와 2개의 팔로워 노드로 구성될 수 있다.  

replication factor 가 2 이상으로 설정된 파티션은 반드시 팔로워 노드가 존재하는 데, 리더 노드는 클라이언트(Producer, Consumer) 를 통한 데이터 write (쓰기 연산) / read (읽기 연산) 를, 팔로워 노드는 데이터 복제만을 담당한다.  
즉, 리더 노드에 발행된 데이터를 팔로워 노드가 복제만 해가는 구조이다. 이걸 부하 분산의 관점에서 다시 바라보면, 결국 각 파티션의 리더가 클러스터 내의 브로커들에 균등하게 분배되도록 설계되었다는걸 알 수 있다.  

여기서 설정하게 되는 replication factor 는 파티션 각각에 대해 카프카 클러스터 내의 모든 브로커에 동일한 값으로 설정되어야 하는 값이다. 이 값으로 클러스터 내의 몇 개 브로커에 데이터를 저장 / 복제해둘지를 결정한다.  

토픽 내의 파티션은 이렇게 리더 - 팔로워 구조를 유지하고 있다가, 리더 노드로 등록된 브로커가 죽었을 때 팔로워들 중 하나를 다시 리더로 선출하여 데이터의 유실을 방지하고 복구를 진행할 수 있도록 한다. 따라서 N+1 의 replication factor 를 가진 TopicPartition 이라면 N 번의 장애까지 견딜 수 있다. 개인적으로는 카프카가 가용성을 확보하기 위한 수단 중 하나라고 생각한다.  

#### Producer

Producer 는 카프카 클러스터에서 호스팅하는 토픽에 데이터 입력을 요청할 수 있는 클라이언트이다. 정확히는, 토픽 파티션의 리더 노드에게 쓰기 연산에 대한 요청을 보내서 지정된 토픽 파티션에 메시지를 발행하는 주체이다.  
이 때 메시지에는 header (String key, byte[] value), key, value 가 포함된다. 만약 key 를 별도로 지정하지 않는다면 카프카는 라운드로빈 방식으로 파티션에 메시지를 분배하여 발행하게 된다.

#### Producer config(설정할 수 있는값들) 중 주요한것  

* acks (default = 1)
	* 개인적으로는 카프카에서 QoS 를 보장하는 수단이 될 수 있는 옵션이라 생각한다.
	* Producer 가 토픽 파티션의 리더 노드에게 메시지 발행을 요청한 후, 해당 요청을 완료하는 것에 대한 기준이 되는 설정이다.
	* acks 의 값이 클수록 퍼포먼스는 느려지고 안정성은 증가할 수 있다.
	* acks = 0 이면 프로듀서가 어떤 acks 응답도 기다리지 않는다. 즉, 카프카 서버가 데이터를 받았는지를 보장하지 않았으므로 전송 (요청) 실패에 따른 재요청도 없다.
	* acks = 1 이면 리더 노드가 메시지 발행 요청을 받은 건 확인하지만, 팔로워 노드가 그걸 복제해갔는지에 대해선 확인하지 않는다. 속도와 안정성 측면에서 가장 많이 쓰인다고 한다.
	* acks = all (-1) 이면 리더 노드 뿐만 아니라 모든 팔로워 노드들의 응답을 기다리므로 데이터가 손실되지 않는다.
* max.in.flight.requests.per.connection (default = 5)
	* 공식문서 정의 : The maximum number of unacknowledged requests the client will send on a single connection before blocking.
	* 프로듀서가 입력하는 데이터의 최소 단위는 batch.size 에 지정한 값이 되는데, 이 옵션으로는 발행될 이벤트에 대한 batch 단위의 순서를 보장할 수 있다. (물론 1개 파티션 내에서의 이야기가 되겠다.)
	* 예를 들어, 2개의 batch 레코드 셋이 1개 파티션으로 발행 요청된 상황이 있다고 가정해보자. 이 상황에서 첫번째에 발행된 레코드 셋은 요청에 실패하여 retry 를 시도하고 있지만, 두번째 발행된 레코드 셋은 곧바로 성공하였다.
	* 주어진 상황에서 max.in.flight.requests.per.connection 이 1 보다 큰 값으로 설정되어 있다면, 두번째 batch 레코드셋이 먼저 발행에 성공할 수 있게 된다.
	* max.in.flight.requests.per.connection 을 1 로 설정한다면, 두번째 레코드셋은 첫번째에서 retry 를 성공할 때까지 발행 요청을 진행하지 않으므로, 파티션 내에서 프로듀서가 이벤트 발행을 요청한 순서를 보장한다. 다만 개인적으로는, 이렇게 되면 flexible 한 가용성을 충족시켜주진 못할 것 같다.

#### Consumer

![1](https://github.com/justindevcode/TIL/assets/108222981/7c3383b9-e506-4084-ba73-154fc77c1b32)  
Consumer 를 설명할 때에 빠질 수 없는 건 consumer group 이다. 1개 토픽에 대해서 여러개 컨슈머 그룹이 각각 다른 목적으로 존재할 수 있으며, 1개 토픽에 발행된 데이터는 그 발행 횟수에 관계 없이 여러 컨슈머 그룹이 각자의 목적에 맞게 처리하기 위해 여러번 읽어갈 수 있다.   

덧붙이자면, offset 을 지정한다면 1개 컨슈머 그룹이 같은 토픽 내의 데이터를 n 번 읽어갈 수 있으며 이 또한 발행 횟수와는 관계 없다.  

이는 카프카가 가진 스토리지 특성과도 관련이 있다. 카프카는 컨슈머에 의해 처리된 메시지를 곧바로 삭제하지 않고 그대로 저장했다가 수명이 지나면 삭제처리를 하는 시스템이다. 따라서 메시지 처리 도중에 문제가 생겼거나, 로직에 변경이 생겼을 경우 컨슈머가 처음부터 데이터를 다시 처리할 수도 있다.  

다시 offset 과 컨슈머 그룹에 대한 이야기로 돌아와보겠다. 각 컨슈머 그룹은 토픽 파티션에 대한 offset 정보를 관리하는데, 컨슈머 클라이언트 각각은 config 옵션 중 auto.offset.reset 을 통해 해당 offset 정보를 참조하여 데이터를 읽고 처리할 수 있다. 옵션 값으로 none 이 지정되면 컨슈머 그룹이 저장하고 있는 offset 정보를 읽어와서 해당 offset 부터 데이터를 읽어올 수 있게 된다.  


#### Consumer config(설정할 수 있는값들) 중 주요한것  

* max.poll.interval.ms
* 컨슈머는 실제 데이터 polling 을 진행하고 있지 않을 때에도 컨슈머 그룹 멤버로서 지속적으로 존재하기 위해 카프카 클러스터 (coordinator) 에게 heartbeat request 를 전송한다.
* 하지만 컨슈머 클라이언트가 클러스터에 heartbeat 는 보내지만 실제 데이터는 읽어가지 않는 상황이 길어질 경우, 파티션이 무한정으로 점유될 수 있다.
* 이 같은 상황을 방지하기 위해 이 설정을 추가하여, 해당 값 (시간) 이 만료될 때 까지 message polling 을 시도하지 않는다면 컨슈머 그룹 구성에 변화가 생겼음 (해당 컨슈머가 장애라고 판단하고 그룹에서 제외시킬 수 있다) 을 인지할 수 있도록 한다.



레코드 리스너(MessageListener): 단 1개의 레코드를 처리합니다. (스프링 카프카 컨슈머의 기본 리스너 타입)
배치 리스너(BatchMessageListener): 한 번에 여러 개 레코드들을 처리합니다.

* max.poll.size (배치 리스너 일때?)
	* 컨슈머 클라이언트가 특정 토픽을 구독한 뒤부터 poll 작업이 이루어질 수 있는데, 이 때 poll() 한 번에 따라 가져올 수 있는 최대 레코드 수를 지정한다.
 	* 이 때 카프카를 사용하는 애플리케이션에서는 poll 메소드를 호출해서 레코드를 가져오고, 특정 스레드에서 그걸 전부 처리해준 뒤 다시 poll 메소드를 호출해 새로운 레코드를 가져오게 된다.
	* 이 속성의 기본값은 500이므로, 기본 값을 사용하는 경우 poll 메소드로 한번에 최대 500개 레코드까지 가져올 수 있다.

max.poll.size 속성을 통해 컨슈머의 poll 메소드 호출 간격을 결정할 수 있다. 특히 이 호출 간격이 길어진다는 것은 리벨런싱과 직접적으로 연관이 되기 때문에 배포되는 서비스의 경우 중요성을 인지할 필요가 있다.   

#### 참조
https://www.geuni.tech/ko/kafka/kafka_introduce_install_cluster/  
https://velog.io/@hyeondev/Apache-Kafka-%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90  
https://zeroco.tistory.com/105  
https://hanseom.tistory.com/174#recentComments  
https://devoong2.tistory.com/entry/Kafka-%EC%BB%A8%EC%8A%88%EB%A8%B8%EC%9D%98-Poll-%EB%8F%99%EC%9E%91%EA%B3%BC%EC%A0%95-%EB%B0%8F-maxpollrecords-%EC%97%90-%EB%8C%80%ED%95%9C-%EC%98%A4%ED%95%B4  
https://devlog-wjdrbs96.tistory.com/442  

---
## 20240508
### 스프링 Conditional

#### 자동 구성 직접 만들기
실시간으로 자바 메모리 사용량을 웹으로 확인하는 예제이다.  

```java
@Slf4j
public class MemoryFinder {
 
 public Memory get() {
 long max = Runtime.getRuntime().maxMemory();
 long total = Runtime.getRuntime().totalMemory();
 long free = Runtime.getRuntime().freeMemory();
 long used = total - free;
 return new Memory(used, max);
 }
 @PostConstruct
 public void init() {
 log.info("init memoryFinder");
 }
}
```
JVM에서 메모리 정보를 실시간으로 조회하는 기능이다.  
max 는 JVM이 사용할 수 있는 최대 메모리, 이 수치를 넘어가면 OOM이 발생한다.  
total 은 JVM이 확보한 전체 메모리(JVM은 처음부터 max 까지 다 확보하지 않고 필요할 때 마다 조금씩 확보한다.)  
free 는 total 중에 사용하지 않은 메모리(JVM이 확보한 전체 메모리 중에 사용하지 않은 것)  
used 는 JVM이 사용중인 메모리이다. ( used = total - free )  
(크게 중요한 부분은아님 참고만 이런거로 컨트롤러에서 자료출력 예제란것)  

```java
@Configuration
public class MemoryConfig {
 @Bean
 public MemoryController memoryController() {
 return new MemoryController(memoryFinder());
 }
 @Bean
 public MemoryFinder memoryFinder() {
 return new MemoryFinder();
 }
}
```
위의 기능을 아에 다른 라이브러리나 기능이라고 생각해서 내가 개발할곳에서 사용하고싶음 -> 기본적으로 내가 직접 빈등록해줘야함  

#### 빈등록 개선법 @Conditional

```java
@Slf4j
public class MemoryCondition implements  {
 @Override
 public boolean matches(ConditionContext context, AnnotatedTypeMetadata
metadata) {
 String memory = context.getEnvironment().getProperty("memory");
 log.info("memory={}", memory);
 return "on".equals(memory);
 }
}
```
`Condition`인터페이스를 구현해서 설정정보에 `memory=on`이라고 적힐때만 작동하게 할 수 있음  

``` java
@Configuration
@Conditional(MemoryCondition.class) //추가
public class MemoryConfig {
 @Bean
 public MemoryController memoryController() {
 return new MemoryController(memoryFinder());
 }
 @Bean
 public MemoryFinder memoryFinder() {
 return new MemoryFinder();
 }
}
```
빈등록하는곳에 `@Conditional(MemoryCondition.class)`이렇게 넣어주면 `memory=on`이라고 적힐때만 작동  
MemoryCondition 의 matches() 를 실행해보고 그 결과가 true 이면 MemoryConfig 는 정상 동작한다  
`#java -Dmemory=on -jar project.jar`이런식으로 동작할떄 넣는 그것임  

참고로 `getEnvironment`이거 함수 이용해서  
VM Options: java -Dmemory=on -jar project.jar
Program arguments: (-- 가 있으면 스프링이 환경 정보로 사용) java -jar project.jar --memory=on
application.properties파일: memory=on  
이런 다양한 방면의 설정정보 저 함수로 뽑을 수 있음  

#### @Conditional - 다양한 기능

```java
@Configuration
//@Conditional(MemoryCondition.class) //추가
@ConditionalOnProperty(name = "memory", havingValue = "on") //추가
public class MemoryConfig {
 @Bean
 public MemoryController memoryController() {
 return new MemoryController(memoryFinder());
 }
 @Bean
 public MemoryFinder memoryFinder() {
 return new MemoryFinder();
 }
}
```
`@Conditional`이걸 자주 사용하니깐 부트에서 아에 세부사항별로 미리 만들어둔 어노테이션 있음
`@ConditionalOnProperty(name = "memory", havingValue = "on")` 이렇게 쓰면 따로 클레스 선언할 필요없이 바로 했던것 적용됨  

예시

@ConditionalOnClass , @ConditionalOnMissingClass  
클래스가 있는 경우 동작한다. 나머지는 그 반대  
@ConditionalOnBean , @ConditionalOnMissingBean  
빈이 등록되어 있는 경우 동작한다. 나머지는 그 반대  
@ConditionalOnProperty  
환경 정보가 있는 경우 동작한다.  
@ConditionalOnResource  
리소스가 있는 경우 동작한다.  
@ConditionalOnWebApplication , @ConditionalOnNotWebApplication  
웹 애플리케이션인 경우 동작한다.  
@ConditionalOnExpression  
SpEL 표현식에 만족하는 경우 동작한다  

---
## 20240514
### 스프링 순수라이브러리와 자동설정  

#### 라이브러리제작
라이브러리만들 녀석의(메모리출력) 프로젝트
```
plugins {
 id 'java'
}
group = 'memory'
sourceCompatibility = '17'
repositories {
 mavenCentral()
}
dependencies {
 implementation 'org.springframework.boot:spring-boot-starter-web:3.0.2'
 compileOnly 'org.projectlombok:lombok:1.18.24'
 annotationProcessor 'org.projectlombok:lombok:1.18.24'
 testImplementation 'org.springframework.boot:spring-boot-starter-test:3.0.2'
}
test {
 useJUnitPlatform()
}
```
부트를 쓰면 실행가능jar가 만들어지기 때문에 순수 내 코드만 jar로 만들기위해 부트플러그인 안씀  

예시 메모리 코드
```java
@Slf4j
public class MemoryFinder {
 public Memory get() {
 long max = Runtime.getRuntime().maxMemory();
 long total = Runtime.getRuntime().totalMemory();
 long free = Runtime.getRuntime().freeMemory();
 long used = total - free;
 return new Memory(used, max);
 }
 @PostConstruct
 public void init() {
 log.info("init memoryFinder");
 }
}

.
.
.
등등 컨트롤러까지 코드 있음 
```

그리고 이 프로젝트를 `./gradlew clean build` 빌드하면 순수 jar가 만들어짐

#### 라이브러리 사용

`project-v1/libs`폴더 생성 -> 만든 `memory-v1.jar` 복붙

```
plugins {
 id 'org.springframework.boot' version '3.0.2'
 id 'io.spring.dependency-management' version '1.1.0'
 id 'java'
}
group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'
configurations {
 compileOnly {
 extendsFrom annotationProcessor
 }
}
repositories {
 mavenCentral()
}
dependencies {
implementation files('libs/memory-v1.jar') //추가
 implementation 'org.springframework.boot:spring-boot-starter-web'
 compileOnly 'org.projectlombok:lombok'
 annotationProcessor 'org.projectlombok:lombok'
 testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
tasks.named('test') {
 useJUnitPlatform()
}
```
사용할곳에 `implementation files('libs/memory-v1.jar')`추가

여기서 문제는 이 라이브러리를 사용하려면 내가 해당 라이브러리의 클래스를 빈으로 등록해줘야함...

```java
@Configuration
public class MemoryConfig {
 
 @Bean
 public MemoryFinder memoryFinder() {
 return new MemoryFinder();
 }
 
 @Bean
 public MemoryController memoryController() {
 return new MemoryController(memoryFinder());
 }
}
```
이게맞나... 가져가 쓴 라이브러리의 어떤 클레스를 어떤식으로 등록해줘야하는지 알아야함..


#### 라이브러리 자체에서 자동구성 구현

라이브러리 프로젝트에 자동구성추가

```java
@AutoConfiguration
@ConditionalOnProperty(name = "memory", havingValue = "on")
public class MemoryAutoConfig {
 @Bean
 public MemoryController memoryController() {
 return new MemoryController(memoryFinder());
 }
 @Bean
 public MemoryFinder memoryFinder() {
 return new MemoryFinder();
 }
}
```
vm옵셥으로 memory=on 설정되어야지 빈등록되도록 해둠  

하나 더 해줘야하는데 이 설정파일을 `src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`라는 폴더속파일에
`memory.MemoryAutoConfig`라고 설정파일을 써줘야함  

스프링에서 실제 라이브러리들 사용할때 위의 파일위치에서 `memory.MemoryAutoConfig`이 클래스를 읽어서 빈등록해면 되겠구나 하고 확인하는 파일임  

`./gradlew clean build`이 라이브러리를 다시 빌드해서  

사용할프로젝트 `project-v2/libs`에 `memory-v2.jar`를 다시 복붙하고 `implementation`추가   
```
dependencies {
 implementation files('libs/memory-v2.jar') //추가
 implementation 'org.springframework.boot:spring-boot-starter-web'
 compileOnly 'org.projectlombok:lombok'
 annotationProcessor 'org.projectlombok:lombok'
 testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

이러고 `@ConditionalOnProperty(name = "memory", havingValue = "on")`이거 없었으면 그냥 아무것도 안해도 등록됐을건데 설정있어서 `-Dmemory=on`추가해고 스프링돌려주면 자동등록완료  

---
## 20240517
### java 메모리 영역 

#### 참조 자료
https://lucas-owner.tistory.com/38  


#### JVM

JVM = 자바의 가상머신 OS에 상관없이 실행할 수 있는것이 장점이다. (JVM 설치만 하면 어떤 운영체제에서든 java 파일을 실행할 수 있다.)  

실행순서  
java 파일을 컴파일러(Compiler)를 통해 .class 파일로 변환한다.  
class 파일을 JVM 의 ClassLoader(클래스로더)에게 보낸다.  
클래스로더에서 JVM 런타임 영역으로 로딩(할당)하여 메모리에 올린다.  

#### JVM의 RunTime Data Area

Java 메모리 영역을 알기 위해선 런타임 데이터 영역에 대해서 알아야한다.  
클래스 로더가 .class 파일을 이 영역에 올리기 때문이다.  
런타임 데이터 영역에는 5가지 영역이 존재한다.  
* Static Area(Method Area)
* Heap Area
* Stack Area
* PC Register
* Native Method Stack

![1](https://github.com/justindevcode/TIL/assets/108222981/530ead3c-3240-4d8f-8be0-3809c1ff27b7)  

#### Java의 메모리 영역
자바 프로그램을 실행 하게되면 JVM(Java Virtual Machine)은 OS 로 부터 메모리를 할당 받는다, 할당 받은 메모리를 자바 프로그램에 맞게 여러개의 영역으로 나누어 사용하게 된다.  

JVM의 메모리는 크게 3가지로 이루어져 있다.  
Heap 영역.  
Stack 영역.  
Static(Method) 영역.  

#### Java의 변수 종류

변수는 선언 위치에 따라 구분짓게 된다.  
4가지의 종류가 존재한다. (클래스 변수, 인스턴스 변수, 지역변수, 매게변수)  

```java
public class Variable { 

	public static int age = 20; // 클래스 변수(전역 변수)
    
    int height = 60; // 인스턴스 변수(전역 변수)
    
    public static void main(String[] args) { // 매개변수(파라미터)
 		int size = 50; // 지역변수
        
    }
}
```

|변수종류|선언위치|설명|생성시기|소멸시기|저장메모리|
|---|---|---|---|---|---|
|클래스 변수 (Static variable)|클래스 영역|static 키워드가 붙고 여러 객체에서 공통으로 사용 할 때 사용|클래스가 메모리에 올라갈때|프로그램 종료 시|Static|
|인스턴스 변수(Instance variable)|클래스 영역|클래스 영역에서 static이 아닌 변수|인스턴스가 생성될 때|인스턴스 소멸시|Heap|
|지역 변수(Local variable)|메서드 영역|메서드 내부에서 선언된다. 초기 값을 지정해야 사용가능.|블록 내에서 변수의 선언문이 실행 될 때|블록을 벗어날 때|Stack|


#### Static (Method) 영역

Static 영역 혹은 Method 영역이라고 불린다. 클래스 변수나, static 으로 선언된 것들이 해당 메모리 영역에 저장된다.  

* JVM이 실행될 때 Class 가 로딩될 때 생성.
* Class의 정보, Static 변수(클래스 변수), 생성자(Constructor), 메소드(Method)와 같은 것들을 저장한다.
* Static 영역에 있는 것은 어디서든 접근 가능 하다.
* JVM이 종료 시(프로그램이 종료 시) 메모리에서 해제 된다. 즉 프로그램이 종료되기 전까진 메모리 상에 존재하게된다. 그렇기 때문에 어디서든 접근이 가능한 것이며, 무분별 하게 사용될 경우 메모리 부족 현상이 발생할 수 있다.

#### Heap 영역

인스턴스를 생성할 때 사용되는 메모리 영역이다.  
참조형 데이터 객체의 실제 데이터가 저장되는 공간이다. Stack 영역에서 실제데이터가 존재하는 Heap 영역의 참조값을 가지고 있다.  
new 키워드로 인스턴스를 생성 할 때, Heap 영역에는 생성된 객체가 저장, Stack 영역에서 생성된 객체에 대한 주소 값(Reference)이 저장된다.  


* new 를 사용해 객체를 생성할 때 저장된다.
* 참조형 데이터 타입이 저장된다. (String, 배열(array), enum, class, interface), Object
* Heap 영역의 데이터들을 가르키는 Reference(참조 주소)는 Stack영역에 적재된다.
* Reference를 통해서만 Heap 영역의 데이터들에 접근, 핸들링 할 수 있다.
* 호출이 종료되도 삭제되지 않는다. -> GC(가비지 컬렉터)에 의해 메모리에서 해제된다.
* 쓰레드가 몇개가 존재하든, 단 하나의 영역만 존재한다. (Stack 영역의 경우 쓰레드 별로 1개씩 생성된다.)

![1](https://github.com/justindevcode/TIL/assets/108222981/45416e7f-1b36-481a-b4e7-5b50a39e73bd)  

* String name = "lucas"; 이라는 줄이 있을때.
* "lucas" 는 Heap 영역에 저장.
* name 부분은 Stack 영역에 저장. (name 은 주소를 갖고 있으며 -> Heap 영역의 "lucas" 를 가르킨다.)

#### Stack 영역

* 기본 자료형(원시 자료형, Primitive type), 지역변수, 매개변수가 저장되는 메모리. (int, double, boolean, byte)
* 메서드 내부의 기본자료형에 해당하는 변수 적재.
* Heap 영역에 생성된 데이터의 참조값이 할당됨. -> *그림 : Heap, Stack 저장되는부분 참고.
* 메소드가 호출될 때 메모리에 할당, 메서드 종료시 메모리에서 삭제됨.
* 자료구조 Stack의 구조이다, LIFO(Last In First Out)
* 각 Thread 마다 자신만의 Stack 을 가진다. (1:1) - (Thread : Stack)
* Thread는 내부적으로 Static, Heap, Stack 영역을 가진다.
* Thread는 다른 Thread에 접근 할 수 없지만, static, Heap 영역을 공유하여 사용 가능.

---
## 20240518
### 스프링 자동구성의 이해

우리는 `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 이런 긴 파일에 설정 클레스를 넣어주니 그것이 등록되었다.  
어떤 과정을 거치기에 가능 할까?  

#### 동작순서이해

```java
@SpringBootApplication
public class AutoConfigApplication {
 public static void main(String[] args) {
 SpringApplication.run(AutoConfigApplication.class, args);
 }
}
```
스프링 부트의 메인 함수를 보면  
ran() -> AutoConfigApplication이 클래스 바로 설정정보로 사용 -> SpringBootApplication어노테이션 ->   

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes =
TypeExcludeFilter.class),
@Filter(type = FilterType.CUSTOM, classes =
AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {...}
```
@EnableAutoConfiguration ->  

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {…}
```
@Import -> AutoConfigurationImportSelector.class???

쭉 따라 들어가보면 Import가 있는데 여기에는 원래 설정클레스만 넣었는데 ImportSelector란것이 있다.  

#### ImportSelector

기존방식(정적방식)  
```java
@Configuration
@Import({AConfig.class, BConfig.class})
public class AppConfig {...}
```
원래는 이렇게 사용했다. 근데 저 클래스 파일을 동적으로 바꾸고 싶으면 어떻게 해야할까?  

이때 사용하는것이 `ImportSelector`
```java
package org.springframework.context.annotation;
public interface ImportSelector {
String[] selectImports(AnnotationMetadata importingClassMetadata);
 //...
}
```
이런 인터페이스 인데 어려운것 없다. `selectImports`함수가 실행되고 그 리턴값의 `String[]`들 이름으로 파일을 찾아 그 설정정보를 실행하는것이다.

예시
```java
public class HelloImportSelector implements ImportSelector {
 @Override
 public String[] selectImports(AnnotationMetadata importingClassMetadata) {
 return new String[]{"hello.selector.HelloConfig"};
 }
}
```
여기서는 간단히 스트링값으로 설정정보 클래스의 위치를 넘겼는데 안에서 if문을 쓰던 해서 내맘데로 바꿀 수 있다.  

즉 아까 위의 @EnableAutoConfiguration ->@Import -> AutoConfigurationImportSelector.class는
그냥 `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`의 파일을 자바동적코드로 읽어서 스트링값을 취합한다음 그 설정클래스를 등록하는것이다.  

정말 상세하게는 다른 순서나 내부 동작 제거할거는 제거하고 이런 과정이 있긴하지만 이정도만 이해해도 충분하다.  

#### 여담

@AutoConfiguration도 설정파일이라 @Configuration이 있긴한데 일반 스프링 설정과 라이프사이클이 다르기 때문에 컴포넌트 스캔의 대상이 되면 안된다 그래서 `resources/META-INF...`이런 파일에 명시를 따로 해주는것 컴포넌트 스캔에서는 `@AutoConfiguration` 을 제외하는 `AutoConfigurationExcludeFilter` 필터가 포함되어 있다.  

`@AutoConfiguration`을 직접 사용할일은 정말 라이브러리를 직접 만들때밖에없다. 하지만 이 빈이 왜 자동으로 등록됐는지 확인하려면 역으로 찾는 과정이 필요한데 이때 코드는 읽을 수 있어야한다. 그래서 이정도 공부해두는것  
