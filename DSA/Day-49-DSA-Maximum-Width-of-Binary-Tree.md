Day 49 DSA: **Maximum Width of a Binary Tree — Count the Gaps with Position Indexes**

Yesterday, you selected the rightmost node at each level. Today, you’ll measure the distance from the leftmost node to the rightmost node at each level.

There is one important difference from simply counting nodes:

> **Width includes missing positions between the first and last existing nodes.**

Imagine a row of numbered seats. Someone sits in seat `1`, and someone else sits in seat `4`. Only two people are present, but their occupied span covers four seats.

Consider:

```text
          1
         / \
        3   2
       / \   \
      5   3   9
```

At the third level, the possible positions are:

```text
Position:  0   1   2   3
Node:      5   3   ·   9
```

The width is:

```text
last position - first position + 1
= 3 - 0 + 1
= 4
```

Expected answer:

```python
maximum_width(root)  # 4
```

We assume a valid binary tree. Node values may repeat because positions—not values—determine width.

**Clarify what “width” means**

Earlier, you practiced counting actual nodes in a level. That is a different measurement.

| Measurement | What it counts |
|---|---|
| Level node count | Existing nodes only |
| Today’s positional width | Existing nodes and gaps between the endpoints |

For:

```text
5   3   ·   9
```

The node count is `3`, while the positional width is `4`.

We do not count empty positions outside the first and last existing nodes.

**The 80/20 idea: give each node a position**

Use the same indexing scheme as an array representation of a complete binary tree:

```text
Root position: 0

For a node at position p:
Left child:  2*p + 1
Right child: 2*p + 2
```

For example:

```text
                0
              /   \
             1     2
            / \   / \
           3   4 5   6
```

These numbers are positions, not node values.

Even if the node at position `5` is missing, the node at position `6` keeps its position. That preserves the gap.

At each level:

```python
width = last_position - first_position + 1
```

Use BFS to process levels, just as you did on Days 32 and 48.

**Do we need to enqueue missing nodes?**

No.

Store only existing nodes, together with their positions:

```python
(node, position)
```

The position numbers already represent the gaps.

For the main example, the third level contains:

```text
(node 5, position 3)
(node 3, position 4)
(node 9, position 6)
```

The missing position `5` contributes to the width without requiring a `None` entry in the queue.

**Keep position numbers smaller by normalizing each level**

Absolute indexes can grow rapidly in deep trees.

For a right-only chain, they would be:

```text
0 → 2 → 6 → 14 → 30 → ...
```

But every level has width `1`.

Before processing a level, subtract its first position from all positions in that level:

```python
position = raw_position - base
```

Example:

```text
Before: [100, 101, 103]
After:  [0,   1,   3]
```

The width is unchanged:

```text
103 - 100 + 1 = 4
3 - 0 + 1     = 4
```

We can generate child positions from these normalized positions. This shifts all next-level positions by the same amount, preserving their relative distances.

Normalization removes unnecessary common offsets. It does not eliminate genuinely large gaps: a very sparse tree can still have a huge positional width.

**Define the state before coding**

| Variable | Meaning |
|---|---|
| `queue` | Existing nodes and their position indexes |
| `level_size` | Number of existing nodes in the current level |
| `base` | First position in the current level |
| `last_position` | Position of the last processed node after normalization |
| `best` | Largest width found so far |

At the start of each level, the queue is in left-to-right order.

Because the first normalized position is `0`:

```python
level_width = last_position + 1
```

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


def maximum_width(root: TreeNode | None) -> int:
    if root is None:
        return 0

    queue = deque([(root, 0)])
    best = 0

    while queue:
        level_size = len(queue)
        base = queue[0][1]
        last_position = 0

        for _ in range(level_size):
            node, raw_position = queue.popleft()
            position = raw_position - base
            last_position = position

            if node.left is not None:
                queue.append((node.left, 2 * position + 1))

            if node.right is not None:
                queue.append((node.right, 2 * position + 2))

        best = max(best, last_position + 1)

    return best


root = TreeNode(
    1,
    TreeNode(3, TreeNode(5), TreeNode(3)),
    TreeNode(2, right=TreeNode(9)),
)

print(maximum_width(root))  # 4
```

The function does not modify the tree. It returns an integer:

```python
width = maximum_width(root)
```

An empty tree returns `0`; a single-node tree returns `1`.

**Dry run**

For the main example:

| Level | Positions in queue | Base | Normalized positions | Width |
|---|---|---:|---|---:|
| Root | `[0]` | 0 | `[0]` | 1 |
| Second | `[1, 2]` | 1 | `[0, 1]` | 2 |
| Third | `[1, 2, 4]` | 1 | `[0, 1, 3]` | 4 |

Why does the third row contain `[1, 2, 4]` rather than absolute positions `[3, 4, 6]`?

The previous level was normalized to `[0, 1]` before generating its children:

```text
Parent at 0:
left child  = 1
right child = 2

