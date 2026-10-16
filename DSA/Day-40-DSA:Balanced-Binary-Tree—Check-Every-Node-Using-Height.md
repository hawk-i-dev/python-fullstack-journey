Day 40 DSA: **Balanced Binary Tree — Check Every Node Using Height**

Today, you’ll determine whether a binary tree is **height-balanced**.

This is a useful continuation of Day 39 because we’ll reuse the same height formula:

```python
height = 1 + max(left_height, right_height)
```

Before starting, the answer to our previous practice question was:

```text
Left height = 2, right height = 2

Height returned    = 1 + max(2, 2) = 3 nodes
Diameter candidate = 2 + 2        = 4 edges
```

Today, instead of adding the heights for a diameter, we’ll compare their **difference**.

Imagine two towers attached to every junction in a tree. Their heights can differ slightly, but neither should be more than one level taller than the other.

A binary tree is height-balanced when, **at every node**:

```python
abs(left_height - right_height) <= 1
```

`abs()` gives the nonnegative difference:

```python
abs(3 - 1)  # 2
abs(1 - 3)  # 2
```

We assume valid binary-tree structure. Values and BST ordering do not affect this check.

Consider:

```text
        1
       / \
      2   3
     /
    4
```

At node `1`, the left subtree has height `2` and the right has height `1`. The difference is `1`, so that node passes.

At node `2`, the heights are `1` and `0`. It also passes.

The leaves pass because both child heights are `0`.

```python
is_balanced(root)  # True
```

Now consider:

```text
      1
     /
    2
   /
  3
```

At root `1`:

```text
Left height  = 2
Right height = 0
Difference   = 2
```

That exceeds `1`, so the tree is unbalanced.

**The 80/20 idea**

Remember these three rules:

1. Calculate child heights before checking the parent.
2. A node passes only when its child heights differ by at most `1`.
3. **Both subtrees must also be balanced.**

Checking only the root is insufficient.

```text
           1
          / \
         2   3
        /     \
       4       5
      /         \
     6           7
```

The root’s left and right subtree heights are both `3`, so the root’s difference is `0`.

But node `2` has child heights `2` and `0`. Node `3` has the reverse. Both are unbalanced.

The entire tree must therefore return `False`.

**Keep height, diameter, and balance separate**

This table connects the last two lessons:

| Question | Calculation |
|---|---|
| How tall is this subtree? | `1 + max(left_height, right_height)` |
| How long is a path through this node? | `left_height + right_height` |
| Is this node locally balanced? | `abs(left_height - right_height) <= 1` |

We continue using **height in nodes**:

```text
Empty subtree → height 0
Leaf          → height 1
```

Balance is about height difference, not the number of nodes in each subtree. Two subtrees can contain different numbers of nodes and still have similar heights.

**Think before coding: return height or report failure**

A straightforward approach checks balance at every node by recalculating subtree heights. That repeats work and can take `O(n²)` time in the worst case.

We can combine height calculation and balance checking in one traversal.

Give the helper this contract:

```text
Return the subtree’s height if it is balanced.
Return -1 if it is unbalanced.
```

Here, `-1` is a special failure marker, often called a *sentinel*. It is safe because real heights under our convention are never negative.

```text
0, 1, 2, ... → valid height
-1          → unbalanced subtree
```

At each node:

1. Get the left result. If it is `-1`, pass the failure upward.
2. Get the right result. If it is `-1`, pass the failure upward.
3. If the height difference exceeds `1`, return `-1`.
4. Otherwise, return the normal height.

This uses **postorder DFS**: children first, then parent.

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


def is_balanced(root: TreeNode | None) -> bool:
    def checked_height(node: TreeNode | None) -> int:
        if node is None:
            return 0

        left_height = checked_height(node.left)

        if left_height == -1:
            return -1

        right_height = checked_height(node.right)

        if right_height == -1:
            return -1

        if abs(left_height - right_height) > 1:
            return -1

        return 1 + max(left_height, right_height)

    return checked_height(root) != -1


balanced_tree = TreeNode(
    1,
    TreeNode(2, left=TreeNode(4)),
    TreeNode(3),
)

unbalanced_tree = TreeNode(
    1,
    left=TreeNode(
        2,
        left=TreeNode(3),
    ),
)

print(is_balanced(balanced_tree))    # True
print(is_balanced(unbalanced_tree))  # False
```

The public function returns a boolean. The helper returns a height or the failure marker.

```python
return checked_height(root) != -1
```

This means:

> “The tree is balanced if its helper result is not the failure marker.”

The function does not modify the tree.

**Dry run: the unbalanced chain**

```text
      1
     /
    2
   /
  3
