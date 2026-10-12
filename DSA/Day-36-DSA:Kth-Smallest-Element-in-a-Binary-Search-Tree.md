Day 36 DSA: **Kth Smallest Element in a Binary Search Tree**

You can now validate, search, insert, and delete in a BST. Today, you’ll answer a new question:

> “What is the kth smallest value?”

This combines Day 31’s inorder traversal with the BST ordering you’ve been practicing.

Imagine numbered books arranged so you can collect them from smallest to largest. To find the third-smallest book, count the books as you collect them and stop at number three.

Here, **k is a position, starting from 1**. It is not the value we are searching for.

```text
           8
         /   \
        3     10
       / \      \
      1   6      14
         / \     /
        4   7   13
```

The values in ascending order are:

```text
Values:    1   3   4   6   7   8   10   13   14
Position:  1   2   3   4   5   6    7    8    9
```

Therefore:

```python
k = 1  # answer: 1
k = 3  # answer: 4
k = 6  # answer: 8
```

We’ll assume a valid strict BST with integer values and no duplicates.

**The 80/20 idea**

Remember these three rules:

1. Inorder traversal visits **Left → Root → Right**.
2. In a strict BST, that produces values in increasing order.
3. Count nodes when you visit them; stop at the kth visit.

The central connection is:

```text
BST + inorder traversal → sorted visit order
Sorted visit order + counter → kth smallest
```

**Start with the straightforward approach**

You could collect all inorder values into a list:

```python
values = [1, 3, 4, 6, 7, 8, 10, 13, 14]
answer = values[k - 1]
```

The index is `k - 1` because positions start at `1`, while Python list indexes start at `0`.

This works, but collecting every value takes `O(n)` time and `O(n)` output storage, even when you only want the smallest value.

Instead, generate the values in order and stop once you have the answer.

**How to think before coding**

Suppose we want `k = 3`.

We must reach the smallest node first. In a BST, follow left children until there are no more.

```text
8 → 3 → 1
```

But after visiting `1`, how will we remember to return to `3` and later `8`?

Use a **stack** to remember nodes whose turns are still waiting.

```text
Stack shown bottom → top:

[8, 3, 1]
```

Pop `1`: that is our first visit.

Then pop `3`: that is our second visit.

Before returning to `8`, we must process `3`’s right subtree:

```text
      6
     / \
    4   7
```

Go left through that subtree, saving `6` and `4`. Pop `4`: that is our third visit, so return `4`.

The stack helps simulate the “return to the parent” behavior of recursive inorder traversal.

Keep these variables:

| Variable | Purpose |
|---|---|
| `current` | Subtree whose left path we are exploring |
| `stack` | Nodes waiting to be visited |
| `visited` | Number of nodes already processed |
| `k` | Requested position |

A node is **visited when popped**, not when pushed. Pushing a node only records that we must return to it after its left subtree.

**Convert the idea into an algorithm**

Repeat these steps:

1. Push `current` and keep moving left until `current` is `None`.
2. Pop the next node.
3. Increase `visited`.
4. If `visited == k`, return that node’s value.
5. Move to its right child and repeat.

If the traversal finishes before reaching `k`, the requested position does not exist.

Our function will raise `ValueError` when:

- `k < 1`.
- The tree contains fewer than `k` nodes, including an empty tree.

We assume the caller supplies an integer `k`.

**Complete runnable Python**

```python
class TreeNode:
    def __init__(
        self,
        value: int,
        left: "TreeNode | None" = None,
        right: "TreeNode | None" = None,
    ):
        self.value = value
        self.left = left
        self.right = right


def kth_smallest(root: TreeNode | None, k: int) -> int:
    if k < 1:
        raise ValueError("k must be at least 1")

    stack = []
    current = root
    visited = 0

    while current is not None or stack:
        # Save the path to the smallest unvisited node.
        while current is not None:
            stack.append(current)
            current = current.left

        # Its left subtree is complete: visit this node.
        current = stack.pop()
        visited += 1

        if visited == k:
            return current.value

        # Next, explore this node's right subtree.
        current = current.right

    raise ValueError("k exceeds the number of nodes")


root = TreeNode(
    8,
    TreeNode(
        3,
        TreeNode(1),
        TreeNode(6, TreeNode(4), TreeNode(7)),
    ),
    TreeNode(
        10,
        right=TreeNode(14, left=TreeNode(13)),
    ),
)

print(kth_smallest(root, 1))  # 1
print(kth_smallest(root, 3))  # 4
print(kth_smallest(root, 6))  # 8
print(kth_smallest(root, 9))  # 14
```

This function does not modify the tree.

The outer condition uses `or`:

```python
while current is not None or stack:
```

Even if `current` is `None`, the stack may contain nodes waiting to be processed. Conversely, an empty stack does not mean we are finished if `current` points to another subtree.

