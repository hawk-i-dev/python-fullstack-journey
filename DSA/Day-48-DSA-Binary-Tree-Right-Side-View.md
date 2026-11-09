Day 48 DSA: **Binary Tree Right Side View — The Last Node at Each Level**

Today, you’ll find the values visible when looking at a binary tree from its right side.

This revisits Day 32’s **level-order traversal with a queue**, with one extra decision:

> At each level, keep only the rightmost node.

Imagine students standing in rows. Looking from the right, you see the student at the right end of each row—even when that student belongs to the left side of the overall group.

Consider:

```text
        1
       / \
      2   3
       \   \
        5   4
```

The right-side view is:

```python
[1, 3, 4]
```

We assume valid binary-tree structure. Values may repeat, and BST ordering is unnecessary.

**The important distinction: visible does not mean “follow right children”**

Consider:

```text
        1
       / \
      2   3
       \
        5
         \
          6
```

The right-side view is:

```python
[1, 3, 5, 6]
```

At the first two levels, nodes `1` and `3` are visible. Below node `3`, there are no nodes on that side, so `5` and `6` become visible.

Simply following:

```python
current = current.right
```

would miss them.

Our precise definition is:

> Return the rightmost existing node at every depth, from top to bottom.

**The 80/20 idea**

Use left-to-right BFS:

1. Put the root in a queue.
2. Save the number of nodes in the current level.
3. Process exactly that many nodes.
4. Add each node’s left child before its right child.
5. Save the value of the **last node processed at that level**.

Because nodes are processed left to right, the last one is the rightmost.

This is almost the same as Day 32:

| Day 32: Level order | Day 48: Right-side view |
|---|---|
| Save every value in each level | Save only the last value |
| Output nested lists | Output one flat list |
| Same queue and level-size logic | Same queue and level-size logic |

**Define the state before coding**

| Variable | Meaning |
|---|---|
| `queue` | Nodes waiting to be processed |
| `level_size` | Number of nodes in the current level |
| `index` | Current position within that level |
| `view` | One rightmost value per completed level |

At the start of each outer loop:

> The queue contains exactly the nodes of the next level, in left-to-right order.

During processing, children join the queue. Freezing `level_size` prevents them from being treated as part of their parents’ level.

**Complete runnable Python**

```python
from collections import deque


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


def right_side_view(root: TreeNode | None) -> list[int]:
    if root is None:
        return []

    view = []
    queue = deque([root])

    while queue:
        level_size = len(queue)

        for index in range(level_size):
            node = queue.popleft()

            # Last node in this left-to-right level.
            if index == level_size - 1:
                view.append(node.value)

            if node.left is not None:
                queue.append(node.left)

            if node.right is not None:
                queue.append(node.right)

    return view


root = TreeNode(
    1,
    TreeNode(2, right=TreeNode(5)),
    TreeNode(3, right=TreeNode(4)),
)

print(right_side_view(root))
# [1, 3, 4]
```

This function does not modify the tree. It returns a new list:

```python
visible_values = right_side_view(root)
```

That differs from yesterday’s `flatten(root)`, which changed the tree and returned `None`. Always check the function’s contract before deciding whether to capture its return value.

**Dry run**

For the main example:

```text
        1
       / \
      2   3
       \   \
        5   4
```

| Level | Queue at level start | Processed order | Value saved |
|---|---|---|---:|
| 1 | `[1]` | `1` | 1 |
| 2 | `[2, 3]` | `2 → 3` | 3 |
| 3 | `[5, 4]` | `5 → 4` | 4 |

Result:

```python
[1, 3, 4]
```

Look carefully at level 2:

```text
Start:                  [2, 3]
Process 2; add child 5:  [3, 5]
Process 3; add child 4:  [5, 4]
```

We stop after processing two nodes because:

```python
level_size == 2
```

The newly queued `5` and `4` belong to the next level.

**Why the algorithm works**

The root starts the first level.

If the current level is processed left to right, adding each parent’s left child before its right child places the next level into the queue in left-to-right order as well.

Therefore, the final node processed at each level is exactly the node required by the right-side-view definition.

Every node is processed once, and one value is saved per level.

**A second approach: right-first DFS**

You can also solve this using recursion.

Visit:

```text
Current → Right → Left
```

The first node you encounter at a new depth is the rightmost node at that depth.

```python
def right_side_view_dfs(root: TreeNode | None) -> list[int]:
    view = []

    def visit(node: TreeNode | None, depth: int) -> None:
        if node is None:
            return

        # No value has been saved for this depth yet.
        if depth == len(view):
            view.append(node.value)

        visit(node.right, depth + 1)
        visit(node.left, depth + 1)

    visit(root, 0)
    return view
```

