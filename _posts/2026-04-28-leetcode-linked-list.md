---
category: "LeetCode"
---
## Linked List
### 1. Reverse Linked List (Easy)

- 링크드 리스트를 반대로 뒤집고 싶으면 어떻게 해야 할까?
- 크게 세가지 변수(포인터)를 준비하면 된다. 과거, 현재, 미래를 의미하는 포인터들이다.
- curr = head 로 ‘지금 조작할 노드’를 선택해주고,
- nxt = curr.next 로

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        prev = None
        curr = head

        while curr is not None: # 현재 작업할 노드가 존재하면
            nxt = curr.next
            
            # 여기가 중요! 
            curr.next = prev # 지금 노드의 화살표를 prev 로 뒤 노드를 향하게 한다!

            prev = curr
            curr = nxt
        
        return prev
        
```

### 2. Merge Two Sorted Lists (Easy)

- dummy = ListNode() 를 만들어야 시작점을 잃어버리지 않을 수 있다. 그대신 직접적으로 사용하는 건 아니다!
- 대신, curr = dummy 변수를 만들어서 이걸 가지고 노드를 이어줄거야~ (얘가 돌아다니는 거임)

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode()
        curr = dummy

        while list1 and list2:
            if list1.val <= list2.val:
                curr.next = list1
                list1 = list1.next
            else:
                curr.next = list2
                list2 = list2.next
            curr = curr.next

        if list1:
            curr.next = list1
        elif list2:
            curr.next = list2

        return dummy.next

```

### 3. Linked List Cycle (Easy)

- 이 링크드 리스트에 순환(사이클)이 있는가? 를 검증하는 메서드를 만들어 보자
- ‘토끼와 거북이’ 방법을 쓰면 된다!
- fast 랑 slow 변수를 만들어서 slow 는 한 번에 한 노드씩 움직이고, fast 는 두 개의 노드씩 움직여서, 만약에 이 링크드 리스트가 순환이라면 꼭 둘이 만나게 된다.
- 무한 루프가 도는 거 아닌가? 해서 while 문 조건으로 ‘fast is not None and fast.next is not None’ 을 썼다. 그래서 만약 순환이 아니라 끊기는 지점이 있다면 True 를 얻기 전에 while문을 탈출하게 되고 결국 return False 하게 된다. 그래서 걱정 노노!

```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = head
        fast = head

        while fast is not None and fast.next is not None:
            slow = slow.next
            fast = fast.next.next

            if fast == slow: return True
        
        return False
```

리스트가 비었을 때도 생각해야 한다

```python
Constraints:

The number of the nodes in the list is in the range [0, 104].
-105 <= Node.val <= 105
 

Follow up: Can you solve it using O(1) (i.e. constant) memory?
```

### 4. Remove Nth Node From End of List (Medium)

- 앞에서 N번째가 아니라 뒤에서 N번째를 제거하려면 어떻게 해야 할까?
- 아까 위에서 풀었던 것처럼 토끼와 거북이 방식을 사용해보자.
- fast 를 먼저 n 번 가도록 하고, 그 다음에 둘이 함께 돌리면 (둘 다 한 칸씩 전진) 그러면 fast가 마지막 노드를 만났을 때 (== fast.next is None 일 때) 가 slow는 제거해야 하는 그 노드의 바로 앞 노드에 위치하게 된다.
- slow.next = slow.next.next 로 제거하고 싶은 그 노드를 건너 뛰어서 연결하면 된다.

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        dummy = ListNode(0, head)
        fast = dummy
        slow = dummy

        for _ in range(n):
            fast = fast.next

        while fast.next is not None:
            fast = fast.next
            slow = slow.next
        
        slow.next = slow.next.next

        return dummy.next
```

### 5. Reorder List (Medium)

- 아주…. 애먹었던 문제이다.
- 주석에 자세하게 썼다!

```python
class Solution:
    def reorderList(self, head: Optional[ListNode]) -> None:
        if not head or not head.next:
            return

        # 중간 찾기 (토끼와 거북이)
        slow = head
        fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        curr = slow.next # 시작점을 잡고,
        slow.next = None # 가위로 잘라주기!

        prev = None
        while curr is not None: 
            nxt = curr.next
            curr.next = prev 
            prev = curr
            curr = nxt
        
        # 번갈아 가며 연결하기
        first = head       # 앞 덩어리의 시작
        second = prev      # 뒤집힌 뒷 덩어리의 시작

        while second:
            # 1. 길이 끊기기 전에 각각의 '다음 노드'를 임시 대피소에 저장한다.
            tmp1 = first.next
            tmp2 = second.next
            
            # 2. 앞 노드가 뒷 노드를 가리키게 한다. 
            first.next = second
            
            # 3. 뒷 노드가 아까 저장해둔 앞 노드의 다음을 가리키게 한다.
            second.next = tmp1
            
            # 4. 두 포인터 모두 다음 작업 위치로 전진!
            first = tmp1
            second = tmp2
```
