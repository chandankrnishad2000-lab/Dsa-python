# 06 — Trees & Binary Search Trees

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## 1. The four traversals (know all, iteratively too)

```python
def inorder(node, out):
    if node:
        inorder(node.left, out)
        out.append(node.val)
        inorder(node.right, out)

def preorder(node, out):
    if node:
        out.append(node.val)
        preorder(node.left, out)
        preorder(node.right, out)

def postorder(node, out):
    if node:
        postorder(node.left, out)
        postorder(node.right, out)
        out.append(node.val)

from collections import deque
def level_order(root):
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        res.append(level)
    return res
```

Iterative inorder (the one interviewers ask for):
```python
def inorder_iter(root):
    res, stack, cur = [], [], root
    while stack or cur:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        res.append(cur.val)
        cur = cur.right
    return res
```

### Choosing a traversal
| Question | Traversal |
|----------|-----------|
| sorted output of a BST | inorder |
| build/serialize a tree | preorder |
| bottom-up aggregation (height, sum, diameter) | postorder |
| level, width, right-side view | BFS |

## 2. Recursive DFS patterns (return a value upward)

### Maximum depth / balanced check
```python
def max_depth(root):
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

def is_balanced(root):
    def height(node):
        if not node:
            return 0
        lh, rh = height(node.left), height(node.right)
        if lh < 0 or rh < 0 or abs(lh - rh) > 1:
            return -1
        return 1 + max(lh, rh)
    return height(root) >= 0
```

### Diameter (return height, track global best)
```python
def diameter(root):
    best = 0
    def dfs(node):
        nonlocal best
        if not node:
            return 0
        l, r = dfs(node.left), dfs(node.right)
        best = max(best, l + r)
        return 1 + max(l, r)
    dfs(root)
    return best
```

### Path sum / max path sum
```python
def max_path_sum(root):
    best = float("-inf")
    def gain(node):
        nonlocal best
        if not node:
            return 0
        l = max(gain(node.left), 0)
        r = max(gain(node.right), 0)
        best = max(best, node.val + l + r)
        return node.val + max(l, r)
    gain(root)
    return best
```

### Lowest common ancestor (general binary tree)
```python
def lca(root, p, q):
    if not root or root is p or root is q:
        return root
    left = lca(root.left, p, q)
    right = lca(root.right, p, q)
    return root if left and right else left or right
```

## 3. BST invariants

In a BST: `left subtree < node < right subtree` for **all** descendants, not just children.
Inorder traversal is strictly increasing.

### Validate BST (pass bounds down, do not compare only with parent)
```python
def is_bst(root):
    def valid(node, lo, hi):
        if not node:
            return True
        if not (lo < node.val < hi):
            return False
        return (valid(node.left, lo, node.val)
                and valid(node.right, node.val, hi))
    return valid(root, float("-inf"), float("inf"))
```

### Search / insert / delete
```python
def search_bst(root, val):
    while root:
        if val == root.val:
            return root
        root = root.left if val < root.val else root.right
    return None

def insert_bst(root, val):
    if not root:
        return TreeNode(val)
    if val < root.val:
        root.left = insert_bst(root.left, val)
    else:
        root.right = insert_bst(root.right, val)
    return root

def delete_bst(root, val):
    if not root:
        return None
    if val < root.val:
        root.left = delete_bst(root.left, val)
    elif val > root.val:
        root.right = delete_bst(root.right, val)
    else:
        if not root.left:
            return root.right
        if not root.right:
            return root.left
        succ = root.right
        while succ.left:
            succ = succ.left
        root.val = succ.val
        root.right = delete_bst(root.right, succ.val)
    return root
```

### k-th smallest (iterative inorder, O(h + k))
```python
def kth_smallest(root, k):
    stack, cur = [], root
    while stack or cur:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        k -= 1
        if k == 0:
            return cur.val
        cur = cur.right
```

### Convert sorted array → balanced BST
```python
def sorted_array_to_bst(nums):
    def build(lo, hi):
        if lo > hi:
            return None
        mid = (lo + hi) // 2
        node = TreeNode(nums[mid])
        node.left = build(lo, mid - 1)
        node.right = build(mid + 1, hi)
        return node
    return build(0, len(nums) - 1)
```

## 4. Serialization / construction

```python
def serialize(root):
    if not root:
        return "N"
    return f"{root.val},{serialize(root.left)},{serialize(root.right)}"

def deserialize(data):
    tokens = iter(data.split(","))
    def build():
        tok = next(tokens)
        if tok == "N":
            return None
        node = TreeNode(int(tok))
        node.left = build()
        node.right = build()
        return node
    return build()
```

Build from preorder + inorder:
```python
def build_tree(preorder, inorder):
    idx = {v: i for i, v in enumerate(inorder)}
    def build(lo, hi):
        if lo > hi:
            return None
        val = preorder.pop(0)
        root = TreeNode(val)
        mid = idx[val]
        root.left = build(lo, mid - 1)
        root.right = build(mid + 1, hi)
        return root
    return build(0, len(inorder) - 1)
```

## 5. Common global-state tricks

- **Root-to-leaf paths**: pass `path + [node.val]`; copy at leaf.
- **Count nodes in complete tree**: compare left/right heights, O(log² n).
- **Right side view**: BFS taking the last of each level.
- **Zigzag order**: BFS, reverse alternate levels (or use a deque).
- **Boundary traversal**: left boundary + leaves + right boundary reversed.

## Complexity notes

- Balanced tree height `h = O(log n)`; skewed tree `h = O(n)`.
- DFS recursion uses O(h) stack. Python default recursion limit ~1000, so set it higher for chains.
- Inorder/BST validation is O(n) time, O(h) space.
- BFS uses O(width) queue space.

## Practice set

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Maximum Depth of Binary Tree | DFS | Easy |
| Same Tree / Symmetric Tree | DFS | Easy |
| Invert Binary Tree | DFS | Easy |
| Diameter of Binary Tree | postorder | Easy |
| Balanced Binary Tree | postorder | Easy |
| Path Sum / Path Sum II | DFS + paths | Easy/Medium |
| Binary Tree Level Order Traversal | BFS | Medium |
| Zigzag Level Order | BFS | Medium |
| Right Side View | BFS | Medium |
| Lowest Common Ancestor | postorder | Medium |
| Validate BST | bounds | Medium |
| Kth Smallest in BST | inorder | Medium |
| Construct Tree from Preorder+Inorder | recursion + map | Medium |
| Serialize and Deserialize Binary Tree | preorder | Hard |
| Binary Tree Maximum Path Sum | postorder | Hard |
| Count Complete Tree Nodes | height compare | Medium |
| Flatten Binary Tree to Linked List | preorder | Medium |
