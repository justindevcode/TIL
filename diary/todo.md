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
