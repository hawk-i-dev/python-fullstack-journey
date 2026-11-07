Day 47 DSA: **Flatten a Binary Tree to a Linked List — Preserve Preorder, Rewire Carefully**

Today, you’ll transform a binary tree into a single chain using its existing nodes.

The chain must follow **preorder traversal**:

```text
Root → Left → Right
```

Every node’s:

- `left` pointer becomes `None`.
- `right` pointer points to the next node in preorder.

This combines Day 31’s traversal order with Day 14’s pointer-rewiring discipline.

Imagine a branching railway network. You want to rearrange the existing tracks into one route, visiting stations in preorder. Before changing a track, you must preserve access to stations you have not visited yet.

Consider:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

Preorder:

```text
1 → 2 → 3 → 4 → 5 → 6
```

After flattening:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

These are the **same node objects**, connected differently. We are not creating a separate list of copied values.

**Define the contract first**

Our function will:

- Modify the input tree.
- Preserve every existing node exactly once.
- Preserve each node’s value.
- Produce preorder order along `right` pointers.
- Set every `left` pointer to `None`.
- Return `None`, because the transformation happens through the existing root reference.

An empty tree requires no work.

We assume valid tree structure: no cycles or shared child nodes.

**The 80/20 idea: put the original right subtree after the left subtree**

At a node with a left child:

```text
Before:

        current
        /     \
      LEFT   RIGHT
```

Preorder requires:

```text
current → all of LEFT → all of RIGHT
```

Therefore, we must:

1. Preserve the original right subtree.
2. Move the left subtree to the right.
3. Arrange for the original right subtree to come after it.

The key challenge is choosing a safe place to attach that original right subtree.

**First understand the stack approach**

A straightforward solution uses a stack to remember nodes in preorder.

```text
Pop current.
Save its original right child.
Save its original left child.
Connect the previous node to current.
```

Push right before left because a stack is last-in, first-out. That makes the left child get processed first.

This gives `O(n)` time and up to `O(h)` stack space for a valid binary tree.

Today, we’ll use a pointer-rewiring approach that needs **O(1) auxiliary space**.

**Find a safe attachment point**

For the current node, start at its left child and keep following right pointers:

```python
tail = current.left

while tail.right is not None:
    tail = tail.right
```

This finds the end of the left subtree’s current right-pointer chain.

Attach the original right subtree there:

```python
tail.right = current.right
```

Then move the left subtree:

```python
current.right = current.left
current.left = None
```

Finally, advance:

```python
current = current.right
```

An important detail: `tail` is not necessarily a leaf or the final preorder node of that subtree. It may have a left child. Later iterations will move those left descendants into place before the attached continuation.

**Work through the example**

Start with:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

At node `1`, its left subtree starts at `2`.

Follow right pointers:

```text
2 → 4
```

Node `4` has no right child, so attach the original right subtree rooted at `5` there.

Then move `2` to become `1.right` and clear `1.left`:

```text
1
 \
  2
 / \
3   4
     \
      5
       \
        6
```

The tree is not fully flattened yet. Node `2` still has a left child.

At node `2`:

- Its left subtree is node `3`.
- Node `3` has no right child.
- Attach the current right subtree rooted at `4` to `3.right`.
- Move `3` to `2.right`.
- Clear `2.left`.

Result:

```text
1 → 2 → 3 → 4 → 5 → 6
```

The remaining nodes have no left children, so we simply move forward.

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


def flatten(root: TreeNode | None) -> None:
    current = root

    while current is not None:
        if current.left is not None:
            # Find the end of the left subtree's right chain.
            tail = current.left

            while tail.right is not None:
                tail = tail.right

            # Preserve the original right subtree.
            tail.right = current.right

            # Move the left subtree to the right.
            current.right = current.left
            current.left = None

        current = current.right


root = TreeNode(
    1,
    TreeNode(2, TreeNode(3), TreeNode(4)),
    TreeNode(5, right=TreeNode(6)),
)

flatten(root)

values = []
current = root

while current is not None:
    assert current.left is None
    values.append(current.value)
    current = current.right

