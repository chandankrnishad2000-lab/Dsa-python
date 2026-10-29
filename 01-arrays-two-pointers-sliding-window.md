# 01 — Arrays: Two Pointers & Sliding Window

The highest-yield pattern family in interviews. Master these four variants:
opposite-direction, same-direction (fast/slow), fixed window, variable window.

## 1. Opposite-direction two pointers

Use when the array is **sorted** or when you can shrink from both ends.

### Template: two-sum on sorted array
```python
# T: opposite pointers on sorted array
def two_sum_sorted(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        s = nums[lo] + nums[hi]
        if s == target:
            return [lo, hi]
        if s < target:
            lo += 1
        else:
            hi -= 1
    return []
```
Time O(n), space O(1).

### Variant: container with most water
```python
def max_area(height):
    lo, hi, best = 0, len(height) - 1, 0
    while lo < hi:
        best = max(best, (hi - lo) * min(height[lo], height[hi]))
        if height[lo] < height[hi]:
            lo += 1
        else:
            hi -= 1
    return best
```
Why move the shorter wall: the width always decreases, so only a taller wall can improve.

### Variant: in-place removal / compaction
```python
def remove_duplicates(nums):
    slow = 0
    for fast in range(len(nums)):
        if slow == 0 or nums[fast] != nums[slow - 1]:
            nums[slow] = nums[fast]
            slow += 1
    return slow
```

### Variant: three-sum (sort + fix one + two pointers)
```python
def three_sum(nums):
    nums.sort()
    res = []
    n = len(nums)
    for i in range(n - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue                      # skip duplicate anchors
        lo, hi = i + 1, n - 1
        while lo < hi:
            s = nums[i] + nums[lo] + nums[hi]
            if s < 0:
                lo += 1
            elif s > 0:
                hi -= 1
            else:
                res.append([nums[i], nums[lo], nums[hi]])
                lo += 1
                hi -= 1
                while lo < hi and nums[lo] == nums[lo - 1]:
                    lo += 1
                while lo < hi and nums[hi] == nums[hi + 1]:
                    hi -= 1
    return res
```
Time O(n²), space O(1) excluding output.

## 2. Fast/slow same-direction pointers

Use for in-place partitioning, cycle detection in arrays, and "move zeros" style problems.

```python
# T: partition / stable compaction
def move_zeroes(nums):
    write = 0
    for read in range(len(nums)):
        if nums[read] != 0:
            nums[write], nums[read] = nums[read], nums[write]
            write += 1
```

Dutch-national-flag (three-way partition) for sort colors:
```python
def sort_colors(nums):
    lo, mid, hi = 0, 0, len(nums) - 1
    while mid <= hi:
        if nums[mid] == 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi -= 1
```

## 3. Fixed-size sliding window

Use when the window length is given, or when you need exactly k elements.

```python
# T: fixed window — maintain a running aggregate of the last k elements
def max_sum_k(nums, k):
    window = sum(nums[:k])
    best = window
    for i in range(k, len(nums)):
        window += nums[i] - nums[i - k]
        best = max(best, window)
    return best
```

With counts/multiplicity (anagram window):
```python
from collections import Counter

def find_anagrams(s, p):
    need = Counter(p)
    k = len(p)
    window = Counter(s[:k])
    res = [0] if window == need else []
    for i in range(k, len(s)):
        window[s[i]] += 1
        left = s[i - k]
        window[left] -= 1
        if window[left] == 0:
            del window[left]
        if window == need:
            res.append(i - k + 1)
    return res
```

## 4. Variable-size sliding window (the money pattern)

Use for "longest / shortest subarray satisfying a condition" where validity is **monotone**:
once a window is invalid, extending it never makes it valid.

```python
# T: variable window — longest window satisfying `valid`
def longest_window(nums, k):
    left = 0
    state = 0                     # aggregate: sum, counts, distinct count...
    best = 0
    for right, x in enumerate(nums):
        state += x                # 1) expand
        while state > k:          # 2) shrink until valid
            state -= nums[left]
            left += 1
        best = max(best, right - left + 1)   # 3) record
    return best
```

### Longest substring without repeating characters
```python
def length_of_longest_substring(s):
    last = {}
    left = best = 0
    for right, ch in enumerate(s):
        if ch in last and last[ch] >= left:
            left = last[ch] + 1
        last[ch] = right
        best = max(best, right - left + 1)
    return best
```

### Minimum window substring (shortest window that covers a target)
```python
from collections import Counter

def min_window(s, t):
    need = Counter(t)
    missing = len(t)
    left = 0
    best = (float("inf"), 0, 0)
    for right, ch in enumerate(s):
        if need[ch] > 0:
            missing -= 1
        need[ch] -= 1
        while missing == 0:
            if right - left + 1 < best[0]:
                best = (right - left + 1, left, right)
            need[s[left]] += 1
            if need[s[left]] > 0:
                missing += 1
            left += 1
    return "" if best[0] == float("inf") else s[best[1]:best[2] + 1]
```

### At-most-k distinct (convert "exactly k" into two "at most" calls)
```python
def at_most_k_distinct(nums, k):
    from collections import defaultdict
    left, best = 0, 0
    count = defaultdict(int)
    for right, x in enumerate(nums):
        count[x] += 1
        while len(count) > k:
            count[nums[left]] -= 1
            if count[nums[left]] == 0:
                del count[nums[left]]
            left += 1
        best = max(best, right - left + 1)
    return best

def exactly_k_distinct(nums, k):
    return at_most_k_distinct(nums, k) - at_most_k_distinct(nums, k - 1)
```

## Decision guide

| Signal | Pattern |
|--------|---------|
| sorted array, pair/triplet sums | opposite pointers |
| in-place removal / partition | fast/slow |
| "subarray of size k" | fixed window |
| "longest/shortest subarray with property" | variable window |
| "at most k" / "exactly k" | at-most trick |
| string anagram / permutation | fixed window + Counter |

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Two Sum II (sorted) | opposite | Easy |
| Valid Palindrome | opposite | Easy |
| Container With Most Water | opposite | Medium |
| 3Sum / 3Sum Closest | opposite + sort | Medium |
| Trapping Rain Water | opposite / prefix | Hard |
| Remove Duplicates from Sorted Array | fast/slow | Easy |
| Move Zeroes | fast/slow | Easy |
| Sort Colors | 3-way partition | Medium |
| Maximum Average Subarray I | fixed | Easy |
| Permutation in String / Find All Anagrams | fixed + Counter | Medium |
| Longest Substring Without Repeating Characters | variable | Medium |
| Longest Repeating Character Replacement | variable | Medium |
| Minimum Window Substring | variable + need map | Hard |
| Subarrays with K Different Integers | at-most trick | Hard |
| Sliding Window Maximum | monotonic deque (see 03) | Hard |
