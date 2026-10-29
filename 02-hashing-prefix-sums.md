# 02 — Hashing, Prefix Sums & Interval Arithmetic

## 1. Hash-map patterns

### Seen-map: "have I met the complement?"
```python
def two_sum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
    return []
```

### Prefix-state map: "has this running state occurred before?"
The most powerful hashing idea. For each index, compute a running state (sum, xor,
balance, remainder) and store the **first index** where it appeared.

```python
# T: prefix state — longest subarray with sum == k
def max_subarray_len_k(nums, k):
    first = {0: -1}                     # state 0 before the array starts
    running = 0
    best = 0
    for i, x in enumerate(nums):
        running += x
        if running - k in first:
            best = max(best, i - first[running - k])
        if running not in first:        # keep the earliest occurrence
            first[running] = i
    return best
```

Same template with `running` changed to:
- `running % k` → subarray sum divisible by k (count with a Counter).
- `running ^ x` → subarray xor == k.
- `+1 / -1 balance` → longest equal 0s and 1s.

### Frequency map for grouping / canonical keys
```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)
    for w in words:
        key = tuple(sorted(w))          # or Counter(w) as frozen key
        groups[key].append(w)
    return list(groups.values())
```
For lowercase words, a 26-length count tuple is faster: `key = [0]*26`.

### Index-value encoding (in-place hashing without extra space)
```python
def find_duplicates(nums):              # values in [1, n]
    res = []
    for x in nums:
        idx = abs(x) - 1
        if nums[idx] < 0:
            res.append(abs(x))
        else:
            nums[idx] *= -1
    return res
```

## 2. Prefix sums

### 1D prefix
```python
pref = [0] * (len(nums) + 1)
for i, x in enumerate(nums):
    pref[i + 1] = pref[i] + x
# sum(nums[l:r+1]) == pref[r + 1] - pref[l]
```

### 2D prefix (sum of any rectangle in O(1))
```python
def build_2d_prefix(mat):
    R, C = len(mat), len(mat[0])
    P = [[0] * (C + 1) for _ in range(R + 1)]
    for r in range(R):
        for c in range(C):
            P[r + 1][c + 1] = (mat[r][c] + P[r][c + 1]
                               + P[r + 1][c] - P[r][c])
    return P

def rect_sum(P, r1, c1, r2, c2):        # inclusive coordinates
    return (P[r2 + 1][c2 + 1] - P[r1][c2 + 1]
            - P[r2 + 1][c1] + P[r1][c1])
```

### Prefix + hash map = subarray sum problems
Covered by the template above. If you need the **count**, use `Counter` of states:
```python
from collections import Counter
def count_subarrays_sum_k(nums, k):
    count = Counter({0: 1})
    running = ans = 0
    for x in nums:
        running += x
        ans += count[running - k]
        count[running] += 1
    return ans
```

## 3. Difference arrays (range updates in O(1))

Apply many range increments, read the final array once.

```python
# T: difference array
def apply_range_adds(n, updates):
    diff = [0] * (n + 1)
    for l, r, v in updates:             # inclusive [l, r]
        diff[l] += v
        diff[r + 1] -= v
    out = [0] * n
    cur = 0
    for i in range(n):
        cur += diff[i]
        out[i] = cur
    return out
```
Use for: meeting-room counts, car-pooling capacity, corporate flight bookings,
range-addition problems, "max overlap" with known coordinate range.

## 4. Suffix / prefix products and "product except self"
```python
def product_except_self(nums):
    n = len(nums)
    out = [1] * n
    for i in range(1, n):
        out[i] = out[i - 1] * nums[i - 1]
    suffix = 1
    for i in range(n - 1, -1, -1):
        out[i] *= suffix
        suffix *= nums[i]
    return out
```

## 5. Interval patterns (overlap detection)

Sorted-by-start greedy approach (more in file 11):

```python
def can_attend_all(intervals):
    intervals.sort()
    for (s1, e1), (s2, e2) in zip(intervals, intervals[1:]):
        if s2 < e1:
            return False
    return True
```

Sweep line with events (start = +1, end = -1, process ends before starts for
"touching is OK"):
```python
def max_overlap(intervals):
    events = []
    for s, e in intervals:
        events.append((s, 1))
        events.append((e, -1))
    events.sort(key=lambda x: (x[0], x[1]))   # -1 before +1 at same time
    cur = best = 0
    for _, delta in events:
        cur += delta
        best = max(best, cur)
    return best
```

## Decision guide

| Question phrasing | Pattern |
|-------------------|---------|
| pair with a target | seen-map |
| subarray with sum/xor/count property | prefix state + map |
| group items by signature | canonical-key map |
| range updates, then query once | difference array |
| sum over rectangle many times | 2D prefix |
| "how many meetings at once" | sweep line |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Two Sum | seen-map | Easy |
| Contains Duplicate / Valid Anagram | frequency | Easy |
| Group Anagrams | canonical key | Medium |
| Longest Consecutive Sequence | set + expansion | Medium |
| Subarray Sum Equals K | prefix + count | Medium |
| Continuous Subarray Sum (divisible by k) | prefix % k | Medium |
| Contiguous Array (equal 0/1) | prefix balance | Medium |
| Subarray Sums Divisible by K | prefix count | Medium |
| Product of Array Except Self | prefix/suffix | Medium |
| Range Addition | difference array | Medium |
| Car Pooling / Meeting Rooms II | diff / sweep | Medium |
| Matrix Block Sum / NumMatrix | 2D prefix | Medium |
| First Missing Positive | index encoding | Hard |
