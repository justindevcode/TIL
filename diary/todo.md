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
