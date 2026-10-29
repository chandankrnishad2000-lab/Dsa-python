# 13 — Pattern Cheatsheet

One page to decide the approach in the first 60 seconds.

## Decision guide by problem wording

| Wording | Pattern | File |
|---------|---------|------|
| sorted array + pair/triplet | two pointers | 01 |
| contiguous subarray, longest/shortest | sliding window | 01 |
| subarray sum / xor / balance | prefix + hash map | 02 |
| range updates | difference array | 02 |
| next greater/smaller, span | monotonic stack | 03 |
| window max/min | monotonic deque | 03 |
| linked list head may change | dummy head | 04 |
| middle / cycle / kth from end | fast/slow | 04 |
| sorted, boundaries, "first true" | binary search | 05 |
| "minimize the maximum" / "can we do it with X" | binary search on answer | 05 |
| next greater / span | monotonic stack | 03 |
| level, width, shortest in unweighted graph | BFS | 06, 08 |
| subtree aggregation, height, diameter | postorder DFS | 06 |
| BST sorted order / kth smallest | inorder | 06 |
| top-k / kth / streaming | heap | 07 |
| k sorted streams | k-way merge | 07 |
| running median | two heaps | 07 |
| dependencies / ordering | topological sort | 08 |
| connected components / dynamic connectivity | DSU | 08 |
| weighted shortest path | Dijkstra | 08 |
| all combinations/permutations | backtracking | 09 |
| count ways / optimal with choices | DP | 10 |
| two strings, subsequence/edit | 2D DP | 10 |
| intervals, max non-overlap | sort by end | 11 |
| merge overlapping | sort by start | 11 |
| prefix strings | trie | 12 |
| pairs via XOR / subsets | bit tricks | 12 |

## Complexity / constraint cheat table

| n | Allowed |
|---|---------|
| 20 | 2^n, n! with pruning, bitmask DP |
| 100-500 | O(n^3) |
| 5,000 | O(n^2) |
| 10^5 | O(n log n) |
| 10^6 | O(n) |

## Template index (copy-paste targets)

| Template | Where |
|----------|-------|
| opposite two pointers | 01 |
| variable sliding window | 01 |
| prefix state + hash map | 02 |
| difference array | 02 |
| next greater (monotonic) | 03 |
| sliding window max (deque) | 03 |
| dummy head | 04 |
| fast/slow + Floyd | 04 |
| reverse list | 04 |
| lower_bound | 05 |
| binary search on answer | 05 |
| BFS shortest path | 08 |
| topological sort | 08 |
| DSU | 08 |
| Dijkstra | 08 |
| backtracking | 09 |
| memoized DP | 10 |
| 0/1 knapsack | 10 |
| interval DP | 10 |
| trie | 12 |

## 15 mistakes that cost interviews

1. Not checking constraints before choosing an algorithm.
2. `list.pop(0)` instead of `deque.popleft()`.
3. Forgetting `path.copy()` when collecting backtracking results.
4. Forgetting to un-choose state in backtracking.
5. Off-by-one in binary search (`lo <= hi` vs `lo < hi` mixing).
6. Not skipping duplicate siblings after sorting (`i > start and nums[i] == nums[i-1]`).
7. Using `==` instead of `is` for nodes in linked lists/trees.
8. Not handling empty input / single element / all-equal elements.
9. Integer vs float division in Python 3 (`/` yields float; use `//` or `int()`).
10. Mutating a list while iterating over it.
11. Building strings with `+=` in a loop (O(n²)).
12. Recursion depth > 1000 without raising the limit.
13. Not counting the recursion stack in space complexity.
14. Forgetting the `if len(order) == n` cycle check in topological sort.
15. Stale heap entries in Dijkstra (check `if d > dist[u]: continue`).

## Interview communication checklist

1. Restate the problem and confirm edge cases.
2. State a brute force + complexity.
3. Improve using a pattern from this sheet.
4. Write code, then dry-run on a small example.
5. State final time/space with justification.
6. Mention trade-offs and possible optimizations.

## Final-week review loop

- Re-solve every problem you failed, from scratch, no notes.
- Time-box: 10 min easy, 25 min medium, 40 min hard.
- Explain each solution out loud in 60 seconds.
- Re-read this sheet each morning.
