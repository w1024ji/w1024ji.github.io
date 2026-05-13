---
category: "LeetCode"
---
## Graph & DP

### 1. Number of Islands (Medium)

- 섬을 발견하면 개수를 세고, 센 섬을 모두 바다로 바꾸자.
- 지도의 왼쪽 위부터 오른쪽 아래까지 흝어보면서, 1을 발견하면 count ++ 하고, 방금 밟은 땅(1)과 연결된 모든 땅을 찾아 전부 0으로 바꿔준다. (이게 sink() 의 역할이다)
- 이중 for 문으로 모든 칸(r, c)을 순회하면 된다.
- 참고로 sink() 는 DFS 이다!

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def sink(r, c):
            if r<0 or r>=len(grid) or c<0 or c>=len(grid[0]) or grid[r][c] == '0':
                return
            
            # 땅을 밟았다면 0으로 바꿔주기
            grid[r][c] = '0'

            sink(r + 1, c) # 하
            sink(r - 1, c) # 상
            sink(r, c + 1) # 우
            sink(r, c - 1) # 좌

        count = 0
        for r in range(len(grid)):
            for c in range(len(grid[0])):
                if grid[r][c] == '1':
                    count += 1
                    sink(r, c)

        return count
```

### 2. Rotting Oranges (Medium)

- 이전 문제 ‘Number of Islands’에서는 DFS를 이용해 한 번 땅을 밟으면 끝까지 파고들어 연결된 섬을 지웠다. 하지만 이 문제는 오렌지들이 동시에! 주변을 오염시키기 때문에, 그리고 시간을 재야 하기 때문에 BFS를 사용해야 한다. 이를 위해 큐 deque를 사용했다.
- 일단 첫번째 for문에서 0분일 때의 상태를 파악했다. (썩을 귤 몇개? 싱싱한 귤 몇 개?) 그리고 썩을 귤들을 모두 큐에 한꺼번에 넣었다.
- while 문이 BFS 역할을 맡았다. 1분 단위로 시간을 끊어서 처리한다.
- level_size = len(queue) 가 중요한 포인트이다! 현재 큐에 들어있는 썩은 귤의 개수만큼을 돌림으로써, 딱 이번 1분 동안 전염시킬 수 있는 귤만 처리하고 넘어갈 수 있게 만들었다.
- while 문 안에 있는 이중 for 문은 주변의 싱싱한 귤을 썩게 만드는 역할을 맡았다.
- 큐에서 썩은 귤을 하나 꺼내 상하좌우를 살피고, 싱싱한 귤(1)을 발견하면 즉시 오염시켜(2) 중복 방문을 방지한다.
- fresh -= 1 하는 것도 잊지 않기!

```python
from collections import deque

class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        rows, cols = len(grid), len(grid[0])
        queue = deque()
        fresh = 0
        minutes = 0

        # 일단 스캔부터 한다. 썩은 귤은 모두 큐에 넣고, 싱싱한 귤은 총 개수를 센다
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == 2:
                    queue.append((r, c))
                elif grid[r][c] == 1:
                    fresh += 1

        if fresh == 0:
            return 0
                       # 하    # 상     # 우     # 좌
        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

        # 큐가 있고 아직 싱싱한 귤이 있으면
        while queue and fresh > 0:
            level_size = len(queue)
            minutes += 1

            for _ in range(level_size):
                r, c = queue.popleft() # 튜플을 쪼개서 받을 수 있다!
                
                for dr, dc in directions:
                    nr, nc = r + dr, c + dc

                    if 0<=nr<rows and 0<=nc<cols and grid[nr][nc] == 1:
                        grid[nr][nc] = 2
                        fresh -= 1

                        queue.append((nr, nc)) # 새롭게 썩은 귤을 큐에 넣는다
        
        return minutes if fresh == 0 else -1
```

### 3. Climbing Stairs (Easy)

- 이 문제는 이산수학 수업에서도 자주 본 아주 유명한 문제이다.
- 포인트는, 처음부터 끝까지 모든 경우의 수를 직접 세는 게 아니라 도착하기 바로 직전의 상황을 상상하면 바로 해결이 된다!
- n 층까지 가는 방법의 수는 = n-1 층까지 가는 방법의 수 + n-2 층까지 가는 방법의 수 이다.
- 이를 점화식으로 표현하면 dp[i] = dp[i - 1] + dp[i - 2] 이다.
- dp 배열을 준비하고 3층부터 n층까지 그 결과를 적어두면 된다.

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        # 예외 처리: 1층이나 2층은 그대로 반환
        if n <= 2:
            return n
            
        # n층까지 기록할 수 있는 배열 만들기
        dp = [0] * (n + 1)
        
        dp[1] = 1
        dp[2] = 2
        
        # 3층부터 n층까지 차근차근 올라가기
        for i in range(3, n + 1):
            dp[i] = dp[i - 1] + dp[i - 2]
            
        return dp[n]
```

### 4. Coin Change (Medium)

- 거스름돈 문제이다. 가장 적은 동전의 개수로 거슬러 줘야 한다.
- 아이디어는 이렇다. ‘방금 낸 동전 하나를 빼고 생각해보자’. 만약, 11원을 만들어야 하는데 내게 1원, 2원, 5원짜리 동전이 있다면 총 세가지의 경우 중에서 가장 동전의 개수가 적은 경우이다.
    1. 1원짜리 동전을 마지막으로 쓴 경우 → (10원을 만드는 최소 동전들) + 1개
    2. 2원짜리 동전을 마지막으로 쓴 경우 → (9원을 만듦) + 1개
    3. 5원짜리 동전을 마지막으로 쓴 경우 → (6원을 만듦) + 1개
- 이걸 점화식으로 표현하면 dp[a] = min(dp[a], dp[a - coin] + 1) 이다.
- 처음에 dp 배열의 모든 칸을 큰 값으로 채워두고 이중 for문으로 bottom-up을 해준다.
- 그리고 마지막 리턴문에서 배열 원소의 값이 amount+1 으로 그대로라면, 가진 동전들로는 그 금액을 만들 수 없다는 뜻이니까 -1을 반환한다.

```python
from typing import List

class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        dp = [amount + 1] * (amount + 1)
        
        dp[0] = 0
        
        for a in range(1, amount + 1):
            for coin in coins:
                if a - coin >= 0:
                    dp[a] = min(dp[a], dp[a - coin] + 1)
                    
        return dp[amount] if dp[amount] != amount + 1 else -1
```