Why does this condition work?

```python
depth == len(view)
```

Initially, the list is empty. The first node at depth `0` adds the first value. The first node at depth `1` adds the second value, and so on.

Nodes encountered later at the same depth do not replace the saved value.

The traversal order and selection rule must agree:

| Traversal | Selection rule | Result |
|---|---|---|
| Left-to-right BFS | Last node per level | Right view |
| Right-to-left BFS | First node per level | Right view |
| Right-first DFS | First node per depth | Right view |
| Left-first DFS | First node per depth | Left view |

Do not reverse child order without checking whether the selection rule also needs to change.

**Complexity**

Let `n` be the node count, `h` the height, and `w` the maximum number of nodes at any level.

| Approach | Time | Auxiliary space, excluding output |
|---|---:|---:|
| BFS | `O(n)` | `O(w)` |
| Recursive DFS | `O(n)` | `O(h)` |

Both return `O(h)` values for a nonempty tree.

The BFS queue may temporarily contain parts of two adjacent levels; its size remains `O(w)`.

A wide tree can require a large BFS queue. A deep chain can require a large recursive stack and may exceed Python’s recursion limit.

**Common mistakes**

| Mistake | Why it fails |
|---|---|
| Follow only right-child pointers | Misses deeper visible nodes from left subtrees |
| Save every leaf | Visibility is determined by depth and horizontal order |
| Save the first node in left-to-right BFS | Produces the left view |
| Recompute the level boundary while adding children | Mixes different levels |
| Select the maximum value per level | Rightmost is a structural position, not a numerical maximum |
| Remove duplicate values from the result | Different levels may have the same visible value |
| Forget the empty-tree case | May try to access attributes of `None` |

For example:

```text
       1
      / \
    100  2
```

The right-side view is:

```python
[1, 2]
```

It is not `[1, 100]`.

**Tests to run**

```python
assert right_side_view(None) == []
assert right_side_view(TreeNode(7)) == [7]

assert right_side_view(root) == [1, 3, 4]

# Deeper nodes from the left subtree remain visible.
left_extension = TreeNode(
    1,
    TreeNode(
        2,
        right=TreeNode(5, right=TreeNode(6)),
    ),
    TreeNode(3),
)
assert right_side_view(left_extension) == [1, 3, 5, 6]

# A left-only chain is fully visible.
left_chain = TreeNode(
    1,
    left=TreeNode(2, left=TreeNode(3)),
)
assert right_side_view(left_chain) == [1, 2, 3]

# Structural position matters, not numerical size.
different_values = TreeNode(1, TreeNode(100), TreeNode(2))
assert right_side_view(different_values) == [1, 2]

# Equal values on different levels are preserved.
duplicates = TreeNode(5, TreeNode(5), TreeNode(5))
assert right_side_view(duplicates) == [5, 5]

# Independent approaches should agree.
assert right_side_view(root) == right_side_view_dfs(root)
assert right_side_view(left_extension) == right_side_view_dfs(left_extension)
```

**Your assignment**

Build:

```text
          10
         /  \
        5    15
       / \   /
      3   7 12
           \
            8
```

The nodes by level are:

```text
Level 1: [10]
Level 2: [5, 15]
Level 3: [3, 7, 12]
Level 4: [8]
```

Expected right-side view:

```python
[10, 15, 12, 8]
```

Node `8` is the right child of `7`. It remains visible because no other node exists at that depth.

Complete these tasks:

1. Implement the BFS solution independently.
2. Record the queue at the beginning of each level.
3. Explain why node `8` appears despite belonging to the root’s left subtree.
4. Implement the right-first DFS solution and compare outputs.
5. Verify that the original tree’s structure remains unchanged.

As an extension, implement `left_side_view()`.

For the same tree:

```python
[10, 5, 3, 8]
```

With left-to-right BFS, save the first node at each level instead of the last.

At **institute level**, explain the queue invariant and why “rightmost” differs from “largest” or “right child.” Dry-run an example where the left subtree extends deeper than the right.

At **corporate level**, choose BFS or DFS based on likely width and depth. Preserve the original structure if this is a reporting operation, and define whether the output contains values, unique IDs, or complete records.

At **client level**, clarify what “right side” means. A UI can place nodes differently, while this algorithm uses the tree’s left/right child order. If the requirement is “latest item at each level” or “highest-priority item,” that is a different selection rule.

For GitHub:

```text
Day-48-DSA-Binary-Tree-Right-Side-View.md
```

Before coding, answer this:

```text
      1
     / \
    2   3
   /
  4
```

**What is the right-side view, and why does node `4` appear?**
