---
category: "LeetCode"
---
### Leetcode 문제 풀이
2026년이 된 기념으로 뭐를 할까 고민하다가 LeetCode를 풀어봤다.
한 문제 한 문제씩 풀며 처음에 나는 어떻게 생각했고, 더 나은 풀이는 어떤 게 있었는지, 정답이랑 내가 처음에 생각했던 아이디어랑 비교하는 과정이 재밌었다.
문제를 풀면서 시간 복잡도랑 공간 복잡도를 더 고려하게 되고, 더 better 한 알고리즘은, 방식은 뭐가 있는지 찾으면서 세상에는 똑똑한 사람들이 참 많다고 느꼈다.
아무튼..
노션에만 적어둔건데, 기록용으로 깃허브에도 올리고 싶었다.

딱 6가지의 문제 유형만을 집중적으로 풀려고 노력했다.

1. Arrays & Hashing
2. Two Pointers & Sliding Window
3. Linked List
4. Stack & Binary Search
5. Trees
6. Graphs & Dynamic Programming

근데 LeetCode 에 sql 문제도 있더라? 나 sql 좋아하니까 그것도 해봐야지~~

## Arrays & Hashing


### 1. Two Sum (Easy)

- enumerate() 를 써서 인덱스(i) 와 요소(num) 을 한번에 꺼내는 걸 알아야 한다.
- 해시 - 딕셔너리 - 를 사용해서 seen={} 에 compare (즉, target과 num의 차이) 를 보관한다.
- 만약 보관함에 알맞는 compare가 있을 경우, 리스트 형태로 감싸서 각 숫자의 인덱스를 반환!

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}
        for i, num in enumerate(nums):
            compare = target - num

            if compare in seen:
                return [seen[compare], i]
                
            seen[num] = i
```

### 2. Valid Anagram (Easy)

- sorted() 는 timsort 를 써서 시간 복잡도는 O(NlogN)이고,
- 공간 복잡도는 새 문자열을 각각 만드니까 (s_sort, t_sort) O(N) 이다.

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        s_sort = sorted(s)
        t_sort = sorted(t)

        return s_sort == t_sort
```

### 3. Contains Duplicate (Easy)

- 여기서 중요한 건 set(). set() 이란 중복을 허용하지 않고 순서가 없는 데이터의 모임이다.
- 그래서 set() 으로 숫자들의 종류를 정리할 수 있다.

```python
class Solution:
    def containsDuplicate(self, nums: List[int]) -> bool:
        # 전체 len 을 찾고, 그게 총 숫자의 종류보다 크면 True 아닐까?
        return len(nums) > len(set(nums))       
```

### 4. Group Anagrams (Medium)

- 저번처럼 set() 을 쓰려다가 그러면 같은 애너그램을 가진 것들끼리 그룹을 나눌 수 없어서 버리고,
- 사실 join() 을 쓸 줄 아느냐 마느냐가 컸다. 그리고 딕셔너리를 편하게 활용하자!
- key = "".join(sorted(s))
- 아마도 O(NlogN) 이다.

```python
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        dicts = {}
        for s in strs:
            key = "".join(sorted(s))

            if key in dicts:
                dicts[key].append(s)
            else:
                dicts[key] = [s] # 이렇게 넣어줘야지 리스트 형태로 저장된다

        return list(dicts.values())
```

### 5. Top K Frequent Elements (Medium)

- sorted() 는 Timsort 로 O(NlogN) 이 걸린다. 하지만 문제에서는 시간 복잡도가 O(NlogN) 보다 ‘빨라야 한다’ 라는 조건이 있었기에 어려운 문제였다.
- 딕셔너리 - Hash Map - 을 사용해서 O(N) 으로 풀어야 했다.
- 종류와 빈도수를 정리한 dicts =  {2:2, 1:3, 3:1} 를 얻는 것까지는 성공. 하지만 여기서 어떻게 값을 기준으로 k번 까지 큰 키들을 가져올 수 있을까가 이 문제의 관건이었다.
- *sorted_items = sorted(dicts.items(), key=lambda x:x[1], reverse=True)* 를 써봤지만 sorted() 때문에 O(NlogN) 이므로 실패.
- heapq 가 솔루션이었다.
- import heapq 한 다음에, heap = [] 을 만들고 힙큐에 튜플 형태로 (빈도수, 종류) 넣어준다. 그리고 heappush(만든 힙, 튜플) 그리고 heappop(만든 힙) 을 이용하면 문제를 풀 수 있었다.
- 마지막에 리턴할 때 힙에서 두번째 값(우리가 원하는 키)를 꺼내기 위해서는 for 문을 돌리면서 인덱스1번째를 가져와 리스트로 만드는 작업을 해야 한다.

```python
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        dicts = {}
        for n in nums:
            if n in dicts:
                dicts[n] += 1
            else:
                dicts[n] = 1

        heap = []
        for num, freq in dicts.items():
            # 힙에는 튜플 형태로 (기준점, 데이터)를 넣습니다.
            # 빈도수를 기준으로 비교해야 하니까 freq를 앞에 둡니다!
            heapq.heappush(heap, (freq, num))

            if len(heap) > k:
                heapq.heappop(heap)

        return [item[1] for item in heap]
```

### 6. Product of Array Except Self (Medium)

- 문제에서 요구한 건 O(N) 이라서 이중 for문을 쓰면 안되었다. (보통 잘 쓰면 안되지만)
- 참고로 product 는 곱이라는 뜻이다.
- 관건은 ‘자기 자신을 제외하고 곱한다’ 를 어떻게 나눗셈을 쓰지 않고 계산할 수 있는가
    - 왼쪽과 오른쪽으로 범위를 나누고 한 개의 리스트, 그리고 새로 나오는 리스트를 만들지 않고 바로 곱해서 리턴값을 만들어 낼 수 있는가? 였다.
- 참고로 insert(0, n) 을 쓰려다가 (append() 와 반대로 작동한다. 앞에다가 끼워넣는다. 하지만 기존의 원소들을 재배치하는 과정에서 O() 가 많이 발생.) 시간 복잡도 때문에 insert() 대신 for문을 잘 조작해야 했다
    - for i in range(len(nums) -1, -1, -1) 을 써야했다. for n in nums[::-1] 을 쓰고 싶었는데 기존의 리스트의 맨뒤 원소랑 곱하기 시작하려면 인덱스가 필요해서 i 를 만드는 수밖에 없었다.

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        answer = []
        multiples = 1

        for n in nums:
            answer.append(multiples) 
            multiples *= n           
        
        multiples2 = 1

        for i in range(len(nums)-1, -1, -1):
            answer[i] *= multiples2
            multiples2 *= nums[i]

        return answer
```
