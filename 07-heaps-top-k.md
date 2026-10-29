# 07 — Heaps & Top-K

Python's `heapq` is a **min-heap** over a list. For a max-heap, push negated values.

```python
import heapq

heapq.heappush(h, x)      # O(log n)
heapq.heappop(h)          # O(log n)
h[0]                      # O(1) min
heapq.heapify(list)       # O(n) — prefer over n pushes
```

## 1. Top-K patterns

### K largest via size-k min-heap
```python
# T: k largest, O(n log k)
def k_largest(nums, k):
    h = []
    for x in nums:
        heapq.heappush(h, x)
        if len(h) > k:
            heapq.heappop(h)
    return h            # smallest of the k largest is h[0]
```

### K smallest via size-k max-heap (negate)
```python
def k_smallest(nums, k):
    h = []
    for x in nums:
        heapq.heappush(h, -x)
        if len(h) > k:
            heapq.heappop(h)
    return [-x for x in h]
```

### K closest points to origin
```python
def k_closest(points, k):
    h = []
    for x, y in points:
        d = x * x + y * y
        heapq.heappush(h, (-d, x, y))       # max-heap of distances
        if len(h) > k:
            heapq.heappop(h)
    return [[x, y] for _, x, y in h]
```

### Quickselect (average O(n), worst O(n²))
Use when you only need the k-th value, not a heap:
```python
import random

def quickselect(nums, k):          # k is 0-indexed
    def partition(lo, hi):
        pivot = nums[random.randint(lo, hi)]
        i = lo
        for j in range(lo, hi):
            if nums[j] < pivot:
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
        nums[i], nums[hi] = nums[hi], nums[i]
        return i
    lo, hi = 0, len(nums) - 1
    while True:
        p = partition(lo, hi)
        if p == k:
            return nums[p]
        if p < k:
            lo = p + 1
        else:
            hi = p - 1
```

## 2. K-way merge

### Merge k sorted lists
```python
def merge_k_lists(lists):
    h = [(node.val, i, node) for i, node in enumerate(lists) if node]
    heapq.heapify(h)
    dummy = tail = ListNode(0)
    while h:
        val, i, node = heapq.heappop(h)
        tail.next = node
        tail = node
        if node.next:
            heapq.heappush(h, (node.next.val, i, node.next))
    return dummy.next
```
The `i` tiebreaker prevents comparing `ListNode` objects.

### Kth smallest element in a sorted matrix
```python
def kth_smallest_matrix(mat, k):
    h = [(mat[r][0], r, 0) for r in range(len(mat))]
    heapq.heapify(h)
    for _ in range(k):
        val, r, c = heapq.heappop(h)
        if c + 1 < len(mat[0]):
            heapq.heappush(h, (mat[r][c + 1], r, c + 1))
    return val
```

## 3. Streaming median (two heaps)

`lo` = max-heap of the smaller half, `hi` = min-heap of the larger half.
Invariant: `len(lo) == len(hi)` or `len(lo) == len(hi) + 1`.

```python
class MedianFinder:
    def __init__(self):
        self.lo = []           # max-heap (negated)
        self.hi = []           # min-heap

    def add(self, x):
        heapq.heappush(self.lo, -x)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def median(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

## 4. Scheduling / greedy with heaps

### Meeting rooms II (min rooms)
```python
def min_meeting_rooms(intervals):
    intervals.sort()
    h = []                     # end times
    for start, end in intervals:
        if h and h[0] <= start:
            heapq.heappop(h)
        heapq.heappush(h, end)
    return len(h)
```

### Task scheduler / reorganize string
Always use `Counter` + max-heap of remaining counts, with a cooldown buffer.
```python
from collections import Counter

def least_interval(tasks, n):
    freq = Counter(tasks)
    h = [(-c) for c in freq.values()]
    heapq.heapify(h)
    time = 0
    while h:
        cycle = []
        for _ in range(n + 1):
            if h:
                cycle.append(heapq.heappop(h))
        for c in cycle:
            if c + 1 < 0:
                heapq.heappush(h, c + 1)
        time += n + 1 if h else len(cycle)
    return time
```

### Huffman-style / "minimum cost to connect"
Always pop the two smallest, combine, push back. This is "minimum cost to
connect sticks" and "minimum cost to hire k workers" style greedy.

```python
def connect_sticks(sticks):
    heapq.heapify(sticks)
    cost = 0
    while len(sticks) > 1:
        a = heapq.heappop(sticks)
        b = heapq.heappop(sticks)
        cost += a + b
        heapq.heappush(sticks, a + b)
    return cost
```

## 5. Heap under custom ordering

Store tuples `(priority, tiebreaker, payload)`. Python compares tuples
lexicographically, so ensure all earlier elements are comparable.

```python
h = []
heapq.heappush(h, (dist, node_id, node))
```
Use `itertools.count()` as a unique tiebreaker when payloads are not comparable.

## Decision guide

| Signal | Pattern |
|--------|---------|
| "top k / kth largest / k closest" | size-k heap or quickselect |
| "merge k sorted" | heap of heads |
| "running median" | two heaps |
| "minimum rooms / machines" | heap of end times |
| "repeatedly combine two cheapest" | min-heap greedy |
| "schedule with cooldown" | heap + cooldown queue |
| "Dijkstra" | min-heap (see 08) |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Kth Largest Element in an Array | heap / quickselect | Medium |
| Kth Smallest Element in BST | heap or inorder | Medium |
| Top K Frequent Elements | Counter + heap | Medium |
| K Closest Points to Origin | size-k heap | Medium |
| Merge k Sorted Lists | k-way merge | Hard |
| Kth Smallest in a Sorted Matrix | k-way merge | Medium |
| Find Median from Data Stream | two heaps | Hard |
| Sliding Window Median | two heaps / sorted list | Hard |
| Meeting Rooms II | heap of ends | Medium |
| Task Scheduler | heap + cooldown | Medium |
| Reorganize String | heap + cooldown | Medium |
| Minimum Cost to Connect Sticks | min-heap | Medium |
| Furthest Building You Can Reach | heap / greedy | Medium |
| IPO (maximize capital) | two heaps | Hard |
