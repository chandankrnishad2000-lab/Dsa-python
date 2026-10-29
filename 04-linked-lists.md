# 04 — Linked Lists

In Python interviews you are usually given this node definition:

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

Three tools solve almost everything:

1. **Dummy head** — removes edge cases when the head may change.
2. **Fast/slow pointers** — middle, cycle, k-th from end.
3. **Iterative reversal** — reverse all or a segment.

## 1. Dummy head

```python
# T: dummy head
def remove_elements(head, val):
    dummy = ListNode(0, head)
    prev = dummy
    while prev.next:
        if prev.next.val == val:
            prev.next = prev.next.next
        else:
            prev = prev.next
    return dummy.next
```

Use dummy for: remove nodes, merge lists, partition list, insert/delete at position,
and building a new list at the tail.

## 2. Fast / slow pointers

### Middle of the list
```python
def middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow
```
For an even-length list this returns the **second** middle. To get the first middle,
start `fast = head.next`.

### Cycle detection (Floyd)
```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False

def detect_cycle_start(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            slow = head
            while slow is not fast:
                slow = slow.next
                fast = fast.next
            return slow
    return None
```
Why it works: let the cycle length be C and distance from head to cycle start be D.
When they meet, fast has traveled D + kC + m. Resetting one pointer to head and
stepping both at speed 1 meets exactly at the cycle entrance.

### k-th from the end
```python
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    slow = fast = dummy
    for _ in range(n + 1):
        fast = fast.next
    while fast:
        slow = slow.next
        fast = fast.next
    slow.next = slow.next.next
    return dummy.next
```

## 3. Iterative reversal

```python
# T: reverse whole list
def reverse_list(head):
    prev = None
    while head:
        head.next, prev, head = prev, head, head.next
    return prev
```

### Reverse a sublist [left, right]
```python
def reverse_between(head, left, right):
    dummy = ListNode(0, head)
    prev = dummy
    for _ in range(left - 1):
        prev = prev.next
    cur = prev.next
    for _ in range(right - left):
        nxt = cur.next
        cur.next = nxt.next
        nxt.next = prev.next
        prev.next = nxt
    return dummy.next
```

### Reverse in groups of k
```python
def reverse_k_group(head, k):
    dummy = ListNode(0, head)
    group_prev = dummy
    while True:
        kth = group_prev
        for _ in range(k):
            kth = kth.next
            if not kth:
                return dummy.next
        group_next = kth.next
        prev, cur = kth.next, group_prev.next
        while cur is not group_next:
            cur.next, prev, cur = prev, cur, cur.next
        tmp = group_prev.next
        group_prev.next = kth
        group_prev = tmp
```

## 4. Merge

```python
def merge_two_lists(a, b):
    dummy = tail = ListNode(0)
    while a and b:
        if a.val <= b.val:
            tail.next, a = a, a.next
        else:
            tail.next, b = b, b.next
        tail = tail.next
    tail.next = a or b
    return dummy.next
```

### Sort a list (merge sort)
```python
def sort_list(head):
    if not head or not head.next:
        return head
    slow, fast = head, head.next
    while fast and fast.next:
        slow, fast = slow.next, fast.next.next
    mid, slow.next = slow.next, None
    return merge_two_lists(sort_list(head), sort_list(mid))
```
Time O(n log n), space O(log n) stack.

## 5. Structural transformations

- **Copy list with random pointer**: interleave copies (`A → A' → B → B'`), wire random, split.
**LRU cache**: `dict` + doubly linked list, or just `OrderedDict` in Python:

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.od = OrderedDict()

    def get(self, key):
        if key not in self.od:
            return -1
        self.od.move_to_end(key)
        return self.od[key]

    def put(self, key, value):
        if key in self.od:
            self.od.move_to_end(key)
        self.od[key] = value
        if len(self.od) > self.cap:
            self.od.popitem(last=False)
```
- **Add two numbers**: digit-wise with carry, dummy head.
- **Palindrome list**: find middle + reverse second half + compare.

## Python gotchas

- Always guard `while fast and fast.next` before `fast.next.next`.
- Reassign in one line carefully: `head.next, prev, head = prev, head, head.next`
  evaluates the right-hand side first, so it is safe.
- Return `dummy.next`, never `head`, when the head may change.
- Comparing nodes: use `is` for identity, `.val` for equality.

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Reverse Linked List | reversal | Easy |
| Merge Two Sorted Lists | dummy + merge | Easy |
| Remove Duplicates from Sorted List | dummy/prev | Easy |
| Linked List Cycle I / II | Floyd | Easy/Medium |
| Middle of the Linked List | fast/slow | Easy |
| Remove Nth Node From End | gap pointers | Medium |
| Reverse Linked List II | sublist reversal | Medium |
| Palindrome Linked List | middle + reverse | Medium |
| Partition List | dummy heads | Medium |
| Add Two Numbers | carry + dummy | Medium |
| Copy List with Random Pointer | interleave | Medium |
| Sort List | merge sort | Medium |
| Reorder List | middle + reverse + merge | Medium |
| LRU Cache | map + DLL | Medium |
| Reverse Nodes in k-Group | group reversal | Hard |
| Merge k Sorted Lists | heap (see 07) | Hard |
