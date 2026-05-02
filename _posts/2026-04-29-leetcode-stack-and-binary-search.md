---
category: "LeetCode"
---
## Stack & Bianry Search

### 1. Valid Parentheses (Easy)

- if else 를 쓸 때는 조건을 유심히 봐야 한다. 불필요한 if else 문을 쓸 수 있고 이건 더 긴 런타임을 유발한다.
- 예를 들어, if not stack or stack[-1] ≠ parentheses[p] 라는 조건을 통과하면 무조건 리턴 되므로 이 조건 다음에 stack.pop()을 써도 괜찮다.

```python
class Solution:
    def isValid(self, s: str) -> bool:
        parentheses = {
            ")":"(",
            "]":"[",
            "}":"{"
        }
        stack = []

        for p in s:
            if p in "({[":
                stack.append(p)
            else:
                if not stack or stack[-1] != parentheses[p]:
                    return False
                stack.pop()
        
        return len(stack) == 0
```

### 2. Min Stack (Medium)

- 파이썬에 대해 더 알게 되었고, 동시에 굉장히 재밌었다!
- class 안에 전역 변수를 만들 때 아무생각 없이 class MinStack 바로 안에 stack = [] 을 선언했는데, 그러면 MinStack 으로 생성되는 모든 인스턴스가 그걸 공유하게 된다. (안돼~~!!) 그러므로 꼭 이 하나의 인스턴스의 stack 이라는 변수라는 걸 알려주기 위해 self.stack = [] 을 __init__() 에 선언해야 하고, 다른 메서드에서 사용할 때도 반드시 그냥 stack 이 아니라 self.stack 이라고 해야 한다.
- getMin() 은 이 스택의 원소들 중 최소값을 리턴해야 한다. 그런데 내가 처음에 생각했던 방안 - push() 될 때마다 min_value 를 갱신하는 것 - 은 엄청난 취약점이 있다. 만약 스택에 [5, 3, 2] 가 들어있었는데 2가 pop 된 후에 getMin 을 한다면? min_value 는 pop 된 숫자도 가질 수 있다!!!
- 해결방안: push() 를 할 때 현재 들어온 값과 이 시점까지의 최소값을 튜플로 묶어서 스택에 넣어준다. 그리고 현재 들어온 값과 바로 아래층에 기록된 최소값 중 더 작은 것을 선택한다!

```python
class MinStack:
    def __init__(self):
        self.stack = []
        
    def push(self, val: int) -> None:
        if not self.stack:
            self.stack.append((val, val))
        else:
            # 방금 들어온 값(val)과 바로 아래층에 기록된 최소값 중 더 작은 것을 선택!
            current_min = self.stack[-1][1]
            # (현재 들어온 값, 이 시점까지의 최소값) 을 튜플로 묶어서 스택에 넣어준다
            self.stack.append((val, min(val, current_min)))

    def pop(self) -> None:
        if self.stack:
            self.stack.pop()

    def top(self) -> int:
        return self.stack[-1][0]

    def getMin(self) -> int:
        return self.stack[-1][1]
```

### 3. Binary Search (Easy)

- iteration 방법으로 구현한 이진 검색이다.
- 여기서 중요한 건, mid 를 어떻게 계산할 것이냐이다.
- 만약 mid = (left + right) // 2를 한다면 stack overflow 가 일어날 가능성이 높다.
- 그래서 “거리”의 관점으로, left 에 (right - left) // 2 를 한 거리를 더해주는 방식으로 stack overflow 를 방지할 수 있다.
- 그리고 while 조건에 left ≤ right 하는 걸 잊지 말자!!!! left < right 라고만 하면 모든 테스트 케이스를 통과하지 못한다. 왜인지는 여러 nums 경우를 생각하면 된다.

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2

            if nums[mid] < target:
                left = mid + 1
            elif nums[mid] > target:
                right = mid - 1
            else:
                return mid

        return -1
```

### 4. Search a 2D Matrix (Medium)

- 이 문제를 처음 봤을 때 나는 이렇게 생각했다. 수직으로 이진 탐색 해서 O(logM), 그 후에 수평으로 탐색해서 O(logN) 을 하면 되려나?
- 좋은 아이디어이지만, 더! 좋은 아이디어가 있다.
- 각 행의 원소 값이 아래로 갈 수록 커지고, 각 열의 원소 값이 오른쪽으로 갈 수록 커진다면, 이 행들을 쭉~ 연결해서 하나의 리스트로 취급할 수 있다! 즉, 예를 들어 3x4 행렬이라면, 12개의 원소를 가진 1차원 배열로 볼 수 있는 것이다.
- 행의 개수는 len(matrix), 열의 개수는 len(matrix[0]) 로 간단히 구할 수 있었다.
- matrix[row][col]해서 target 을 찾을 때 row 는 n을 mid 로 나눈 몫으로, 그리고 col은 n 을 mid로 나누고 남은 나머지로! 구할 수 있다!! 그대~로 이진 탐색을 하면 된다.

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        m = len(matrix)
        n = len(matrix[0])

        left = 0
        right = m*n -1

        while left <= right:
            mid = left + (right - left) // 2
            # 2D가 1D라고 생각하고 mid를 잡고 그거의 행렬의 실제 위치를 계산하자!
            row = mid // n
            col = mid % n

            mid_value = matrix[row][col]

            if mid_value == target:
                return True
            elif mid_value < target:
                left = mid + 1
            else:
                right = mid - 1
        return False   
```
