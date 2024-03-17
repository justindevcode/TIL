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
