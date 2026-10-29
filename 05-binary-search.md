# 05 — Binary Search

Binary search is not "search a sorted array". It is **monotonic predicate search**:
find the first/last index where `pred(i)` becomes true.

## 1. The only template you need

```python
# T: lower_bound — first index where pred(i) is True
def lower_bound(lo, hi, pred):
    # pred is False...False True...True on [lo, hi)
    while lo < hi:
        mid = (lo + hi) // 2
        if pred(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo        # may equal hi if pred never true
```

Everything else is a choice of `pred` and `lo/hi` bounds.

| Goal | Predicate | Result |
|------|-----------|--------|
| first index with `arr[i] >= x` | `arr[i] >= x` | lower bound |
| first index with `arr[i] > x` | `arr[i] > x` | upper bound |
| count of `x` | `upper - lower` | |
| minimum feasible answer | `feasible(mid)` | answer |
| maximum feasible answer | `not feasible(mid)` then `-1` | answer |

Do not mix in `while lo <= hi` habits; pick one template and be consistent.
Overflow is not a concern in Python, but `lo + (hi - lo) // 2` is good style.

## 2. Standard search variants

```python
from bisect import bisect_left, bisect_right

def search(nums, target):
    i = bisect_left(nums, target)
    return i if i < len(nums) and nums[i] == target else -1
```

### First and last position
```python
def search_range(nums, target):
    lo = bisect_left(nums, target)
    hi = bisect_right(nums, target) - 1
    if lo <= hi and lo < len(nums) and nums[lo] == target:
        return [lo, hi]
    return [-1, -1]
```

### Rotated sorted array
```python
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:            # left half sorted
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:                                 # right half sorted
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

### Find minimum in rotated array (with duplicates)
```python
def find_min(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] > nums[hi]:
            lo = mid + 1
        elif nums[mid] < nums[hi]:
            hi = mid
        else:
            hi -= 1                 # duplicates: shrink cautiously
    return nums[lo]
```

## 3. Binary search on the answer (the high-value skill)

The array is not sorted; the **feasibility function** is monotone.
Ask: "If the answer is X, can I check it in O(n)?" Then binary search X.

### Koko eating bananas
```python
def min_eating_speed(piles, h):
    def feasible(speed):
        return sum((p + speed - 1) // speed for p in piles) <= h
    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Ship packages within D days
```python
def ship_within_days(weights, days):
    def feasible(cap):
        need, cur = 1, 0
        for w in weights:
            if w > cap:
                return False
            if cur + w > cap:
                need += 1
                cur = 0
            cur += w
        return need <= days
    lo, hi = max(weights), sum(weights)
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Split array largest sum / minimize max
Same shape: minimize the maximum group sum under a count constraint.

### Median of two sorted arrays
Binary search the partition of the smaller array.
```python
def find_median_sorted(a, b):
    if len(a) > len(b):
        a, b = b, a
    m, n = len(a), len(b)
    half = (m + n + 1) // 2
    lo, hi = 0, m
    while lo <= hi:
        i = (lo + hi) // 2
        j = half - i
        a_left = a[i - 1] if i > 0 else float("-inf")
        a_right = a[i] if i < m else float("inf")
        b_left = b[j - 1] if j > 0 else float("-inf")
        b_right = b[j] if j < n else float("inf")
        if a_left <= b_right and b_left <= a_right:
            if (m + n) % 2:
                return max(a_left, b_left)
            return (max(a_left, b_left) + min(a_right, b_right)) / 2
        if a_left > b_right:
            hi = i - 1
        else:
            lo = i + 1
```

## 4. Binary search with a custom condition (peak, 2D matrix)

### Find peak element
```python
def find_peak(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < nums[mid + 1]:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

### Search a 2D matrix (sorted rows and columns)
Treat it as a flattened sorted array, or staircase-search from the top-right:
```python
def search_matrix(mat, target):
    if not mat or not mat[0]:
        return False
    r, c = 0, len(mat[0]) - 1
    while r < len(mat) and c >= 0:
        if mat[r][c] == target:
            return True
        if mat[r][c] > target:
            c -= 1
        else:
            r += 1
    return False
```

## Decision guide

| Signal | Pattern |
|--------|---------|
| sorted array, find value/boundary | lower/upper bound |
| rotated sorted array | diagonal comparison |
| "minimize the maximum" / "maximize the minimum" | binary search on answer |
| "can we do it in k days / with capacity C" | feasibility + binary search |
| unsorted array but one peak | local slope comparison |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Binary Search | template | Easy |
| First Bad Version | lower bound | Easy |
| Search Insert Position | lower bound | Easy |
| Sqrt(x) | answer search | Easy |
| Find First and Last Position | two bounds | Medium |
| Search in Rotated Sorted Array I / II | rotated | Medium |
| Find Minimum in Rotated Sorted Array | rotated | Medium |
| Find Peak Element | slope | Medium |
| Koko Eating Bananas | answer search | Medium |
| Capacity to Ship Packages in D Days | answer search | Medium |
| Split Array Largest Sum | answer search | Hard |
| Median of Two Sorted Arrays | partition search | Hard |
| Kth Smallest in Sorted Matrix | heap or answer search | Medium |