```

| Node | Left height | Right height | Difference | Helper returns |
|---|---:|---:|---:|---:|
| `3` | 0 | 0 | 0 | `1` |
| `2` | 1 | 0 | 1 | `2` |
| `1` | 2 | 0 | 2 | `-1` |

The public function evaluates:

```python
-1 != -1  # False
```

Notice that node `2` is balanced even though the whole tree is not.

**Why failure must travel upward**

Suppose a left subtree returns `-1`.

That does **not** mean its height is minus one. It means the subtree is already known to be unbalanced.

So we immediately return `-1`:

```python
if left_height == -1:
    return -1
```

We must not use that value in normal height arithmetic.

Also, once one subtree is unbalanced, checking the other cannot make the entire tree balanced. Early return avoids unnecessary work.

This algorithm answers “Is the whole tree balanced?” If you need a report of **every** unbalanced node, you must continue traversing instead of stopping at the first failure.

**Why the algorithm works**

An empty subtree is balanced and has height `0`.

For each nonempty subtree:

- If either child subtree is unbalanced, the current subtree is unbalanced.
- If both children are balanced but their heights differ by more than `1`, the current subtree is unbalanced.
- Otherwise, the current subtree is balanced, and its height is `1 + max(...)`.

Those are exactly the conditions in the definition, applied at every node.

**Complexity and common mistakes**

Let `n` be the node count and `h` the tree height.

- **Worst-case time: `O(n)`** — every node is processed at most once.
- **Auxiliary space: `O(h)`** — recursive stack.
- Deep chains may exceed Python’s recursion limit.

Early exits can reduce work, but the worst-case time remains linear.

| Mistake | Why it fails |
|---|---|
| Check only the root | A lower subtree may be unbalanced |
| Compare subtree node counts | Balance is defined using heights |
| Require equal heights | A difference of `1` is allowed |
| Forget `abs()` | A large negative difference can slip through |
| Treat `-1` as a real height | Corrupts later calculations |
| Return `bool(height)` | An empty balanced tree has height `0`, which is falsy |
| Assume this function balances the tree | It only checks; it changes no connections |

Also, **balanced does not mean BST**:

```text
      1
     / \
    9   2
```

This tree is height-balanced but violates BST ordering.

**Tests to run**

```python
assert is_balanced(None) is True
assert is_balanced(TreeNode(10)) is True

# A difference of one is allowed.
assert is_balanced(
    TreeNode(1, left=TreeNode(2))
) is True

assert is_balanced(balanced_tree) is True
assert is_balanced(unbalanced_tree) is False

# Root heights match, but descendants are unbalanced.
hidden_imbalance = TreeNode(
    1,
    TreeNode(2, left=TreeNode(4, left=TreeNode(6))),
    TreeNode(3, right=TreeNode(5, right=TreeNode(7))),
)
assert is_balanced(hidden_imbalance) is False

# Values do not determine height balance.
not_a_bst = TreeNode(1, TreeNode(9), TreeNode(2))
assert is_balanced(not_a_bst) is True
```

**Your assignment**

Build Tree A:

```text
          10
         /  \
        4    20
       / \     \
      2   6     30
         /
        5
```

Expected:

```python
is_balanced(tree_a)  # True
```

Now create Tree B by adding node `7` as the left child of node `5`:

```text
          10
         /  \
        4    20
       / \     \
      2   6     30
         /
        5
       /
      7
```

Expected:

```python
is_balanced(tree_b)  # False
```

These are ordinary binary trees, so placing `7` there is allowed; our task checks structure, not BST ordering.

For Tree B, node `6` has:

```text
Left height  = 2
Right height = 0
Difference   = 2 → unbalanced
```

Complete these tasks:

1. Implement `is_balanced()` independently.
2. Trace the helper’s return values for both trees.
3. Explain where Tree B first reports `-1` during the left-first traversal.
4. Test empty, single-node, two-node, chain, and hidden-imbalance cases.

As an extension, write a helper that returns a pair:

```python
(height, balanced)
```

That version keeps the actual height separate from the boolean result and avoids using a sentinel.

At **institute level**, explain why the condition must hold at every node. Be able to distinguish the height formula, diameter formula, and balance check without memorizing the entire program.

At **corporate level**, distinguish validation from repair. This checker does not perform rotations or maintain balance after updates. For frequent checks on a changing tree, repeatedly scanning every node may be too expensive; a suitable tree structure can maintain height information during updates.

At **client level**, clarify what “balanced” means. Similar branch heights, equal numbers of employees, and evenly distributed workload are different requirements. This algorithm checks only the specified height condition.

For your GitHub lesson file, use:

```text
Day-40-DSA-Balanced-Binary-Tree.md
```

Before coding, try this: **if the left height is `3`, the right height is `1`, and both child subtrees are balanced, should this node return a height or `-1`?**
