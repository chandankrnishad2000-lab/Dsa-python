# 10 — Dynamic Programming

DP = recursion + overlapping subproblems + optimal substructure.
Your job in every problem: **define the state**, **write the transition**, **set base cases**.

## 0. The 5-step recipe

1. **State**: what do the args of `dp(...)` mean? (`i`, `j`, `remaining`, `last`)
2. **Transition**: how does a state depend on smaller states?
3. **Base cases**: empty input, index out of range, zero remaining.
4. **Order**: top-down memo, or bottom-up by increasing state.
5. **Answer**: which cell/return value is the final answer?

Interview default: write the **memoized recursion** first (clear and correct),
then optimize to bottom-up / rolling arrays if space matters.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def dp(i, state):
    if base:
        return base_value
    return combine(dp(i - 1, ...), dp(i - 2, ...))
```

Always clear the cache between test cases when using module-level functions.

## 1. 1D DP — linear sequences

### Climbing stairs / Fibonacci
```python
def climb(n):
    a, b = 1, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

### House robber (cannot take adjacent)
```python
def rob(nums):
    prev, cur = 0, 0
    for x in nums:
        prev, cur = cur, max(cur, prev + x)
    return cur
```

### Coin change — minimum coins (unbounded knapsack)
```python
def coin_change(coins, amount):
    INF = amount + 1
    dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return -1 if dp[amount] == INF else dp[amount]
```
Loop order matters:
- **Coins outer** → combinations (order does not matter).
- **Amount outer** → permutations (order matters). This is the exact difference
  between "Coin Change II" and "Combination Sum IV".

### Word break
```python
def word_break(s, words):
    wordset = set(words)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in wordset:
                dp[i] = True
                break
    return dp[n]
```

### Longest increasing subsequence — O(n²) and O(n log n)
```python
def lis_n2(nums):
    if not nums:
        return 0
    dp = [1] * len(nums)
    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

from bisect import bisect_left
def lis_nlogn(nums):
    tails = []                      # tails[k] = min possible tail of an LIS of length k+1
    for x in nums:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)
```
The O(n log n) version gives the **length**, not the sequence. For the actual
sequence, store indices and reconstruct.

### Decode ways / Kadane
```python
def max_subarray(nums):
    best = cur = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```

## 2. 2D DP — grids and two strings

### Unique paths / minimum path sum
```python
def unique_paths(m, n):
    dp = [1] * n
    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]
    return dp[-1]

def min_path_sum(grid):
    R, C = len(grid), len(grid[0])
    dp = [[0] * C for _ in range(R)]
    dp[0][0] = grid[0][0]
    for r in range(R):
        for c in range(C):
            if r == 0 and c == 0:
                continue
            top = dp[r - 1][c] if r else float("inf")
            left = dp[r][c - 1] if c else float("inf")
            dp[r][c] = grid[r][c] + min(top, left)
    return dp[-1][-1]
```

### Longest common subsequence
```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]
```
Related: edit distance (`dp[i-1][j-1]`, `dp[i-1][j]`, `dp[i][j-1]` +1),
distinct subsequences, wildcard matching, regex matching, interleaving string.

### Edit distance
```python
def edit_distance(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j - 1],   # replace
                                   dp[i - 1][j],       # delete
                                   dp[i][j - 1])       # insert
    return dp[m][n]
```

## 3. Knapsack family

### 0/1 knapsack (each item once)
```python
def knapsack(weights, values, cap):
    dp = [0] * (cap + 1)
    for w, v in zip(weights, values):
        for c in range(cap, w - 1, -1):     # backwards: one use per item
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[cap]
```

### Unbounded knapsack (reuse) — forward loop
```python
for w, v in zip(weights, values):
    for c in range(w, cap + 1):             # forwards: unlimited use
        dp[c] = max(dp[c], dp[c - w] + v)
```

### Subset sum / partition equal subset
```python
def can_partition(nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    for x in nums:
        for s in range(target, x - 1, -1):
            dp[s] = dp[s] or dp[s - x]
    return dp[target]
```

### Count subsets with sum k
`dp[s] += dp[s - x]` (backwards for 0/1, forwards for unlimited).
Beware counting duplicates: for "target sum" with +/- signs, the answer is
`count_subsets(sum+target // 2)` when non-negative.

## 4. Interval DP

`dp[i][j]` = answer for subarray/substring `i..j`, fill by increasing length.

### Longest palindromic substring
```python
def longest_palindrome(s):
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start = length = 0
    for i in range(n - 1, -1, -1):
        for j in range(i, n):
            if s[i] == s[j] and (j - i < 2 or dp[i + 1][j - 1]):
                dp[i][j] = True
                if j - i + 1 > length:
                    start, length = i, j - i + 1
    return s[start:start + length]
```

