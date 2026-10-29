# 03 — Stacks, Queues & Monotonic Stack

## 1. Stack basics

LIFO. In Python, a plain `list` is the right stack (`append` / `pop`).

### Balanced parentheses
```python
def is_valid(s):
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []
    for ch in s:
        if ch in "([{":
            stack.append(ch)
        elif not stack or stack.pop() != pairs[ch]:
            return False
    return not stack
```

### Evaluate reverse polish notation
```python
def eval_rpn(tokens):
    stack = []
    ops = {
        "+": lambda a, b: a + b, "-": lambda a, b: a - b,
        "*": lambda a, b: a * b, "/": lambda a, b: int(a / b),
    }
    for t in tokens:
        if t in ops:
            b, a = stack.pop(), stack.pop()
            stack.append(ops[t](a, b))
        else:
            stack.append(int(t))
    return stack[0]
```

### Min-stack in O(1) per operation
Store `(value, current_min)` pairs, or a parallel min-stack.

```python
class MinStack:
    def __init__(self):
        self.s = []
        self.mins = []

    def push(self, x):
        self.s.append(x)
        self.mins.append(min(x, self.mins[-1] if self.mins else x))

    def pop(self):
        self.mins.pop()
        return self.s.pop()

    def top(self):
        return self.s[-1]

    def get_min(self):
        return self.mins[-1]
```

### Monotonic stack — next greater element
Core idea: keep a stack of **indices with unresolved next-greater**, in decreasing value order.

```python
# T: next greater element to the right
def next_greater(nums):
    n = len(nums)
    res = [-1] * n
    stack = []                     # indices, values decreasing
    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            res[stack.pop()] = x
        stack.append(i)
    return res
```

Variants by changing `<` to `<=` and direction:
- next greater/smaller, left or right
- circular array: iterate `2*n` with `i % n`
- "how far to the next warmer day": store index difference

### Daily temperatures
```python
def daily_temperatures(t):
    res = [0] * len(t)
    stack = []
    for i, temp in enumerate(t):
        while stack and t[stack[-1]] < temp:
            j = stack.pop()
            res[j] = i - j
        stack.append(i)
    return res
```

### Largest rectangle in histogram
```python
def largest_rectangle(heights):
    heights = heights + [0]        # sentinel flushes the stack
    stack = []                     # increasing heights (indices)
    best = 0
    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            left = stack[-1] + 1 if stack else 0
            best = max(best, height * (i - left))
        stack.append(i)
    return best
```
This is also the inner routine for "maximal rectangle" in a binary matrix
(compute running histogram height per row).

## 2. Queue basics

FIFO. Use `collections.deque` for O(1) on both ends.
Never use `list.pop(0)`.

```python
from collections import deque
q = deque()
q.append(1)
q.popleft()
```

### Implement queue with two stacks (amortized O(1))
```python
class MyQueue:
    def __init__(self):
        self.inbox, self.outbox = [], []

    def push(self, x):
        self.inbox.append(x)

    def pop(self):
        if not self.outbox:
            while self.inbox:
                self.outbox.append(self.inbox.pop())
        return self.outbox.pop()

    def peek(self):
        if not self.outbox:
            while self.inbox:
                self.outbox.append(self.inbox.pop())
        return self.outbox[-1]

    def empty(self):
        return not self.inbox and not self.outbox
```

## 3. Monotonic deque (sliding-window maximum/minimum)

A deque of indices whose values are monotonic. Front holds the current window's max.

```python
# T: sliding window maximum
def max_sliding_window(nums, k):
    dq = deque()                   # indices, values decreasing
    res = []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:         # evict out-of-window index
            dq.popleft()
        if i >= k - 1:
            res.append(nums[dq[0]])
    return res
```
Time O(n), space O(k).

Use the same structure for:
- shortest subarray with sum ≥ k
- constrained subsequence problems where you need the best value in a sliding window

## 4. Expression parsing (stack + shunting-yard idea)

For "basic calculator" style problems, keep a stack of `(result_so_far, sign)` and
push on `(`, pop and combine on `)`.

```python
def calculate(s):
    stack = []
    num = 0
    sign = 1
    res = 0
    for ch in s:
        if ch.isdigit():
            num = num * 10 + int(ch)
        elif ch in "+-":
            res += sign * num
            num = 0
            sign = 1 if ch == "+" else -1
        elif ch == "(":
            stack.append((res, sign))
            res, sign = 0, 1
        elif ch == ")":
            res += sign * num
            num = 0
            prev_res, prev_sign = stack.pop()
            res = prev_res + prev_sign * res
    res += sign * num
    return res
```

## Decision guide

| Signal | Pattern |
|--------|---------|
| nested / matched brackets | stack |
| "next greater/smaller element" | monotonic stack |
| "how many days until warmer" | monotonic stack of indices |
| largest rectangle / skyline | monotonic stack with sentinel |
| sliding-window max/min | monotonic deque |
| FIFO with O(1) ends | deque |
| parse arithmetic with parentheses | stack of partial results |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Valid Parentheses | stack | Easy |
| Min Stack | stack + min | Medium |
| Evaluate RPN | stack | Medium |
| Daily Temperatures | monotonic | Medium |
| Next Greater Element I / II (circular) | monotonic | Medium |
| Asteroid Collision | stack | Medium |
| Simplify Path | stack | Medium |
| Largest Rectangle in Histogram | monotonic + sentinel | Hard |
| Maximal Rectangle | histogram per row | Hard |
| Sliding Window Maximum | monotonic deque | Hard |
| Trapping Rain Water (stack variant) | monotonic | Hard |
| Basic Calculator I / II | stack parsing | Hard |
