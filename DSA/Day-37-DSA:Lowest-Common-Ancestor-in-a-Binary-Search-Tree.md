Day 37 DSA: **Lowest Common Ancestor in a Binary Search Tree**

Today, you’ll find the deepest node that connects two target nodes.

This builds on your BST search skills: comparing values tells us whether to move left, move right, or stop.

Imagine two people walking toward their homes from the same entrance. They share part of the route, then separate. Their **last shared junction** is the idea behind the lowest common ancestor, or **LCA**.

“Lowest” means **deepest in the tree**, not the smallest numerical value.

We’ll use a valid strict BST with unique integer values:

```text
           6
         /   \
        2     8
       / \   / \
      0   4 7   9
         / \
        3   5
```

Consider nodes `3` and `5`:

```text
Path to 3: 6 → 2 → 4 → 3
Path to 5: 6 → 2 → 4 → 5
```

Their shared ancestors are `6`, `2`, and `4`. The deepest one is `4`.

```python
LCA(3, 5)  # node containing 4
```

A node is allowed to be its own ancestor for this problem:

```python
LCA(2, 5)  # node containing 2
```

The path to `5` passes through `2`, so `2` is the deepest node that contains both targets in its subtree.

**The 80/20 idea: find where the paths separate**

At each node, make one of three decisions:

| Target values compared with current value | Action |
|---|---|
| Both smaller | Move left |
| Both larger | Move right |
| On opposite sides, or one equals current | Current node is the LCA |

For targets `3` and `5`:

```text
At 6: both smaller → left
At 2: both larger  → right
At 4: 3 < 4 < 5   → stop
```

Why stop at `4`? One target is in its left subtree and the other is in its right subtree. No node below `4` can be an ancestor of both.

For targets `2` and `5`:

```text
At 6: both smaller → left
At 2: one target equals current → stop
```

There is no need to build or store both complete paths.

**Decide the function’s contract before coding**

Some interview problems guarantee that both targets exist. A practical function should make that assumption explicit or check it.

Our function will:

- Accept two integer target values, `a` and `b`.
- Return the LCA **node object** when both values exist.
- Return `None` when either value is missing.
- Allow `a == b`: return that node if it exists.
- Leave the tree unchanged.

Checking existence matters. For example, consider targets `3` and `10`. At root `6`, the values appear to split:

```text
3 < 6 < 10
```

But `10` is absent. Returning `6` would violate our contract.

We’ll reuse BST search to verify both targets, then find their LCA.

**Turn the reasoning into code**

First, arrange the targets into a smaller and larger value:

```python
low = min(a, b)
high = max(a, b)
```

Then the three decisions become:

```text
current.value > high → both targets are smaller → left
current.value < low  → both targets are larger  → right
otherwise           → current lies between them → answer
```

The final condition includes equality:

```text
low <= current.value <= high
```

Here is a complete runnable example:

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


def search_bst(
    root: TreeNode | None,
    target: int,
) -> TreeNode | None:
    current = root

    while current is not None:
        if target == current.value:
            return current

        if target < current.value:
            current = current.left
        else:
            current = current.right

    return None


def lowest_common_ancestor(
    root: TreeNode | None,
    a: int,
    b: int,
) -> TreeNode | None:
    # Our contract requires both target values to exist.
    if search_bst(root, a) is None:
        return None

    if search_bst(root, b) is None:
        return None

    low = min(a, b)
    high = max(a, b)
    current = root

    while current is not None:
        if current.value > high:
            current = current.left

        elif current.value < low:
            current = current.right

        else:
            return current

    return None


root = TreeNode(
    6,
    TreeNode(
        2,
        TreeNode(0),
        TreeNode(4, TreeNode(3), TreeNode(5)),
    ),
    TreeNode(8, TreeNode(7), TreeNode(9)),
)

queries = [
    (3, 5),
    (2, 5),
    (0, 9),
    (7, 9),
    (4, 4),
    (3, 10),
]

for a, b in queries:
    answer = lowest_common_ancestor(root, a, b)
    print((a, b), "->", answer.value if answer else None)
