# 12 — Tries & Bit Manipulation

## Part A — Tries (prefix trees)

A trie stores strings by shared prefixes. Operations are O(L) where L is the
string length, independent of dictionary size.

### Node + insert/search
```python
class TrieNode:
    __slots__ = ("children", "is_word")
    def __init__(self):
        self.children = {}
        self.is_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True

    def search(self, word):
        node = self._walk(word)
        return bool(node and node.is_word)

    def starts_with(self, prefix):
        return self._walk(prefix) is not None

    def _walk(self, s):
        node = self.root
        for ch in s:
            node = node.children.get(ch)
            if node is None:
                return None
        return node
```

### Word dictionary with wildcards (`.` matches any char)
```python
def search_wild(node, word, i):
    if i == len(word):
        return node.is_word
    ch = word[i]
    if ch == ".":
        return any(search_wild(child, word, i + 1)
                   for child in node.children.values())
    child = node.children.get(ch)
    return child is not None and search_wild(child, word, i + 1)
```

### Word Search II (trie + grid backtracking)
Build a trie of all words, DFS the grid, prune when the prefix is missing.
Store the word at terminal nodes to avoid reconstructing paths.

```python
def find_words(board, words):
    root = TrieNode()
    for w in words:
        node = root
        for ch in w:
            node = node.children.setdefault(ch, TrieNode())
        node.word = w

    R, C = len(board), len(board[0])
    res = []

    def dfs(r, c, node):
        ch = board[r][c]
        nxt = node.children.get(ch)
        if not nxt:
            return
        if getattr(nxt, "word", None):
            res.append(nxt.word)
            nxt.word = None                 # dedupe
        board[r][c] = "#"
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and board[nr][nc] != "#":
                dfs(nr, nc, nxt)
        board[r][c] = ch

    for r in range(R):
        for c in range(C):
            dfs(r, c, root)
    return res
```

### Other trie applications
- **Autocomplete / top-k frequent prefixes**: store a heap or count at each node.
- **Maximum XOR of two numbers**: binary trie (see below).
- **Longest common prefix**: walk while one child.
- **Aho-Corasick**: trie + failure links for multi-pattern matching (rare in interviews).

## Part B — Bit manipulation

### Core tricks

| Operation | Expression |
|-----------|------------|
| get bit i | `(x >> i) & 1` |
| set bit i | `x \| (1 << i)` |
| clear bit i | `x & ~(1 << i)` |
| toggle bit i | `x ^ (1 << i)` |
| lowest set bit | `x & -x` |
| clear lowest set bit | `x & (x - 1)` |
| is power of two | `x > 0 and x & (x - 1) == 0` |
| count set bits | `bin(x).count("1")` or loop `x &= x - 1` |
| swap without temp | `a ^= b; b ^= a; a ^= b` |
| all bits of length n | `(1 << n) - 1` |
| mask low n bits | `x & ((1 << n) - 1)` |

Python notes:
- Integers are arbitrary precision; `~x` is `-x - 1`. Mask explicitly when needed:
  `(~x) & ((1 << 32) - 1)`.
- `1 << 63` is fine; no overflow.

### Single number family (XOR)
```python
def single_number(nums):
    x = 0
    for n in nums:
        x ^= n
    return x                 # a ^ a == 0, a ^ 0 == a
```

Two unique numbers: XOR everything, isolate the lowest set bit, partition into two
groups, XOR each group.
```python
def single_number_iii(nums):
    x = 0
    for n in nums:
        x ^= n
    low = x & -x
    a = b = 0
    for n in nums:
        if n & low:
            a ^= n
        else:
            b ^= n
    return [a, b]
```

Three-times numbers: count bits modulo 3 (or use two masks `ones`/`twos`).

### Subsets via bitmask
```python
def subsets_bitmask(nums):
    n = len(nums)
    return [[nums[i] for i in range(n) if mask >> i & 1]
            for mask in range(1 << n)]
```

### Maximum XOR pair (binary trie)
```python
class BitTrie:
    def __init__(self, bits=32):
        self.root = {}
        self.bits = bits

    def insert(self, x):
        node = self.root
        for b in range(self.bits - 1, -1, -1):
            bit = (x >> b) & 1
            node = node.setdefault(bit, {})

    def max_xor(self, x):
        node, best = self.root, 0
        for b in range(self.bits - 1, -1, -1):
            bit = (x >> b) & 1
            want = bit ^ 1
            if want in node:
                best |= 1 << b
                node = node[want]
            else:
                node = node[bit]
        return best
```

### Bitmask DP reminder
`n ≤ 20` and "choose a subset" → iterate `mask in range(1 << n)`.
Submask enumeration for each mask:
```python
sub = mask
while sub:
    # process sub
    sub = (sub - 1) & mask
```
Total O(3^n).

## Practice set

| Problem | Topic | Difficulty |
|---------|-------|------------|
| Implement Trie | trie | Medium |
| Design Add and Search Words | trie + DFS | Medium |
| Replace Words | trie | Medium |
| Word Search II | trie + grid | Hard |
| Maximum XOR of Two Numbers | bit trie | Medium |
| Single Number I / II / III | XOR | Easy/Medium |
| Number of 1 Bits | bit count | Easy |
| Counting Bits | DP + bits | Easy |
| Reverse Bits | bit ops | Easy |
| Power of Two | low bit | Easy |
| Missing Number | XOR / sum | Easy |
| Subsets | bitmask | Medium |
| Maximum Product of Word Lengths | bitmask | Medium |
| Divide Two Integers | bit shifting | Medium |
