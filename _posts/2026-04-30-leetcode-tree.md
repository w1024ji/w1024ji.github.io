---
category: "LeetCode"
---
## Tree

### 1. Invert Binary Tree (Easy)

- 시간 복잡도는 O(N) 이다. N은 노드의 개수
- 공간 복잡도는 O(H) 이다. H는 트리의 높이
- 파이썬에서의 swap 은 쉽다. left, right = right, left
- 그리고 트리에서는 재귀를 자주 쓴다. 제일 첫 루트부터 하나씩 아래 노드로 가면서 재귀를 하면 된다.

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if root is None:
            return None
        # swap!
        root.left, root.right = root.right, root.left
        # recursion
        self.invertTree(root.left)
        self.invertTree(root.right)
        
        return root
```

### 2. Maximum Depth of Binary Tree (Easy)

- 이 문제도 재귀를 사용해서 풀어야 했다. 트리의 깊이가 얼마냐? 를 묻는 문제였다. 아까 1번 문제에서는 제일 위 루트에서부터 재귀를 돌렸다면, 이번에는 제일 밑바닥의 노드에서부터 총 계산한 깊이에 1 (자기 자신)을 더함으로써 다시 총 깊이를 갱신하는 모습이다.
- 꼭 메서드 호출할 때는 self. 를 써야 한다!!

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
            
        # 밑바닥에서 올라온 결과에 1을 더하자
        left_height = self.maxDepth(root.left)
        right_height = self.maxDepth(root.right)

        # 1은 나 자신 + 내 아래의 최대
        return 1 + max(left_height, right_height)
```

- iteration (반복)

```python
from collections import deque

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        
        # (중요!) 큐를 만들고, 1층의 시작점인 root를 먼저 세웁니다.
        queue = deque([root])
        depth = 0
        
        while queue:
            # 현재 큐에 있는 노드 개수 == '이번 층의 전체 인원'
            level_size = len(queue)
            
            for _ in range(level_size):
                curr_node = queue.popleft() 
                
                # (중요)뺀 노드에게 자식이 있다면 큐의 맨 뒤에 다음 층 멤버로 줄을 세웁니다.
                if curr_node.left:
                    queue.append(curr_node.left)
                if curr_node.right:
                    queue.append(curr_node.right)
            
            # 한 층(Level)을 전부 확인 완료 -> 깊이 +1 해준다
            depth += 1
            
        return depth
```

### 3. Same Tree (Easy)

- 재귀를 쓸 때는 while문을 쓰지 않는 것이 원칙이다.
- 재귀에서는 if 를 통한 종료 조건 처리가 중요하다.

```python
class Solution:
    def isSameTree(self, p: Optional[TreeNode], q: Optional[TreeNode]) -> bool:
        if not p and not q:
            return True
        
        if not p or not q:
            return False

        if p.val != q.val:
            return False

        return self.isSameTree(p.left, q.left) and self.isSameTree(p.right, q.right)
```

### 4. Lowest Common Ancestor of a Binary Search Tree (Medium)

- LCA(공간 공통 조상)를 찾는 과정은 한마디로 "갈림길(Split Point) 찾기"라고 정의할 수 있다.
    1. 동반 하강: 만약 p와 q가 둘 다 현재 노드보다 작다면, 둘은 모두 현재 노드의 왼쪽 어딘가에 살고 있다. 그럼 왼쪽으로 내려가야 한다.
    2. 동반 이동: 만약 p와 q가 둘 다 현재 노드보다 크다면, 둘은 모두 현재 노드의 오른쪽 어딘가에 살고 있다. 그럼 오른쪽으로 내려가야 한다.
    3. 갈림길 발견 (LCA): 만약 p는 작고 q는 크다면(혹은 그 반대), 혹은 현재 노드가 p나 q 중 하나라면? 바로 여기가 두 갈래 길로 나뉘는 지점! 즉, 현재 노드가 바로 조상이다.
- BST의 LCA 알고리즘은 이진 탐색과 완전히 동일한 시간 복잡도 O(H)를 가진다. 일반적인 이진 트리(Binary Tree)였다면 모든 노드를 다 뒤져야 하므로 O(N)이 걸렸겠지만, BST의 '정렬된 성질' 덕분에 탐색 성능을 획기적으로 올린 케이스이다.
- 데이터베이스 엔진에서 LCA 개념은 두 데이터 블록의 공통 상위 페이지를 찾거나, 트리의 균형(Rebalancing)을 맞출 때 아주 중요하게 사용된다.
- 재귀 없이 반복문(`while`)만으로도 완벽하게 풀 수 있다

```python
class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', 
													    p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        curr = root

        while curr:
            if p.val > curr.val and q.val > curr.val:
                curr = curr.right
            elif p.val < curr.val and q.val < curr.val:
                curr = curr.left
            else: # here is LCA
                return curr
```

### 5. Binary Tree Level Order Traversal (Medium)

- 2번 문제랑 굉장히 비슷하다!!!!
- BFS를 구현할 때 while 문 안에 for 루프를 넣어야 한다!
- 큐를 준비하고 - collections.deque - root 을 []리스트로 감싸서 넣어줘야 한다. 왜냐면 deque를 사용할 땐 deque() 안에 iterable 한 객체를 넣어야 하는데, []없이 root 만 넣으면 에러가 난다!!
- level_size는 이번 층에 있는 노드 개수
- current_level은 나중에 result 에 담을 그 층의 노드들의 값의 모임이다.

```python
from collections import deque

class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        
        result = []
        queue = deque([root])
        
        while queue:
            level_size = len(queue) 
            current_level = []
            
            for _ in range(level_size):
                node = queue.popleft()
                current_level.append(node.val)
                
                # 자식들을 대기열(큐)의 맨 뒤에 세움
                # 즉, 그 다음 레벨들의 노드들을 넣음
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
            
            # 한 층의 탐색이 끝나면 결과에 추가
            result.append(current_level)
            
        return result
```

### 6. Validate Binary Search Tree (Medium)

- 만약 5 노드 왼쪽에 4가 있는데, 4의 아래 오른쪽에 7이 있다면 이건 BST 가 아니다. BST 의 유효성 검사를 어떻게 해야 할까?
    - 바로바로~ 범위를 정해서 이 안에 들어오는 지 아닌지 검사하자!
    - 7의 자리에는 5보다는 작고 4보다는 큰 수가 들어와야 한다.
- 즉, 5의 왼쪽 서브 트리는 모두 5보다 작아야 하고
- 반대로 오른쪽 서브 트리는 모두 5보다 커야 한다.
- 이 범위를 재귀로 검사하기 위해선 isValidBST() 함수 안에 도우미 함수를 만드는 게 편하다. validate() 라는 도우미 함수를 만들었다.
- return 문을 유심히 보라!!

```python
import math

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        # 처음에는 최소값을 -무한대, 최대값을 +무한대로 설정한다
        def validate(node, low=-math.inf, high=math.inf):
            if not node: # 빈 노드라면
                return True
            if node.val <= low or node.val >= high: # 범위를 벗어났다면 실패
                return False
            
            # 왼쪽 노드와 오른쪽 노드에게 어떻게 넘길 것인가?
            return (validate(node.left, low, node.val) and validate(node.right, node.val, high))

        return validate(root)
```
