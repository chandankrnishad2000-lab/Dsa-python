# 11 — Greedy & Intervals

Greedy is easy to write and hard to trust. Always be ready to justify it with an
**exchange argument**: "any optimal solution can be transformed to include my choice
without getting worse."

## 1. Interval problems (the most common greedy family)

### Merge intervals
```python
def merge(intervals):
    intervals.sort()
    out = []
    for s, e in intervals:
        if out and s <= out[-1][1]:
            out[-1][1] = max(out[-1][1], e)
        else:
            out.append([s, e])
    return out
```
Sort by start; extend the last end when overlapping.

### Insert interval (already sorted, non-overlapping)
```python
def insert(intervals, new):
    out = []
    i, n = 0, len(intervals)
    while i < n and intervals[i][1] < new[0]:
        out.append(intervals[i])
        i += 1
    while i < n and intervals[i][0] <= new[1]:
        new[0] = min(new[0], intervals[i][0])
        new[1] = max(new[1], intervals[i][1])
        i += 1
    out.append(new)
    out.extend(intervals[i:])
    return out
```

### Non-overlapping intervals (minimum removals)
Sort by **end**, greedily keep the interval that finishes earliest.
```python
def erase_overlap(intervals):
    intervals.sort(key=lambda x: x[1])
    removals = 0
    last_end = float("-inf")
    for s, e in intervals:
        if s < last_end:
            removals += 1
        else:
            last_end = e
    return removals
```
Same template = maximum number of non-overlapping intervals = minimum arrows to
burst balloons (with `s <= last_end` because touching balloons burst).

### Meeting rooms II (minimum rooms) — heap variant
See file 07. Greedy proof: always put the next meeting in a room that freed earliest.

### Interval intersection
```python
def intersections(a, b):
    i = j = 0
    res = []
    while i < len(a) and j < len(b):
        lo = max(a[i][0], b[j][0])
        hi = min(a[i][1], b[j][1])
        if lo <= hi:
            res.append([lo, hi])
        if a[i][1] < b[j][1]:
            i += 1
        else:
            j += 1
    return res
```

## 2. Classic greedy problems

### Jump game I (can you reach the end?)
```python
def can_jump(nums):
    reach = 0
    for i, x in enumerate(nums):
        if i > reach:
            return False
        reach = max(reach, i + x)
    return True
```

### Jump game II (minimum jumps)
```python
def min_jumps(nums):
    jumps = end = farthest = 0
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        if i == end:
            jumps += 1
            end = farthest
    return jumps
```

### Gas station
```python
def can_complete_circuit(gas, cost):
    total = tank = start = 0
    for i, (g, c) in enumerate(zip(gas, cost)):
        diff = g - c
        total += diff
        tank += diff
        if tank < 0:
            start = i + 1
            tank = 0
    return start if total >= 0 else -1
```

### Assign cookies / boats to save people
Two pointers after sorting: smallest with smallest, or smallest with largest that fits.

```python
def num_rescue_boats(people, limit):
    people.sort()
    lo, hi, boats = 0, len(people) - 1, 0
    while lo <= hi:
        if people[lo] + people[hi] <= limit:
            lo += 1
        hi -= 1
        boats += 1
    return boats
```

### Partition labels
```python
def partition_labels(s):
    last = {ch: i for i, ch in enumerate(s)}
    res, start, end = [], 0, 0
    for i, ch in enumerate(s):
        end = max(end, last[ch])
        if i == end:
            res.append(end - start + 1)
            start = i + 1
    return res
```

### Task scheduling with cooldown
See heap version in file 07; when only the maximum frequency matters, it reduces to
a formula: `max((max_freq - 1) * (n + 1) + count_of_max, len(tasks))`.

## 3. When greedy fails — know the counterexamples

- **Coin change** with arbitrary denominations: greedy fails (e.g., coins
  `[1, 3, 4]`, amount 6 → greedy picks 4+1+1, optimal is 3+3). Use DP.
- **0/1 knapsack**: greedy by value/weight ratio fails. Use DP.
- **Longest increasing subsequence**: greedy with local choices fails. Use DP/LIS.

If the exchange argument is not obvious, DP is usually the safer answer.

## 4. Greedy + sorting strategies

| Sort key | Use when |
|----------|----------|
| by end | maximize count of non-overlapping intervals |
| by start | merge intervals, scheduling from time 0 |
| by ratio (value/weight) | fractional knapsack (not 0/1) |
| by deadline | min lateness / job sequencing |
| by difference | two-city scheduling (`costA - costB`) |

### Two-city scheduling (min cost)
```python
def two_city(costs):
    costs.sort(key=lambda x: x[0] - x[1])
    n = len(costs) // 2
    return sum(c[0] for c in costs[:n]) + sum(c[1] for c in costs[n:])
```

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Merge Intervals | sort by start | Medium |
| Insert Interval | three-phase scan | Medium |
| Non-overlapping Intervals | sort by end | Medium |
| Minimum Number of Arrows to Burst Balloons | sort by end | Medium |
| Meeting Rooms I / II | sort / heap | Easy/Medium |
| Interval List Intersections | two pointers | Medium |
| Jump Game I / II | reach / levels | Medium |
| Gas Station | running deficit | Medium |
| Assign Cookies | two pointers | Easy |
| Boats to Save People | two pointers | Medium |
| Partition Labels | last occurrence | Medium |
| Candy | two passes | Hard |
| Task Scheduler | formula / heap | Medium |
| Two City Scheduling | sort by difference | Medium |
| Non-decreasing Array | local fix | Medium |
| Best Time to Buy and Sell Stock II | sum positive diffs | Easy |
