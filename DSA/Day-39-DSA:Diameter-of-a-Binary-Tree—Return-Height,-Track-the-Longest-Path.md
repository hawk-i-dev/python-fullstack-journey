Day 39 DSA: **Diameter of a Binary Tree — Return Height, Track the Longest Path**

Today, you’ll find the longest path between any two nodes in a binary tree.

This builds on two familiar ideas:

- Day 30: calculate tree height.
- Day 38: combine information from the left and right subtrees.

Imagine a tree-shaped network of rooms connected by corridors. You want to find the longest route between two rooms without revisiting a room.

The **diameter** is the number of edges on that longest path.

We’ll assume a valid binary tree: no cycles or shared child nodes. Values can repeat because this problem depends on connections, not values.

```text
        1
       / \
      2   3
     / \
    4   5
```

One longest path is:

```text
4 → 2 → 1 → 3
```

It contains four nodes but **three edges**:

```text
4 — 2 — 1 — 3
    3 connections
```

Expected result:

```python
diameter(root)  # 3
```

An equally long path is `5 → 2 → 1 → 3`. We need the length, so multiple longest paths are fine.

**The 80/20 idea: every node can be a meeting point**

At each node, ask:

> “How far can I go down its left side, and how far can I go down its right side?”

The longest path passing through that node combines those two downward branches.

```text
Deepest left descendant
          ↑
      current node
          ↓
Deepest right descendant
```

We need two different results:

| Information | Formula | Purpose |
|---|---|---|
| Height returned to the parent | `1 + max(left_height, right_height)` | Extend one downward branch |
| Path length through this node | `left_height + right_height` | Candidate for the diameter |

This distinction is the heart of today’s lesson:

> **Return one branch upward; combine both branches when measuring a complete path.**

A parent cannot extend both branches below a child without creating a fork. A simple path can pass through a node, but it cannot split into three directions.

**Choose a consistent height definition**

We’ll measure subtree height in **nodes**:

```text
Empty subtree height = 0
Leaf height = 1
```

For a current node, its left subtree’s height in nodes equals the number of edges from the current node to the deepest node on that side.

Example:

```text
      1
     /
    2
   /
  4
```

The subtree rooted at `2` has height `2` nodes: `2 → 4`.

From node `1` to node `4`, there are also `2` edges: `1 → 2 → 4`.

That is why this formula gives the path length in edges:

```python
path_through_current = left_height + right_height
```

Do not add `1` here. That would count nodes on the path instead.

**Think through the first example**

```text
        1
       / \
      2   3
     / \
    4   5
```

Start at the leaves because a parent needs its children’s heights.

- Nodes `4`, `5`, and `3` each return height `1`.
- Node `2` receives heights `1` and `1`.
- Its path candidate is `1 + 1 = 2` edges: `4 → 2 → 5`.
- It returns height `1 + max(1, 1) = 2`.
- Node `1` receives heights `2` and `1`.
- Its path candidate is `2 + 1 = 3` edges.
- The largest candidate is `3`.

This is **postorder DFS**: process the left child, process the right child, then combine their results.

**The longest path does not have to pass through the root**

Consider:

```text
          1
         /
        2
       / \
      3   4
     /     \
    5       6
```

The longest path is:

```text
5 → 3 → 2 → 4 → 6
```

Its length is `4` edges, and it never visits root `1`.

At root `1`, the path candidate is only `3` edges.

Therefore, checking only the root’s left and right heights is insufficient. We must consider the path through **every node**.

**Turn the reasoning into code**

The helper function has one clear job:

```text
height(node) returns the subtree height in nodes.
```

While calculating those heights, it also updates the best diameter seen so far.

Our public function will:

- Return the diameter in edges.
- Return `0` for an empty tree or a single node.
- Leave the tree unchanged.

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


def diameter(root: TreeNode | None) -> int:
    best = 0

    def height(node: TreeNode | None) -> int:
        nonlocal best

        if node is None:
            return 0

        left_height = height(node.left)
        right_height = height(node.right)

        # A complete path may use both branches.
        through_current = left_height + right_height
        best = max(best, through_current)

        # The parent can extend only one branch.
        return 1 + max(left_height, right_height)

    height(root)
    return best


root = TreeNode(
    1,
    TreeNode(2, TreeNode(4), TreeNode(5)),
    TreeNode(3),
)

