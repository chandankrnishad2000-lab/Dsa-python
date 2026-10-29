# DSA with Python — Advanced Patterns Course

An interview-focused, pattern-first study guide. Each file is a self-contained lesson:
intuition → template code → complexity → LeetCode practice set.

## How to use this course

1. Read the pattern file and type out the template **without looking**.
2. Solve the listed problems in order (easy → medium → hard).
3. If stuck > 25 min, read the hint, not the solution. Re-attempt next day.
4. Track progress in [`14-practice-tracker.md`](14-practice-tracker.md).
5. Review the [`13-pattern-cheatsheet.md`](13-pattern-cheatsheet.md) weekly.

## Curriculum

| # | File | Core patterns |
|---|------|---------------|
| 00 | [Complexity & Python Idioms](00-complexity-and-idioms.md) | Big-O, amortized cost, Python gotchas |
| 01 | [Arrays: Two Pointers & Sliding Window](01-arrays-two-pointers-sliding-window.md) | Opposite/same direction, fixed & variable window |
| 02 | [Hashing, Prefix Sums & Intervals](02-hashing-prefix-sums.md) | Frequency maps, prefix/suffix, difference arrays |
| 03 | [Stacks, Queues & Monotonic Stack](03-stacks-queues-monotonic.md) | Next greater, histogram, deque windows |
| 04 | [Linked Lists](04-linked-lists.md) | Dummy head, fast/slow, reversal, merge |
| 05 | [Binary Search](05-binary-search.md) | Boundary search, rotated, answer-space search |
| 06 | [Trees & BST](06-trees-bst.md) | DFS/BFS, serialize, LCA, BST invariants |
| 07 | [Heaps & Top-K](07-heaps-top-k.md) | Heapq, k-way merge, streaming median, scheduling |
| 08 | [Graphs](08-graphs.md) | BFS/DFS, topological sort, union-find, Dijkstra |
| 09 | [Backtracking](09-backtracking.md) | Subsets, permutations, pruning, constraint search |
| 10 | [Dynamic Programming](10-dynamic-programming.md) | 1D, 2D, knapsack, LIS, interval DP, memoization |
| 11 | [Greedy & Intervals](11-greedy-intervals.md) | Merge/insert intervals, scheduling, exchange argument |
| 12 | [Tries & Bit Manipulation](12-tries-bit-manipulation.md) | Prefix trees, XOR tricks, subsets via bits |
| 13 | [Pattern Cheatsheet](13-pattern-cheatsheet.md) | One-page decision guide |
| 14 | [Practice Tracker](14-practice-tracker.md) | Checklist of 120+ curated problems |

## Suggested 8-week plan

| Week | Focus |
|------|-------|
| 1 | 00, 01, 02 — arrays, windows, hashing |
| 2 | 03, 04, 05 — stacks, lists, binary search |
| 3 | 06, 07 — trees, heaps |
| 4 | 08 — graphs (BFS/DFS/topo/UF/Dijkstra) |
| 5 | 09, 11 — backtracking, greedy |
| 6 | 10 — DP part 1 (1D, 2D, knapsack) |
| 7 | 10 — DP part 2 (LIS, intervals, bitmask) + 12 |
| 8 | Mixed timed sets + re-solve every failed problem |

## Python runtime budget (typical interview)

| n | Feasible complexity |
|---|---------------------|
| ≤ 10 | O(n!), O(2^n) |
| ≤ 20 | O(2^n), bitmask DP |
| ≤ 500 | O(n^3) |
| ≤ 5,000 | O(n^2) |
| ≤ 10^6 | O(n log n) |
| ≤ 10^8 | O(n) |

## Conventions in this repo

- `# T:` marks the reusable template.
- All code targets Python 3.10+, no external libraries.
- Recursion depth: call `sys.setrecursionlimit(10**6)` for deep DFS.
- `deque` from `collections`, `heapq`, `bisect`, `defaultdict`, `Counter` are fair game.
