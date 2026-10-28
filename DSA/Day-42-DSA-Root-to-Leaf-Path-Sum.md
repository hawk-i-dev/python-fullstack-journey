Day 42 DSA: **Path Sum — Does a Root-to-Leaf Path Match the Target?**

Yesterday, you found the maximum sum of a path that could start and end anywhere. Today, the question is different:

> **Does any path from the root to a leaf add up exactly to the target?**

This lesson teaches an important habit: understand the path’s rules before choosing the algorithm.

Imagine walking through connected rooms while spending a budget. Each room changes the amount remaining. You succeed only when you reach an exit—a leaf—with exactly zero remaining.

We assume a valid binary tree with integer values. Negative values and duplicates are allowed.

Consider:

```text
           5
          / \
         4   8
        /   / \
       11  13  4
      / \      \
     7   2      1
```

For target `22`, the answer is `True`:

```text
5 → 4 → 11 → 2

5 + 4 + 11 + 2 = 22
```

This path starts at the root and ends at a leaf.

For target `9`, the answer is `False`. Although:

```text
5 + 4 = 9
```

node `4` is not a leaf. We cannot stop there.

**First, distinguish today’s problem from Day 41**

| Rule | Maximum Path Sum | Today’s Path Sum |
|---|---|---|
| Starting point | Any node | Root |
| Ending point | Any node | Leaf |
| Objective | Largest sum | Exact target |
| Output | Integer | Boolean |
| Can stop before a leaf? | Yes | No |
| Can discard negative branches using `max(0, gain)`? | Yes, for optional contributions | No |

A small change in wording changes the algorithm.

**The 80/20 idea: carry the remaining target downward**

At each node, subtract its value:

```python
remaining = target - node.value
```

Then ask a child:

> “Can you complete a path to a leaf using this remaining amount?”

For the successful path:

```text
Target starts at 22

Visit 5:  22 - 5  = 17
Visit 4:  17 - 4  = 13
Visit 11: 13 - 11 = 2
Visit 2:   2 - 2  = 0
```

We reached a leaf with zero remaining, so the answer is `True`.

Remember three rules:

1. A missing node returns `False`.
2. At a leaf, check whether the remaining target is exactly zero after subtraction.
3. At an internal node, either child can provide a successful path.

That last rule means we combine results using **`or`**.

**Recognize a leaf correctly**

A leaf has **no left child and no right child**:

```python
node.left is None and node.right is None
```

This node is not a leaf:

```text
    4
   /
  9
```

Having one missing child does not make a node a leaf.

Also, a missing child is not an exit you can use to complete a path. The endpoint must be an actual leaf node.

**Define the recursive function before coding**

Our function’s meaning is:

```text
has_path_sum(node, target)

Does a path starting at this node and ending at a leaf
have a sum equal to target?
```

The public behavior is:

- Return `True` if at least one valid path matches.
- Return `False` otherwise.
- An empty tree returns `False`, including when the target is zero.
- Do not modify the tree.

The recursive call makes progress by moving from a node to one of its children.

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


def has_path_sum(root: TreeNode | None, target: int) -> bool:
    if root is None:
        return False

    remaining = target - root.value

    is_leaf = root.left is None and root.right is None

    if is_leaf:
        return remaining == 0

    return (
        has_path_sum(root.left, remaining)
        or has_path_sum(root.right, remaining)
    )


root = TreeNode(
    5,
    TreeNode(
        4,
        left=TreeNode(11, TreeNode(7), TreeNode(2)),
    ),
    TreeNode(
        8,
        TreeNode(13),
        TreeNode(4, right=TreeNode(1)),
    ),
)

print(has_path_sum(root, 22))  # True
print(has_path_sum(root, 26))  # True: 5 → 8 → 13
print(has_path_sum(root, 9))   # False: cannot stop at internal 4
print(has_path_sum(root, 18))  # True: 5 → 8 → 4 → 1
```

Each child receives the same remaining amount from its parent. The branches represent separate possible paths.

Python’s `or` short-circuits: if the left subtree returns `True`, the right subtree does not need to be checked.

**Dry run for target `22`**

The implementation searches the left side first.

| Node | Target received | Remaining after subtraction | Decision |
|---|---:|---:|---|
| `5` | 22 | 17 | Explore children |
| `4` | 17 | 13 | Explore children |
| `11` | 13 | 2 | Explore children |
| `7` | 2 | -5 | Leaf: return `False` |
| `2` | 2 | 0 | Leaf: return `True` |

The `True` result travels upward through `11`, `4`, and `5`.

Notice that node `2` receives target `2`, not `-5`. The unsuccessful route through node `7` does not alter its sibling’s remaining amount.

We do not need an explicit “undo subtraction” step here: integers are immutable, and each function call has its own local variables.

**Negative values change what you can safely skip**

Consider:

```text
      5
     /
   -3
