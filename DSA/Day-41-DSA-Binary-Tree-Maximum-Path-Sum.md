Day 41 DSA: **Binary Tree Maximum Path Sum — Keep Helpful Branches, Drop Harmful Ones**

Today, you’ll find the path with the **largest sum of node values**.

This builds directly on Day 39’s diameter pattern:

> Return one branch to the parent; consider two branches for a complete path.

The difference is what we measure.

| Problem | What we maximize |
|---|---|
| Diameter | Number of edges |
| Maximum path sum | Sum of node values |

Imagine connected rooms containing coins. Some rooms give you coins; others charge a fee, represented by negative values. You can choose where your route starts and ends, but you cannot revisit a room or split into multiple routes.

Our rules are:

- A path follows parent-child connections, in either direction.
- It contains at least one node.
- It does not have to pass through the root.
- Its endpoints do not have to be leaves.
- The input is a valid binary tree; BST ordering is unnecessary.

Consider:

```text
        -10
        /  \
       9    20
           /  \
          15   7
```

The best path is:

```text
15 → 20 → 7

Sum = 15 + 20 + 7 = 42
```

Including `-10` would reduce the sum. The answer is therefore:

```python
max_path_sum(root)  # 42
```

**The 80/20 idea**

At every node, calculate the best downward contribution from each child.

Then make two different decisions:

```text
Complete path through this node:
node value + helpful left contribution + helpful right contribution

Contribution returned to parent:
node value + the better helpful contribution
```

A negative child contribution is optional, so replace it with zero:

```python
left_gain = max(0, gain(node.left))
right_gain = max(0, gain(node.right))
```

Zero means **“do not take that child branch.”** It does not mean we delete a node or skip a negative node while still collecting its descendants.

These are the two formulas:

```python
# A complete path can connect both sides.
candidate = node.value + left_gain + right_gain

# A parent can extend only one side.
return node.value + max(left_gain, right_gain)
```

**Understand why the parent receives only one branch**

Suppose node `20` has children `15` and `7`.

```text
        20
       /  \
      15   7
```

The complete path through `20` is:

```text
15 → 20 → 7
```

Its sum is `42`.

But if a parent connects to `20`, we cannot return all `42` as an extendable contribution. That would ask the path to go through the parent and split toward both children.

Instead, return:

```text
20 + max(15, 7) = 35
```

That represents the single downward branch:

```text
20 → 15
```

The complete candidate and returned contribution answer different questions.

**Negative values: discard a branch, not the current node**

Consider:

```text
        5
       / \
     -4   6
```

At node `5`:

```text
Left contribution:  max(0, -4) = 0
Right contribution: max(0,  6) = 6
```

The best path is:

```text
5 → 6
Sum = 11
```

The negative left branch is unnecessary.

But a negative node can still belong to a useful branch:

```text
      5
     /
   -2
   /
 10
```

The contribution from node `-2` is:

```text
-2 + 10 = 8
```

That branch is helpful overall, so the complete path has sum:

```text
5 + (-2) + 10 = 13
```

We include `-2` because it is the connection to `10`. We cannot jump directly from `5` to `10`.

**Handle an all-negative tree correctly**

```text
       -3
       / \
     -5  -2
```

The best nonempty path is the single node `-2`.

```python
answer = -2
```

If we initialize the best answer to `0`, the function would incorrectly choose an empty path.

We’ll initialize:

```python
best = root.value
```

That guarantees the initial answer represents a real node.

For an empty tree, our function will raise `ValueError`, because no nonempty path exists.

**Define the helper before coding**

The helper’s contract is:

> `gain(node)` returns the largest sum of a downward path that starts at this node and follows at most one child at each step.

For a missing child, return `0` as a convenient “no branch” contribution.

At each existing node:

1. Calculate the children’s gains.
2. Replace negative child gains with zero.
3. Update the best complete path using both helpful sides.
4. Return the best single branch to the parent.

This is postorder DFS: calculate child results before combining them.

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


def max_path_sum(root: TreeNode | None) -> int:
    if root is None:
        raise ValueError("A nonempty tree is required")

    best = root.value

    def gain(node: TreeNode | None) -> int:
        nonlocal best

        if node is None:
            return 0

        left_gain = max(0, gain(node.left))
        right_gain = max(0, gain(node.right))

        # Complete path whose highest node is this node.
        candidate = node.value + left_gain + right_gain
        best = max(best, candidate)

        # Extendable branch for the parent.
        return node.value + max(left_gain, right_gain)

    gain(root)
    return best


root = TreeNode(
    -10,
    TreeNode(9),
    TreeNode(20, TreeNode(15), TreeNode(7)),
)

