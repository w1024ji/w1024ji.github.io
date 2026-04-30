## Pointers & Sliding Windows
### 1. Valid Palindrome (Easy)

- Palindrome: 앞으로 읽나, 뒤로 읽나 → 같으면 palindrome 이라고 한다.
- 여기서는 ‘removing all non-alphanumeric characters’ 라는 조건이 있었으므로 **.isalnum()** 을 쓰는 게 중요했다.
- 그리고 모든 알파벳들을 lower 로 일관되게 해준다. **.lower()**
- **two pointer** 를 사용하는 게 핵심! 양 끝에서 left, right 을 잡아주고 점점 가운데로 오면서 (isalnum() 이 아닌 것들은 무시.) 양쪽이 대칭으로 같다면 → True. 하나라도 아니라면 False!

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        left = 0
        right = len(s) - 1
        while left < right:
            while left < right and not s[left]**.isalnum()**:
                left += 1
            while left < right and not s[right]**.isalnum()**:
                right -= 1

            **if s[left].lower() != s[right].lower():
                return False
            left += 1
            right -= 1**
        return True
```

### 2. Best Time to Buy and Sell Stock (Easy)

- 생각보다 헷갈려서 어려웠다.
- 변수를 총 3개 잡아야 한다. min_price, current_profit, max_profit
- min_price: 여태까지의 경험한 최소 가격
- current_profit: 현재 가격에서 최소 가격을 뺀 이익 (price - min_price)
- max_profit: 내가 경험한 이익 중에서 가장 큰 이익 max()
- 결국, max_profit 을 업데이트하면서 가장 컸던 이익을 리턴한다.
- 변수를 새로 만드는 거에 너무 부담갖지 말자! for 문이나 while 문 아니면 변수 생성은 시간 복잡도를 잡아먹지 않아!

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = 100000
        max_profit = 0 
        for price in prices:
            min_price = min(min_price, price)
            current_profit = price - min_price
            max_profit = max(current_profit, max_profit)

        return max_profit
```

### 3. 3Sum (Medium)

- 어려웠던 문제! 하지만 재밌었다~
- 일단, for 문을 시작하지 전에, **nums 를 sort() 해주면 nums 자체가 정렬되므로**, 작은 값은 왼쪽, 큰 값은 오른쪽에 정렬된다. (이는 나중에 i 랑 j 를 옮기는 데 효과적이다)
- 일단, nums 리스트의 세 숫자를 잡고 있는 포인터가 필요하고, 그 중에서 기준 포인터는 k 이므로 for k in range() 를 리스트 왼쪽에서부터 차례로 돌리고, k를 포함한 왼쪽을 제외한 오른쪽의 구역을 가지고 i 와 j 를 포인터로 잘 써야 한다.
- nums[k] + nums[i] + nums[j] 을 total 이라고 두고, **이것이 0이라면 이 k, i, j 그룹이 정답이고, 미리 만들어둔 results에 리스트 형태로 append() 해준다.**
- 만약에 total 이 0보다 작다면, i 가 작은 놈이므로, 크게 만들어야 하니까 i 만 += 해준다.
- 만약에 total 이 0보다 크다면, j 가 큰 놈이고, 얘를 작게 만들어야 하니까 j 만 -= 해준다.
- (왜냐하면 k, i, j 를 적절하게 맞춰서 0을 만들어야 하기 때문이다)

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        **nums.sort()**
        length = len(nums)
        results = []

        for k in range(0, length - 2):
            **# 1. 기준점 k가 이전 숫자와 똑같다면 건너뛰기 (중복 방지)
            if k > 0 and nums[k] == nums[k - 1]:
                continue**

            i = k + 1
            j = length - 1
            
            while i < j:
                total = nums[k] + nums[i] + nums[j]
                
                if total == 0:
                    # 정답 추가!
                    results.append(**[nums[k], nums[i], nums[j]]**)
                    
                    **# 2. i와 j도 중복된 숫자가 있다면 건너뛰기
                    while i < j and nums[i] == nums[i + 1]:
                        i += 1
                    while i < j and nums[j] == nums[j - 1]:
                        j -= 1**
                        
                    # 3. 무한 루프 방지: 정답을 찾았으니 다음 숫자로 양쪽 다 이동!
                    i += 1
                    j -= 1
                    
                **elif total < 0:
                    i += 1
                else:
                    j -= 1**

        return results
```

### 4. Container With Most Water (Medium)

- two pointer랑 greedy 방식으로 풀어야 했다.
- 여기서 중요했던 건 if height[i] < height[j]: 조건에서 i를 뒤로 밀거냐, 아니면 j를 앞으로 당길 거냐- 였다. 만약 i번째의 원소가 더 작다면 움직여서 높이를 늘려야 한다!
- 그리고 꼭 max_area = max(curr_area, max_area) 로 갱신 시키기

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        i, j = 0, len(height) -1
        max_area = 0

        while i < j:
            curr_area = (j - i) * min(height[i], height[j])
            max_area = max(curr_area, max_area)
            if height[i] < height[j]: 
                i += 1
            else: 
                j -= 1

        return max_area

```

### 5. Longest Substring Without Repeating Characters (Medium)

- Sliding window를 이용한 문제이다. 중복 문자가 s에 있을 때 어떻게 처리할 지가 관건이다.
- set() 메서드를 사용해서 종류를 먼저 얻는다.
- right를 가지고 지금 슬라이딩 윈도우 안에 중복이 들어오나 체크하고, 만약 들어온다면 left += 1 해서 뒤로 움직여준다. 이 과정을 반복하며 max 갱신.

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_set = set() 
        left = 0
        max_len = 0
        
        for right in range(len(s)):
            # 중복이 없어질 때까지 left 포인터를 앞으로 이동시키며 바구니에서 뺍니다.
            while s[right] in char_set:
                char_set.remove(s[left])
                left += 1
                
            char_set.add(s[right])
            max_len = max(max_len, right - left + 1)
            
        return max_len

```
