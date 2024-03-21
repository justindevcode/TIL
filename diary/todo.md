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