**Dry run for `k = 3`**

The rightmost stack entry is the top.

| Action | Stack afterward | Visits | Result |
|---|---|---:|---|
| Push `8`, `3`, `1` | `[8, 3, 1]` | 0 | Smallest node is ready |
| Pop `1` | `[8, 3]` | 1 | Continue |
| `1` has no right child; pop `3` | `[8]` | 2 | Continue into `3.right` |
| Push `6`, then `4` | `[8, 6, 4]` | 2 | Next smallest is ready |
| Pop `4` | `[8, 6]` | 3 | Return `4` |

We return without visiting `6`, `7`, `8`, `10`, `13`, or `14`.

Some unvisited nodes are already on the stack. That is expected: storing a node for later is different from processing it.

**Why this works**

For each node, inorder traversal processes:

```text
All smaller values in its left subtree
→ the node itself
→ all larger values in its right subtree
```

The stack preserves that order. Therefore, after the kth pop, exactly `k - 1` smaller values have already been visited. The current value must be the kth smallest.

This depends on the input being a BST. An arbitrary binary tree’s inorder traversal is not necessarily sorted.

**Complexity**

Let `n` be the node count and `h` the tree height.

- For a valid position, time is **`O(h + k)`**, with a worst-case bound of **`O(n)`**.
- Auxiliary space is **`O(h)`** for the stack.
- If `k` exceeds the node count, traversal takes **`O(n)`** before raising the error.

Why isn’t the time always `O(k)`? Reaching the first value may require following a long left path.

```text
        5
       /
      4
     /
    3
   /
  2
 /
1
```

Even for `k = 1`, we must follow the entire path to reach `1`.

For a balanced tree, the bound is `O(log n + k)`. This ordinary BST is not guaranteed to be balanced.

**Common mistakes**

| Mistake | Why it fails |
|---|---|
| Count when pushing | A parent is pushed before its smaller left descendants |
| Use preorder traversal | Values are not visited in sorted order |
| Use BFS | Level order does not provide sorted order |
| Use `and` in the outer condition | Traversal can stop while work remains |
| Forget `current = current.right` | Right subtrees are skipped |
| Use `values[k]` in the list approach | Produces an off-by-one error |
| Return `-1` for invalid input without a clear contract | `-1` could be a legitimate tree value |

Use these checks with the example tree:

```python
assert kth_smallest(root, 1) == 1
assert kth_smallest(root, 3) == 4
assert kth_smallest(root, 9) == 14

single = TreeNode(42)
assert kth_smallest(single, 1) == 42

negative_tree = TreeNode(
    0,
    TreeNode(-5),
    TreeNode(9),
)
assert kth_smallest(negative_tree, 1) == -5

# Invalid positions must raise ValueError.
for tree, k in [(root, 0), (root, 10), (None, 1)]:
    try:
        kth_smallest(tree, k)
    except ValueError:
        pass
    else:
        raise AssertionError("Expected ValueError")
```

**Your assignment: build a ranking query**

Construct this BST using your insertion function:

```python
[20, 10, 30, 5, 15, 25, 35, 13, 17]
```

```text
          20
         /  \
       10    30
      / \   / \
     5  15 25 35
       /  \
      13  17
```

Implement `kth_smallest(root, k)` independently.

Expected results:

| `k` | Answer |
|---:|---:|
| 1 | 5 |
| 3 | 13 |
| 5 | 17 |
| 9 | 35 |

Also verify that `k = 0`, `k = 10`, and an empty tree raise `ValueError`.

Then implement:

```python
def kth_largest(root, k):
    ...
```

Hint: reverse inorder traversal visits **Right → Root → Left**.

Expected results:

```python
kth_largest(root, 1)  # 35
kth_largest(root, 3)  # 25
kth_largest(root, 9)  # 5
```

For your dry run, record the stack and visited count while finding the third-smallest value, `13`.

At **institute level**, explain why counting happens on pop and why sorted inorder order depends on BST validity. Compare collecting the full inorder list with stopping early.

At **corporate level**, consider how frequently ranking queries occur. For many queries, a tree can store each node’s subtree size. If the left subtree contains `L` nodes, the current node has rank `L + 1` within its subtree. This allows an `O(h)` selection query, but every insertion and deletion must maintain those sizes correctly. Balancing is still needed to guarantee logarithmic height.

At **client level**, clarify what “third smallest” means when values tie. For prices `[10, 10, 20]`, the third item’s price is `20`, but a third distinct price does not exist. Today’s strict BST avoids duplicates, so a system that permits tied prices needs an explicit representation and ranking policy.

Also confirm the ordering field. A tree ordered by product ID cannot directly answer “third-cheapest product” using inorder traversal; that requires ordering by price.

Before coding, explain this sentence in your own words:

> “Pushing remembers a node for later; popping visits it after all smaller values in its left subtree.”