print(max_path_sum(root))  # 42
```

The function reads the tree without changing its values or connections. Each call creates its own `best`, so separate calls do not share results.

**Dry run**

For our main example, the nodes are processed in this order:

```text
9 → 15 → 7 → 20 → -10
```

The table shows child gains **after replacing negative gains with zero**:

| Node | Left gain | Right gain | Complete candidate | Gain returned | Best so far |
|---|---:|---:|---:|---:|---:|
| `9` | 0 | 0 | 9 | 9 | 9 |
| `15` | 0 | 0 | 15 | 15 | 15 |
| `7` | 0 | 0 | 7 | 7 | 15 |
| `20` | 15 | 7 | 42 | 35 | 42 |
| `-10` | 9 | 35 | 34 | 25 | 42 |

At the root, the candidate is:

```text
9 + (-10) + 35 = 34
```

That is smaller than the already discovered `42`, so the best answer remains `42`.

Also notice:

```text
gain(root) = 25
final answer = 42
```

Returning the root’s gain would be wrong. The best path may turn below the root.

**Why the algorithm works**

Every simple path has a highest node—the point where its two downward portions meet.

For each possible highest node:

- The left portion should use the best helpful downward branch.
- The right portion should use the best helpful downward branch.
- A negative portion should be omitted.
- The current node must be included.

We evaluate this candidate at every node, so we consider the highest node of the optimal path.

The returned gain preserves the best path a parent can legally extend.

**Keep the three recent tree problems separate**

| Problem | Child information | Local calculation | Return upward |
|---|---|---|---|
| Diameter | Heights | `left_h + right_h` | `1 + max(left_h, right_h)` |
| Balance | Heights or failure | Check height difference | Height or `-1` |
| Maximum path sum | Downward sums | `value + left_gain + right_gain` | `value + max(left_gain, right_gain)` |

For maximum path sum, the gains in this table have already been clamped to at least zero.

Do not copy formulas mechanically. First identify what the child’s return value means.

**Complexity and common mistakes**

For `n` nodes and tree height `h`:

- **Time: `O(n)`** — each node is processed once.
- **Auxiliary space: `O(h)`** — recursive stack.
- A deep chain may exceed Python’s recursion limit.

| Mistake | Why it fails |
|---|---|
| Initialize `best = 0` | Incorrect for an all-negative tree |
| Return both branches to the parent | Creates a fork rather than a simple path |
| Force every child branch into the candidate | Negative branches can reduce the answer |
| Skip a negative connecting node but keep its descendant | Breaks path connectivity |
| Return only `gain(root)` | Misses a better path below the root |
| Count edges instead of values | Solves diameter, not maximum path sum |

**Tests to run**

```python
assert max_path_sum(root) == 42

# A single node is a valid path.
assert max_path_sum(TreeNode(5)) == 5
assert max_path_sum(TreeNode(-8)) == -8

# All negative: choose the best individual node.
all_negative = TreeNode(-3, TreeNode(-5), TreeNode(-2))
assert max_path_sum(all_negative) == -2

# Ignore a harmful branch.
skip_branch = TreeNode(5, TreeNode(-4), TreeNode(6))
assert max_path_sum(skip_branch) == 11

# A negative connector can still lead to a helpful branch.
connector = TreeNode(
    5,
    left=TreeNode(-2, left=TreeNode(10)),
)
assert max_path_sum(connector) == 13

# Repeated calls must not share best values.
assert max_path_sum(root) == 42
assert max_path_sum(TreeNode(-8)) == -8

try:
    max_path_sum(None)
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError")
```

**Your assignment**

Build this tree:

```text
          10
         /  \
        2    10
       / \     \
      20  1    -25
               / \
              3   4
```

Expected maximum path sum:

```text
20 → 2 → 10 → 10

Sum = 42
```

There are two different nodes containing `10`; both can appear in the path.

Complete these tasks:

1. Implement `max_path_sum()` without copying.
2. Record each node’s child gains, complete candidate, and returned gain.
3. Explain why the branch starting at `-25` is omitted by its parent.
4. Test a single negative node and an all-negative tree.
5. Explain why returning both child gains to the parent would be invalid.

As an extension, return the actual path as well as its sum. Keep in mind that copying whole path lists at many recursive calls can increase the running time; do not assume the extended version automatically remains `O(n)`.

At **institute level**, explain the difference between the complete candidate and the extendable gain. Your dry run should explicitly show both.

At **corporate level**, specify whether the input can be empty and whether the result needs a sum, endpoints, or an entire path. Use an iterative postorder approach for trees that may be too deep for recursion.

At **client level**, clarify whether the route can start and end anywhere. “Maximum root-to-leaf sum” is a different requirement: the path must start at the root and reach a leaf, so optional-branch logic cannot be reused unchanged. Also distinguish node values from costs stored on edges.

For GitHub, use:

```text
Day-41-DSA-Binary-Tree-Maximum-Path-Sum.md
```

Try this before coding: **a node has value `5`, its left child returns gain `-3`, and its right child returns gain `8`. What is its complete candidate, and what gain should it return to its parent?**