```

Target:

```python
2
```

The correct path is:

```text
5 → -3
Sum = 2
```

At node `5`, the remaining amount becomes:

```text
2 - 5 = -3
```

That negative remainder does not mean failure. The next node can bring it back to zero:

```text
-3 - (-3) = 0
```

Therefore, this shortcut is incorrect when negative values are allowed:

```python
if remaining < 0:
    return False
```

You can only introduce pruning when the input constraints justify it.

**Zero remaining is not enough by itself**

Consider:

```text
      5
     /
    0
```

With target `5`, the remainder becomes zero at the root. But the root is not a leaf, so we continue.

The path `5 → 0` ends at a leaf and still sums to `5`, so the final answer is `True`.

Now change the child:

```text
      5
     /
    2
```

With target `5`, the answer becomes `False`. The only root-to-leaf path sums to `7`.

The rule is:

> Zero remaining is success only at a leaf, after processing that leaf’s value.

**Why the algorithm works**

Every root-to-leaf path includes the current node’s value.

After subtracting that value, a successful path must complete the remaining sum through either the left child or the right child.

At a leaf, there are no more nodes to choose, so the sum must match exactly.

A missing subtree contains no valid path.

These cases cover every possibility.

**Complexity and common mistakes**

Let `n` be the number of nodes and `h` the tree height.

- **Worst-case time: `O(n)`** — we may inspect every node.
- **Auxiliary space: `O(h)`** — recursive call stack.
- A successful early path may let us stop sooner.
- A deep chain may exceed Python’s recursion limit.

| Mistake | Why it fails |
|---|---|
| Return `True` whenever the remainder reaches zero | The current node may not be a leaf |
| Treat one missing child as a leaf | A leaf requires both children to be missing |
| Return `target == 0` for a missing node | Can accept a path ending through a missing child |
| Use `and` between child results | Only one matching path is required |
| Reject every negative remainder | Negative node values can restore the target |
| Use Day 41’s `max(0, gain)` logic | Today’s path must start at the root and reach a leaf |

**Tests to run**

```python
assert has_path_sum(None, 0) is False

assert has_path_sum(TreeNode(7), 7) is True
assert has_path_sum(TreeNode(7), 0) is False

assert has_path_sum(root, 22) is True
assert has_path_sum(root, 26) is True
assert has_path_sum(root, 9) is False

# Negative remainder can still lead to success.
negative_path = TreeNode(5, left=TreeNode(-3))
assert has_path_sum(negative_path, 2) is True

# Matching an internal node is insufficient.
internal_match = TreeNode(5, left=TreeNode(2))
assert has_path_sum(internal_match, 5) is False

# Zero-valued nodes are allowed.
zero_path = TreeNode(5, left=TreeNode(0))
assert has_path_sum(zero_path, 5) is True

# A single zero node is a valid nonempty path.
assert has_path_sum(TreeNode(0), 0) is True
```

**Your assignment: find a matching route**

Build this tree:

```text
           10
          /  \
         5   -3
        / \    \
       3   2    11
      /     \
     -2      1
```

Its root-to-leaf paths are:

```text
10 → 5 → 3 → -2 = 16
10 → 5 → 2 → 1  = 18
10 → -3 → 11    = 18
```

Implement `has_path_sum()` independently.

Expected results:

| Target | Result |
|---:|---|
| 16 | `True` |
| 18 | `True` |
| 15 | `False` |
| 7 | `False` |
| 100 | `False` |

Targets `15` and `7` match internal prefixes, but neither ends at a leaf.

For your submission:

1. Write the tree construction and function.
2. Trace the remaining target for `16`.
3. Explain why target `7` returns `False`.
4. Include empty-tree, single-node, negative-value, and internal-match tests.

As an extension, implement an iterative DFS using a stack of:

```python
(node, remaining_target)
```

Make the meaning precise: let `remaining_target` be the amount needed **before processing that node**. Pop, subtract the node’s value, check for a leaf, then push children with the updated remainder.

At **institute level**, explain the function contract and why the leaf check is necessary. Compare this problem with Day 41 without mixing their path rules.

At **corporate level**, choose recursion or iteration based on possible tree depth. Keep per-path state separate, and confirm whether the caller needs a boolean, one matching path, all paths, or a count—those outputs require different handling.

At **client level**, clarify what counts as a completed route. If completion means reaching a designated terminal state, that may not be the same as a structural leaf. For monetary amounts, define an exact representation such as integer minor units rather than relying on approximate floating-point equality.

For GitHub:

```text
Day-42-DSA-Root-to-Leaf-Path-Sum.md
```

Before coding, answer this:

```text
    4
   /
 -4
```

**For target `0`, should the result be `True` or `False`? At which node is success confirmed?**