### Burst balloons / matrix chain (try every split point)
```python
def max_coins(nums):
    a = [1] + nums + [1]
    n = len(a)
    dp = [[0] * n for _ in range(n)]
    for length in range(2, n):
        for i in range(n - length):
            j = i + length
            for k in range(i + 1, j):
                dp[i][j] = max(dp[i][j],
                               dp[i][k] + a[i] * a[k] * a[j] + dp[k][j])
    return dp[0][n - 1]
```

## 5. State-machine DP (stock problems)

States: `hold` (own a stock), `sold`/`cash` (not holding). Transitions encode the
rules (cooldown, transaction limit).

```python
def max_profit_k(k, prices):
    if not prices:
        return 0
    buy = [-float("inf")] * (k + 1)
    sell = [0] * (k + 1)
    for p in prices:
        for t in range(1, k + 1):
            buy[t] = max(buy[t], sell[t - 1] - p)
            sell[t] = max(sell[t], buy[t] + p)
    return sell[k]
```

## 6. DP on trees

State lives at the node; return a tuple to the parent.

```python
def rob_tree(root):
    def dfs(node):
        if not node:
            return (0, 0)              # (rob this node, skip this node)
        l_rob, l_skip = dfs(node.left)
        r_rob, r_skip = dfs(node.right)
        return (node.val + l_skip + r_skip, max(l_rob, l_skip) + max(r_rob, r_skip))
    return max(dfs(root))
```

## 7. Bitmask DP (`n ≤ 20`)

State = subset of items as an integer bitmask.

```python
def tsp(dist):                                 # minimum Hamiltonian path
    n = len(dist)
    INF = float("inf")
    dp = [[INF] * n for _ in range(1 << n)]
    for i in range(n):
        dp[1 << i][i] = 0
    for mask in range(1 << n):
        for last in range(n):
            if dp[mask][last] == INF:
                continue
            for nxt in range(n):
                if mask >> nxt & 1:
                    continue
                nm = mask | (1 << nxt)
                dp[nm][nxt] = min(dp[nm][nxt],
                                  dp[mask][last] + dist[last][nxt])
    return min(dp[(1 << n) - 1])
```

## 8. Optimization checklist

- **Space**: 2D that depends only on the previous row → rolling array.
  Knapsack → 1D backwards.
- **Time**: prefix sums for range costs; monotonic deque for sliding-window min;
  `bisect` for LIS; convex hull trick is rarely needed.
- **Reconstruction**: keep a `parent`/`choice` array alongside `dp`.
- **Impossible states**: use a sentinel like `INF` and skip them.
- **Count vs optimize**: counts use `+`, optimization uses `min/max`; modulo
  counts with `10**9 + 7`.

## Pattern recognition table

| Signal | DP shape |
|--------|----------|
| "number of ways" to reach a target | 1D/2D counting |
| "min/max cost" with choices per step | knapsack / decision DP |
| subsequence of two strings | 2D LCS-style |
| contiguous subarray with optimal sum | Kadane / prefix + state |
| "must pick increasing/valid sequence" | LIS |
| operations on a string with intervals | interval DP |
| "at most k transactions / k changes" | DP with extra dimension k |
| n ≤ 20, all subsets | bitmask DP |
| tree with local decisions | tree DP (postorder tuples) |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Climbing Stairs | 1D | Easy |
| House Robber I / II | 1D | Medium |
| Coin Change I / II | knapsack | Medium |
| Word Break | 1D + set | Medium |
| Longest Increasing Subsequence | LIS | Medium |
| Maximum Subarray | Kadane | Medium |
| Unique Paths I / II | grid | Medium |
| Minimum Path Sum | grid | Medium |
| Longest Common Subsequence | 2D | Medium |
| Edit Distance | 2D | Medium |
| Longest Palindromic Substring | interval | Medium |
| Palindromic Substrings | interval | Medium |
| Partition Equal Subset Sum | subset sum | Medium |
| Target Sum | subset count | Medium |
| Best Time to Buy and Sell Stock III / IV | state machine | Hard |
| Burst Balloons | interval | Hard |
| Regular Expression Matching | 2D | Hard |
| Wildcard Matching | 2D | Hard |
| House Robber III | tree DP | Medium |
| Partition to K Equal Subsets | bitmask DP | Medium |
| Shortest Path Visiting All Nodes | bitmask + BFS | Hard |
| Cherry Pickup | 2D with two walkers | Hard |