Parent at 1:
right child = 4
```

The shift changes absolute numbers but preserves the gap and width.

Final answer:

```python
4
```

**Why queue length is not enough**

Consider:

```text
        1
       / \
      2   3
     /     \
    4       5
```

The last level has:

```text
Node positions: 4   ·   ·   5
```

There are two existing nodes, but the positional width is four.

```python
len(queue)             # 2 at the start of that level
last - first + 1        # 4
```

This is why the queue needs position information.

**Why the algorithm works**

The child-index formulas reserve two positions for every parent. Missing nodes leave gaps instead of causing later nodes to shift left.

BFS visits a level in left-to-right order, so its first and last indexes identify the endpoints.

Subtracting the same base from every position preserves their difference:

```text
(last - base) - (first - base)
= last - first
```

Therefore, normalization preserves width, and taking the largest level width gives the answer.

**Complexity**

Let `n` be the node count and `w` the maximum number of **existing nodes** at a level.

Under the usual constant-time index-arithmetic model:

- **Time: O(n)** — each node enters and leaves the queue once.
- **Auxiliary space: O(w)** — only existing nodes are queued.

Here, `w` is actual node count, not positional width. A level can have two nodes separated by many empty positions.

Python supports large integers, but arithmetic and storage for very large indexes have costs proportional to their size. Normalization helps avoid unnecessary growth; it cannot make an enormous true width small.

**Common mistakes**

| Mistake | Why it fails |
|---|---|
| Use the number of queued nodes as width | Ignores internal gaps |
| Use node values as positions | Values do not describe placement |
| Forget `+1` | A single-node level would have width zero |
| Enqueue every missing position | Can waste enormous memory |
| Subtract a different base for each node | Destroys relative distances |
| Mix incompatible indexing formulas | Produces incorrect positions |
| Assume normalization removes all large indexes | Genuine gaps can still be large |

Use one fixed base for the entire level.

**Tests to run**

```python
assert maximum_width(None) == 0
assert maximum_width(TreeNode(7)) == 1
assert maximum_width(root) == 4

# Two existing nodes separated by two gaps.
sparse = TreeNode(
    1,
    TreeNode(2, left=TreeNode(4)),
    TreeNode(3, right=TreeNode(5)),
)
assert maximum_width(sparse) == 4

# A chain has one existing position at every level.
chain = TreeNode(
    1,
    right=TreeNode(2, right=TreeNode(3)),
)
assert maximum_width(chain) == 1

# Values do not affect positional width.
duplicates = TreeNode(
    5,
    TreeNode(5, left=TreeNode(5)),
    TreeNode(5, right=TreeNode(5)),
)
assert maximum_width(duplicates) == 4
```

**Your assignment**

Build:

```text
          10
         /  \
        5    15
       /       \
      3         20
       \       /
        4     18
```

Using complete-tree positions, the levels have widths:

```text
Level 1: 1
Level 2: 2
Level 3: 4
Level 4: 6
```

At level 4, the two nodes occupy absolute indexes `8` and `13`:

```text
Width = 13 - 8 + 1 = 6
```

Expected result:

```python
maximum_width(root)  # 6
```

Complete these tasks:

1. Implement the function independently.
2. Record each level’s raw indexes, base, normalized indexes, and width.
3. Explain why the last level’s node count is `2`, but its width is `6`.
4. Confirm that values and connections remain unchanged.
5. Test an empty tree, a chain, and a sparse tree.

As an extension, return both the maximum width and its level number. Define the root as level `1`, and keep the earliest level if several levels tie.

For this assignment:

```python
(6, 4)
```

At **institute level**, explain why indexes capture gaps and why subtracting a shared base preserves width. Keep actual node count separate from positional width.

At **corporate level**, consider sparse inputs and numeric limits in the implementation language. Avoid materializing missing nodes, and specify tie behavior if you return level information.

At **client level**, clarify whether “width” means actual items, complete-tree positions, or rendered pixels. This algorithm measures complete-binary-tree positions; it does not calculate a diagram’s visual pixel width.

For GitHub:

```text
Day-49-DSA-Maximum-Width-of-Binary-Tree.md
```

Before coding, answer this:

> A level has existing nodes at positions `12`, `13`, and `19`. What are its node count, positional width, and normalized positions?