print(values)
# [1, 2, 3, 4, 5, 6]
```

Call it like this:

```python
flatten(root)
```

Do not write:

```python
root = flatten(root)
```

Our function returns `None`. Assigning that return value to `root` would lose your root reference.

This differs from your BST insertion and deletion functions, which explicitly returned the updated root.

**Why the order of assignments matters**

This is the safe sequence:

```python
tail.right = current.right
current.right = current.left
current.left = None
```

The original right subtree is connected somewhere safe before its previous connection is overwritten.

If you move the left subtree first without saving the old right subtree, you can lose access to nodes.

That is the same principle as linked-list reversal:

> Preserve the next structure you still need before changing its connection.

**What stays true during the loop?**

Before processing each `current` node:

- Nodes already passed form the correct preorder prefix.
- Their left pointers are `None`.
- All unprocessed nodes remain reachable through the structure beginning at `current`.

When a left subtree exists, the rewiring places it before the original right subtree, which is exactly the order preorder requires.

Repeatedly applying this operation eventually removes every left pointer.

**A subtle example: the attachment node can have a left child**

```text
        1
       / \
      2   5
       \
        3
       /
      4
```

At node `1`, following the left subtree’s right chain reaches `3`.

Node `3` still has left child `4`, but it is safe to attach `5` to `3.right`.

Later, when processing `3`, the algorithm moves `4` before `5`.

Final order:

```text
1 → 2 → 3 → 4 → 5
```

This is why “follow right pointers until there is no right child” is more precise than “find a leaf.”

**Complexity**

Let `n` be the node count.

- **Time: O(n).**
- **Auxiliary space: O(1).**
- No new tree nodes are created.

The nested loop does not imply `O(n²)`. Across the whole run, the right-chain searches traverse disjoint portions of the original tree’s right links; their total work is linear. The outer loop also visits each node once.

This is an amortized argument: count work across the entire execution rather than multiplying loop lengths mechanically.

**Common mistakes**

| Mistake | Consequence |
|---|---|
| Overwrite `current.right` before preserving it | Loses the original right subtree |
| Forget `current.left = None` | Output is not the required chain |
| Create new nodes from the values | Does not preserve node identity |
| Use inorder instead of preorder | Produces the wrong order |
| Assume the attachment point must be a leaf | Adds an unnecessary and potentially incorrect search |
| Write `root = flatten(root)` | Root becomes `None` under this contract |
| Check only output values | May miss copied, lost, or repeated node objects |

**Test structure and node identity**

Capture the expected node order before mutation:

```python
def preorder_nodes(root):
    if root is None:
        return []

    result = []
    stack = [root]

    while stack:
        node = stack.pop()
        result.append(node)

        if node.right is not None:
            stack.append(node.right)

        if node.left is not None:
            stack.append(node.left)

    return result
```

Then verify the transformed chain:

```python
def assert_flattened(root, expected_nodes):
    current = root

    for expected in expected_nodes:
        assert current is expected
        assert current.left is None
        current = current.right

    assert current is None


tree = TreeNode(
    1,
    TreeNode(2, TreeNode(3), TreeNode(4)),
    TreeNode(5, right=TreeNode(6)),
)

expected = preorder_nodes(tree)
flatten(tree)
assert_flattened(tree, expected)

# Empty tree.
flatten(None)

# Single node.
single = TreeNode(7)
flatten(single)
assert single.left is None
assert single.right is None

# Flattening an already flattened tree changes nothing.
flatten(tree)
assert_flattened(tree, expected)
```

The test helpers use additional memory, but that does not change the flattening algorithm’s own O(1) auxiliary-space bound.

**Your assignment**

Build:

```text
          10
         /  \
        5    15
       / \     \
      3   7     20
         /
        6
```

Expected chain:

```text
10 → 5 → 3 → 7 → 6 → 15 → 20
```

Complete these tasks:

1. Implement `flatten()` independently.
2. Record the attachment point for each node that has a left child.
3. Draw the structure after processing nodes `10`, `5`, and `7`.
4. Verify that every left pointer is `None`.
5. Verify that all node objects appear exactly once in the original preorder order.

Also test a left-only chain, an already flattened right-only chain, and repeated values.

As an extension, implement the stack version and compare its output node order with the O(1)-space version using separate copies of the same input tree.

At **institute level**, explain the assignment order and demonstrate why the old right subtree is not lost. Be ready to justify linear total work despite the inner loop.

At **corporate level**, treat this as a destructive structural transformation: callers sharing these node objects will observe the changed connections. If the original hierarchy is still needed, agree on creating a separate representation instead.

At **client level**, clarify whether “flatten” means changing the hierarchy or merely displaying it as a list. A flattened report usually should preserve the underlying business relationships. Also agree on preorder versus another ordering.

For GitHub:

```text
Day-47-DSA-Flatten-Binary-Tree-to-Linked-List.md
```

Before coding, answer this:

> At node `1`, the left subtree starts at `2` and the right subtree starts at `5`. Why must we preserve the connection to `5` before assigning `current.right = current.left`?
