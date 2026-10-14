Day 38 DSA: **Lowest Common Ancestor in a Binary Tree**

Yesterday, BST ordering told you which direction to search. Today, the values have **no ordering rule**, so we must find the targets by exploring the tree.

This makes the problem useful for understanding hierarchies: the relationships matter, not whether one ID is numerically smaller than another.

Imagine two children in different classrooms. To find their closest shared supervisor, follow the hierarchy until their paths meet. That meeting point is their **lowest common ancestor**, or LCA.

“Lowest” means deepest. A node can be its own ancestor.

Use this binary tree:

```text
           3
         /   \
        5     1
       / \   / \
      6   2 0   8
         / \
        7   4
```

This is not a BST: `5` is on the left of `3`, even though it is larger.

Expected answers:

```text
LCA(6, 4) = 5
LCA(7, 4) = 2
LCA(5, 4) = 5
LCA(6, 8) = 3
```

**The 80/20 idea: let each subtree report what it found**

At each node, ask:

> “Did my left subtree find a target? Did my right subtree find a target?”

Then combine their answers:

| Left result | Right result | Return |
|---|---|---|
| Nothing | Nothing | `None` |
| A node | Nothing | Left result |
| Nothing | A node | Right result |
| A node | A node | Current node |

There is one earlier rule: if the current node **is one of the targets**, return it.

This is a recursive DFS approach. For non-target nodes, we process both children before combining their results—a postorder-style pattern.

**Think through an example before coding**

Find the LCA of nodes `6` and `4`.

```text
           3
          /
         5
        / \
       6   2
            \
             4
```

- Node `6` finds itself and returns its node reference.
- Node `4` finds itself and returns its node reference.
- Node `2` receives a result only from its right side, so it passes that result upward.
- Node `5` receives a result from **both sides**, so it returns itself.
- Node `3` receives node `5` from the left and nothing from the right, so it passes node `5` upward.

Answer: node `5`.

Notice that the root does not replace the answer with itself. When only one child returns a result, we pass that result upward.

**Decide what identifies a node**

In an ordinary binary tree, different nodes may contain the same value:

```text
       9
      / \
     5   5
```

“Find node `5`” is ambiguous.

Today, the function accepts **actual node objects**, named `p` and `q`. We compare identity:

```python
node is p
```

That means “this exact node,” not merely “a node with the same value.”

Our contract is:

- Return the existing LCA node if both target objects belong to the tree.
- Return `None` if either target is absent.
- If both arguments refer to the same existing node, return that node.
- Do not modify the tree.
- Assume valid tree structure: no cycles or shared child nodes.

**Handle missing targets explicitly**

The compact recursive LCA algorithm normally assumes both targets exist.

Without that guarantee, it can return the one target it finds—even when the other is missing.

For clarity, we’ll first check that both node objects belong to the tree, then run the LCA search. These extra traversals still give `O(n)` total time.

Here is a complete runnable implementation:

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


def contains_node(
    root: TreeNode | None,
    target: TreeNode,
) -> bool:
    if root is None:
        return False

    if root is target:
        return True

    return (
        contains_node(root.left, target)
        or contains_node(root.right, target)
    )


def lowest_common_ancestor(
    root: TreeNode | None,
    p: TreeNode,
    q: TreeNode,
) -> TreeNode | None:
    if not contains_node(root, p):
        return None

    if not contains_node(root, q):
        return None

    def find(node: TreeNode | None) -> TreeNode | None:
        if node is None:
            return None

        if node is p or node is q:
            return node

        left_result = find(node.left)
        right_result = find(node.right)

        if left_result is not None and right_result is not None:
            return node

        if left_result is not None:
            return left_result

        return right_result

    return find(root)


# Create separate objects so we can reference exact nodes.
n6 = TreeNode(6)
n7 = TreeNode(7)
n4 = TreeNode(4)
n2 = TreeNode(2, n7, n4)
n5 = TreeNode(5, n6, n2)

n0 = TreeNode(0)
n8 = TreeNode(8)
n1 = TreeNode(1, n0, n8)

root = TreeNode(3, n5, n1)

queries = [
    (n6, n4),
    (n7, n4),
    (n5, n4),
    (n6, n8),
]

for p, q in queries:
    answer = lowest_common_ancestor(root, p, q)
    print((p.value, q.value), "->",
          answer.value if answer is not None else None)
