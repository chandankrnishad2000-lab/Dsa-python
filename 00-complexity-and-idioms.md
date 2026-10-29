# 00 — Complexity & Python Idioms

Advanced students lose points on *constant factors* and *language details*, not Big-O theory.
This file covers what actually matters in Python interviews.

## 1. Complexity you must know cold

| Structure | Access | Search | Insert | Delete |
|-----------|--------|--------|--------|--------|
| list (array) | O(1) | O(n) | O(n) back / O(1) amortized append | O(n) |
| deque | O(n) | O(n) | O(1) both ends | O(1) both ends |
| dict / set | — | O(1) avg / O(n) worst | O(1) avg | O(1) avg |
| heap (heapq) | O(1) peek | O(n) | O(log n) | O(log n) pop |
| balanced BST | O(log n) | O(log n) | O(log n) | O(log n) |

Sorting: `O(n log n)` — Timsort is O(n) on nearly-sorted data.

## 2. Amortized analysis (the interview traps)

- **list.append** — O(1) amortized because capacity doubles. Still O(n) total to build.
- **list.pop(0)** — O(n)! Use `deque.popleft()`.
- **`x in list`** — O(n). Use a `set` when doing repeated membership checks.
- **String concatenation in a loop** — `s += c` is O(n²) total. Use a list + `''.join(parts)`.
- **Slicing** — `arr[i:j]` copies, O(j-i). In recursion this can silently make your solution O(n²).
- **`heapq.heapify`** — O(n), cheaper than n pushes (O(n log n)).

## 3. Space complexity accounting

Interviewers count auxiliary space, including:
- recursion stack (depth d → O(d))
- the output you build (usually excluded from "space" but say it out loud)
- hash maps of size n

## 4. Python idioms that change your complexity

### Frequency counting
```python
from collections import Counter
freq = Counter(nums)          # O(n)
freq.most_common(3)           # O(n log k)
```

### Default dict for grouping
```python
from collections import defaultdict
groups = defaultdict(list)
for i, w in enumerate(words):
    groups[len(w)].append(i)
```

### Bisect for sorted inserts / boundary search
```python
from bisect import bisect_left, bisect_right, insort
i = bisect_left(arr, x)       # first index with arr[i] >= x
j = bisect_right(arr, x)      # first index with arr[j] >  x
# count of x in sorted arr == j - i
```

### Sorting with keys (multi-key, descending)
```python
events.sort(key=lambda e: (e[0], -e[1]))
```

### Infinite sentinel and unpacking
```python
INF = float("inf")
a, b = b, a                   # swap, no temp
```

### Recursion
```python
import sys
sys.setrecursionlimit(1_000_000)
```
Python has no tail-call optimization — an iterative version is always safer for depth > 10^5.

## 5. Common micro-optimizations (only after correctness)

- Local-bind hot attributes: `append = res.append`.
- Use `for i, x in enumerate(nums)` instead of `range(len(nums))`.
- Avoid function-call overhead inside O(n²) loops when n is near 5,000.
- `int` is big-int in Python: bit tricks are cheap, overflow does not exist.
- For XOR / bit ops, negative numbers use infinite two's complement — mask with `& ((1<<32)-1)` if needed.

## 6. Reading constraints to pick the algorithm

| Constraint | Implied approach |
|------------|------------------|
| n ≤ 20 | brute force / bitmask DP |
| n ≤ 100, values small | O(n³) DP, Floyd-Warshall |
| n ≤ 5,000 | O(n²) DP |
| n ≤ 10^5 | O(n log n) sort / heap / binary search |
| n ≤ 10^6 | O(n) two pointers / counting |
| "sorted array" | two pointers / binary search |
| "contiguous subarray" | sliding window / prefix sum |
| "top k" | heap / quickselect |
| "shortest path, unweighted" | BFS |
| "shortest path, weighted" | Dijkstra |
| "connected components" | DFS / Union-Find |
| "all combinations" | backtracking |

## Checkpoint

- [ ] I can state time and space complexity for every template in this course.
- [ ] I know why `pop(0)` and `s += c` are traps.
- [ ] I check constraints before writing code.
