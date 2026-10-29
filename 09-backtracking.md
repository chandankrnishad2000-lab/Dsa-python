# 09 — Backtracking

Backtracking = DFS over a decision tree: **choose → explore → un-choose**.
The skill is in (a) defining the state, (b) choosing the loop, (c) pruning early.

## 1. The universal template

```python
# T: backtracking
def solve(candidates):
    res = []
    path = []

    def backtrack(start, state):
        if is_solution(state):
            res.append(path.copy())        # copy!
            return
        for choice in choices(start, state):
            if not is_valid(choice, state):
                continue
            path.append(choice)            # choose
            backtrack(next_start(start, choice), update(state, choice))
            path.pop()                     # un-choose

    backtrack(0, initial_state)
    return res
```

## 2. Subsets (the base case of all combination problems)

```python
def subsets(nums):
    res = []
    def bt(start, path):
        res.append(path.copy())
        for i in range(start, len(nums)):
            path.append(nums[i])
            bt(i + 1, path)
            path.pop()
    bt(0, [])
    return res
```

With duplicates — sort and skip equal siblings:
```python
def subsets_with_dup(nums):
    nums.sort()
    res = []
    def bt(start, path):
        res.append(path.copy())
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:
                continue
            path.append(nums[i])
            bt(i + 1, path)
            path.pop()
    bt(0, [])
    return res
```

## 3. Combinations & combination sum

### Combinations of k numbers from 1..n
```python
def combine(n, k):
    res = []
    def bt(start, path):
        if len(path) == k:
            res.append(path.copy())
            return
        for i in range(start, n + 1):
            if n - i + 1 < k - len(path):     # pruning
                break
            path.append(i)
            bt(i + 1, path)
            path.pop()
    bt(1, [])
    return res
```

### Combination sum (reuse allowed)
```python
def combination_sum(cands, target):
    res = []
    def bt(start, remaining, path):
        if remaining == 0:
            res.append(path.copy())
            return
        for i in range(start, len(cands)):
            if cands[i] > remaining:
                break                      # requires sorting
            path.append(cands[i])
            bt(i, remaining - cands[i], path)   # i, not i+1: reuse
            path.pop()
    bt(0, target, [])
    return res
```
- Reuse allowed → recurse with `i`.
- Each used once → recurse with `i + 1`.
- Duplicates in input → sort + `i > start and cands[i] == cands[i-1]: continue`.

## 4. Permutations

```python
def permute(nums):
    res = []
    used = [False] * len(nums)
    def bt(path):
        if len(path) == len(nums):
            res.append(path.copy())
            return
        for i, x in enumerate(nums):
            if used[i]:
                continue
            used[i] = True
            path.append(x)
            bt(path)
            path.pop()
            used[i] = False
    bt([])
    return res
```

Duplicates: sort, and skip `i > 0 and nums[i] == nums[i-1] and not used[i-1]`.

## 5. Constraint search: N-Queens

```python
def solve_n_queens(n):
    res = []
    cols, diag, anti = set(), set(), set()
    board = [["."] * n for _ in range(n)]

    def bt(r):
        if r == n:
            res.append(["".join(row) for row in board])
            return
        for c in range(n):
            if c in cols or (r - c) in diag or (r + c) in anti:
                continue
            cols.add(c); diag.add(r - c); anti.add(r + c)
            board[r][c] = "Q"
            bt(r + 1)
            board[r][c] = "."
            cols.remove(c); diag.remove(r - c); anti.remove(r + c)

    bt(0)
    return res
```

Same structure powers Sudoku solver: try digits 1-9 in each empty cell, validate
row/col/box, recurse, undo.

## 6. Word search (grid backtracking)

```python
def exist(board, word):
    R, C = len(board), len(board[0])
    def dfs(r, c, i):
        if i == len(word):
            return True
        if not (0 <= r < R and 0 <= c < C) or board[r][c] != word[i]:
            return False
        tmp, board[r][c] = board[r][c], "#"
        found = any(dfs(r + dr, c + dc, i + 1)
                    for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)))
        board[r][c] = tmp
        return found
    return any(dfs(r, c, 0) for r in range(R) for c in range(C))
```

## 7. Pruning checklist (this is what separates pass from TLE)

- **Sort first** so you can `break` on oversized candidates.
- **Skip duplicate siblings**: `i > start and nums[i] == nums[i-1]`.
- **Bound check**: if `remaining < 0`, return; if even the smallest candidate
  cannot fit, break.
- **Necessary-condition prune**: e.g., N-Queens column/diagonal sets; Sudoku
  precomputed masks.
- **Memoize** when the same state repeats (turns exponential into DP). Key on the
  decision state, not the path. Example: "partition to k equal sum subsets"
  memoizes `(index, filled_buckets, current_sum)`.
- **Symmetry breaking**: when choices are interchangeable (e.g., filling k equal
  buckets), force the first item into the first bucket.

## 8. Complexity intuition

| Problem | Leaves | Time |
|---------|--------|------|
| subsets | 2^n | O(n · 2^n) with copy cost |
| permutations | n! | O(n · n!) |
| combinations C(n,k) | C(n,k) | O(k · C(n,k)) |
| N-Queens | ≤ n! | pruned heavily |

Always state the tree size and the cost per leaf (usually path length).

## Practice set

| Problem | Variant | Difficulty |
|---------|---------|------------|
| Subsets I / II | include/exclude | Medium |
| Combinations | fixed size | Medium |
| Combination Sum I / II / III | reuse / dedupe | Medium |
| Permutations I / II | used array | Medium |
| Letter Combinations of a Phone Number | product | Medium |
| Palindrome Partitioning | split points | Medium |
| Restore IP Addresses | split + validation | Medium |
| Word Search | grid | Medium |
| N-Queens | constraint sets | Hard |
| Sudoku Solver | constraint masks | Hard |
| Word Search II | trie + backtracking (12) | Hard |
| Partition to K Equal Sum Subsets | memo + prune | Medium |
| Matchsticks to Square | symmetry prune | Medium |
| Generate Parentheses | balance prune | Medium |