```

Expected output:

```text
(3, 5) -> 4
(2, 5) -> 2
(0, 9) -> 6
(7, 9) -> 8
(4, 4) -> 4
(3, 10) -> None
```

The two searches validate the inputs. The final traversal locates the deepest shared ancestor.

**Dry run: targets `3` and `5`**

Both targets are found during validation. Then:

```python
low = 3
high = 5
```

| Current node | Check | Decision |
|---|---|---|
| `6` | `6 > 5` | Both targets are left; move to `2` |
| `2` | `2 < 3` | Both targets are right; move to `4` |
| `4` | `3 <= 4 <= 5` | Return node `4` |

Notice that `4` is neither target. An LCA can be a third node, one of the targets, or the root.

**Why this finds the lowest ancestor**

If both targets are left of the current node, the left child’s subtree contains both. Therefore, the current node cannot be the deepest shared ancestor—we can go deeper.

The same reasoning applies when both targets are right.

When the targets split, neither child’s subtree contains both. The current node is the deepest shared ancestor.

If the current node equals one target, no proper descendant of that node can be an ancestor of it. The current node is therefore the answer.

**Complexity and mistakes**

Let `h` be tree height and `n` the number of nodes.

- Each existence check takes at most `O(h)`.
- Finding the LCA takes at most `O(h)`.
- Total time remains **`O(h)`**: three traversals change the constant, not the complexity class.
- Auxiliary space is **`O(1)`** because all traversals are iterative.

A balanced tree gives `O(log n)` time. A chain can require `O(n)`.

| Mistake | Why it fails |
|---|---|
| Interpret “lowest” as smallest value | LCA refers to depth |
| Compare only one target | You may move away from the other |
| Forget equality | A target itself can be the LCA |
| Return a split point without an existence guarantee | A target might be missing |
| Apply value comparisons to an arbitrary binary tree | Only BST ordering justifies choosing a branch |
| Always return the root | A deeper shared ancestor may exist |
| Assume the function must return one of the targets | The answer may be another node |

There is also no requirement that `a < b`. Using `min()` and `max()` makes the target order irrelevant.

**Test the contract**

```python
assert lowest_common_ancestor(root, 3, 5) is root.left.right
assert lowest_common_ancestor(root, 5, 3) is root.left.right

# One target is the other's ancestor.
assert lowest_common_ancestor(root, 2, 5) is root.left

# Targets are in opposite root subtrees.
assert lowest_common_ancestor(root, 0, 9) is root

# Same existing target.
assert lowest_common_ancestor(root, 4, 4) is root.left.right

# Missing target and empty tree.
assert lowest_common_ancestor(root, 3, 10) is None
assert lowest_common_ancestor(root, 10, 10) is None
assert lowest_common_ancestor(None, 3, 5) is None
```

Using `is` checks that the function returns the actual existing node object, rather than creating a new node with the same value.

**Your assignment: find shared ancestors**

Build this BST using your insertion function:

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

Implement `lowest_common_ancestor(root, a, b)` independently.

Expected results:

| Targets | LCA value |
|---|---:|
| `13, 17` | `15` |
| `5, 17` | `10` |
| `25, 35` | `30` |
| `13, 35` | `20` |
| `10, 17` | `10` |
| `17, 17` | `17` |
| `13, 99` | `None` |

For each valid query, write the nodes inspected during the **LCA traversal**, separately from the existence checks.

Example:

```text
Targets: 13 and 17
LCA traversal: 20 → 10 → 15
Answer: 15
```

As an extension, return the path from the root to the LCA:

```python
path_to_lca(root, 13, 17)
# [20, 10, 15]

path_to_lca(root, 13, 99)
# None
```

Keep the same missing-target contract.

At **institute level**, explain the three branching decisions and demonstrate an ancestor-target case. Include a missing-target test so your assumptions are visible.

At **corporate level**, choose whether callers guarantee target existence or the function verifies it. If the tree can change between validation and traversal, coordinate reads and updates so the query sees a consistent tree. Otherwise, a target could disappear after validation.

At **client level**, clarify what kind of hierarchy is being searched. “Find the closest common manager” resembles an LCA problem, but an organization chart usually is not a BST ordered by employee ID. Its solution must follow reporting relationships rather than comparing IDs. Also clarify whether a person may count as their own ancestor and what should happen when a requested person is absent.

Before coding, explain this in your own words:

> “If both targets are on the same side, I can go deeper. If they split—or I reach one target—I have found their lowest common ancestor.”