print(diameter(root))  # 3
```

`nonlocal best` tells Python that the helper updates the `best` variable belonging to the surrounding `diameter()` call.

Because `best = 0` is created inside the public function, each call starts fresh. Results from an earlier tree do not leak into a later call.

**Dry run**

For the example above, the nonempty nodes are processed in this order:

```text
4 → 5 → 2 → 3 → 1
```

| Node | Left height | Right height | Path through node | Height returned | Best so far |
|---|---:|---:|---:|---:|---:|
| `4` | 0 | 0 | 0 | 1 | 0 |
| `5` | 0 | 0 | 0 | 1 | 0 |
| `2` | 1 | 1 | 2 | 2 | 2 |
| `3` | 0 | 0 | 0 | 1 | 2 |
| `1` | 2 | 1 | 3 | 3 | 3 |

At node `3`, the local candidate is `0`, but `best` remains `2`. This is why we use:

```python
best = max(best, through_current)
```

rather than simply replacing `best` with the latest candidate.

**Why the algorithm works**

Every simple path in a tree has a highest node: the point where its two downward portions meet. This is the LCA of its endpoints.

For a fixed meeting point, the longest such path uses the deepest available branch on each side. Its length is:

```python
left_height + right_height
```

We evaluate that candidate at every node. Since the actual longest path has a meeting point somewhere in the tree, we will consider it.

Meanwhile, each height is calculated correctly from its children:

```python
1 + max(left_height, right_height)
```

So both the local candidates and the final maximum are correct.

**Complexity and common mistakes**

Let `n` be the node count and `h` the tree height.

- **Time: `O(n)`** — each node is processed once.
- **Auxiliary space: `O(h)`** — recursive call stack.
- Balanced tree: `O(log n)` stack space.
- Chain: `O(n)` stack space and possible Python recursion-depth failure.

A slower approach calculates left and right heights from scratch at every node. On a chain, that repeats work and can take `O(n²)` time. Our helper calculates each subtree height once and passes it upward.

| Mistake | What goes wrong |
|---|---|
| Return `left_height + right_height` as height | A parent receives a branching path instead of a downward height |
| Add `1` to the diameter candidate | Counts path nodes instead of edges |
| Check only the root | Misses a longer path entirely inside a subtree |
| Replace `best` without taking `max` | Can erase a better result |
| Keep `best` as persistent global state | Separate calls may affect each other |
| Use BST comparisons | Values do not determine path lengths |

**Tests to run**

```python
assert diameter(None) == 0
assert diameter(TreeNode(10)) == 0

# One edge.
assert diameter(
    TreeNode(1, right=TreeNode(2))
) == 1

# Four nodes in a chain: three edges.
chain = TreeNode(
    1,
    right=TreeNode(
        2,
        right=TreeNode(3, right=TreeNode(4)),
    ),
)
assert diameter(chain) == 3

# Longest path does not pass through the root.
inside_subtree = TreeNode(
    1,
    left=TreeNode(
        2,
        TreeNode(3, left=TreeNode(5)),
        TreeNode(4, right=TreeNode(6)),
    ),
)
assert diameter(inside_subtree) == 4

# Repeated values do not affect the answer.
duplicates = TreeNode(5, TreeNode(5), TreeNode(5))
assert diameter(duplicates) == 2

# A new call must start with a fresh best value.
assert diameter(inside_subtree) == 4
assert diameter(TreeNode(99)) == 0
```

**Your assignment: measure the longest route**

Build this binary tree directly:

```text
          10
         /  \
        4    20
       / \     \
      2   6     30
         /        \
        5          40
```

Implement `diameter(root)` independently.

Expected result:

```python
6
```

A longest path is:

```text
5 → 6 → 4 → 10 → 20 → 30 → 40
```

Seven nodes, six edges.

For your submission:

1. Write the tree construction and diameter function.
2. Record each node’s left height, right height, and diameter candidate.
3. Test empty, single-node, chain, and subtree-only longest paths.
4. Explain why the helper returns `1 + max(...)` while the diameter uses a sum.

As an extension, return both endpoints of a longest path. You can have the height helper also report the deepest descendant on its chosen branch. When a new best candidate appears, record its two endpoints. For the assignment tree, those endpoints are nodes `5` and `40`.

At **institute level**, distinguish height from diameter and edges from nodes. Your dry run should show both the value returned to the parent and the best answer maintained separately.

At **corporate level**, define whether you need only a length, the endpoints, or the entire path. Keep per-call state isolated, and use an iterative postorder approach when tree depth can exceed recursion limits.

At **client level**, clarify what a connection represents. This algorithm measures the number of edges in an unweighted tree. If each edge has a travel time or distance, the calculation must account for weights. If the network contains cycles, it is a graph and this tree algorithm does not apply unchanged.

Before coding, explain this aloud:

> “My parent needs one extendable branch. My diameter candidate can join two branches.”