```

Expected output:

```text
(6, 4) -> 5
(7, 4) -> 2
(5, 4) -> 5
(6, 8) -> 3
```

**Understand what the recursive return means**

A non-`None` result can represent:

- A target found in that subtree.
- The LCA already discovered deeper in that subtree.

You do not need separate flags for those cases under the existence guarantee.

If both targets were already found under one child, the other child cannot contain either target. The deeper LCA simply passes upward unchanged.

If results arrive from both children, the targets are separated across those children. Their lowest shared ancestor is the current node.

Now consider an ancestor-target pair:

```text
LCA(5, 4)
```

The helper reaches node `5` and returns it immediately. That is correct because `5` is itself a target and is an ancestor of `4`. The preliminary checks have already confirmed that both targets exist.

**Dry run: targets `7` and `4`**

The following table shows the key return values:

| Current node | Left result | Right result | What it returns |
|---|---|---|---|
| `7` | Not explored | Not explored | Itself: target found |
| `4` | Not explored | Not explored | Itself: target found |
| `2` | Node `7` | Node `4` | Node `2`: targets split |
| `5` | `None` from node `6`’s subtree | Node `2` | Node `2` |
| `3` | Node `2` | `None` | Node `2` |

The answer remains `2` as recursion returns through `5` and `3`.

**Complexity and common mistakes**

Let `n` be the number of nodes and `h` the tree height.

- **Time: `O(n)`**. Each membership check and the LCA traversal visits at most `n` nodes.
- **Auxiliary space: `O(h)`** for recursion.
- A balanced tree uses `O(log n)` stack space.
- A chain may use `O(n)` stack space and exceed Python’s recursion limit.

The membership checks run sequentially. Three linear traversals remain `O(n)`, not `O(n³)`.

| Mistake | Why it fails |
|---|---|
| Choose left or right using target values | This tree has no BST ordering |
| Compare only `node.value` | Equal values can belong to different nodes |
| Return current when only one side reports a result | Can replace a deeper LCA with a higher ancestor |
| Ignore missing-target behavior | May return the one target that exists |
| Forget that a target can be an ancestor | Misses valid answers such as `LCA(5, 4) = 5` |
| Assume every hierarchy is a binary tree | Real hierarchies may have many children or multiple parents |

**Test identity, not just displayed values**

```python
assert lowest_common_ancestor(root, n6, n4) is n5
assert lowest_common_ancestor(root, n7, n4) is n2
assert lowest_common_ancestor(root, n5, n4) is n5
assert lowest_common_ancestor(root, n6, n8) is root

# Same existing object.
assert lowest_common_ancestor(root, n4, n4) is n4

# Same value does not mean the same node.
outside = TreeNode(4)
assert lowest_common_ancestor(root, n6, outside) is None

# Empty tree.
assert lowest_common_ancestor(None, n6, n4) is None

# Two distinct nodes may share a value.
left = TreeNode(5)
right = TreeNode(5)
duplicate_tree = TreeNode(9, left, right)

assert lowest_common_ancestor(
    duplicate_tree, left, right
) is duplicate_tree
```

**Your assignment: find shared ancestors without BST ordering**

Build this tree and keep references to each node:

```text
           10
          /  \
        20    5
       / \     \
      8  30     2
         / \
        7   9
```

Implement the function independently.

Expected answers, shown by value for readability:

| Target nodes | LCA value |
|---|---:|
| `8, 9` | `20` |
| `7, 9` | `30` |
| `20, 9` | `20` |
| `8, 2` | `10` |
| `5, 2` | `5` |
| Same node `7` twice | `7` |
| Node `8` and a new outside node | `None` |

Submit your code, tests, and a return-value dry run for targets `8` and `9`.

As an extension, implement an iterative solution using a parent map:

1. Traverse the tree and record each node’s parent.
2. Verify that both targets were found.
3. Store all ancestors of `p`, including `p`.
4. Walk upward from `q`; return the first node in that set.

This avoids recursive calls but uses `O(n)` extra space.

At **institute level**, explain why yesterday’s BST shortcut is unavailable. Demonstrate the difference between “both sides found something” and “only one side found something.”

At **corporate level**, use stable unique identifiers when converting external records into node objects. Define missing-record behavior, validate the hierarchy structure, and choose an iterative approach for potentially deep data.

At **client level**, a “closest common manager” feature needs clear rules: may an employee count as their own ancestor, can employees belong to separate organizations, and can someone report to multiple managers? Multiple parents create a graph, where a unique lowest common ancestor is not always guaranteed.

Before writing the assignment, explain this aloud:

> “Both sides report a target: return me. Only one side reports a result: pass that result upward.”
