Day 45 DSA: **Build a Binary Tree from Preorder and Inorder Traversals**

Until now, you have mostly received a tree and explored it. Today, you’ll do the reverse:

> Given two traversal sequences, reconstruct the original binary tree.

This connects Day 31’s traversal orders with recursion, array boundaries, and hashmap lookup.

Imagine receiving two clues about a family tree:

- One list tells you which parent appears before its descendants.
- Another tells you which descendants belong on the left and right.

Together, those clues let you rebuild the structure.

We’ll assume **unique integer values**. This matters: repeated values can make reconstruction ambiguous.

Consider:

```python
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]
```

Expected tree:

```text
        3
       / \
      9   20
         / \
        15  7
```

**The 80/20 idea: one traversal chooses the root; the other splits the tree**

Remember the traversal orders:

```text
Preorder: Root → Left → Right
Inorder:  Left → Root → Right
```

Therefore:

1. The next unconsumed preorder value is the current subtree’s root.
2. Find that value in inorder.
3. Everything to its left belongs to the left subtree.
4. Everything to its right belongs to the right subtree.
5. Reconstruct the left subtree before the right subtree.

For our example:

```text
Preorder:
[3, 9, 20, 15, 7]
 ↑
Root = 3

Inorder:
[9]  3  [15, 20, 7]
left     right
```

Now repeat the same reasoning inside each part.

The left subtree contains only `9`.

For the right subtree:

```text
Next root = 20

Inorder section:
[15]  20  [7]
left       right
```

That gives `20` a left child `15` and a right child `7`.

**Why preorder alone is insufficient**

These trees have the same preorder:

```text
Tree A:       Tree B:

    1             1
   /               \
  2                 2
```

Both produce:

```python
[1, 2]
```

Their inorder sequences differ:

```text
Tree A: [2, 1]
Tree B: [1, 2]
```

For unique values, a valid preorder–inorder pair determines one tree. The lists must also be consistent with each other; merely containing the same values is not sufficient.

**Think about state before writing code**

We need two kinds of state:

| State | Meaning |
|---|---|
| `pre_index` | Position of the next unused root in preorder |
| `left`, `right` | Inclusive boundaries of the current subtree in inorder |
| `positions` | Maps each value to its inorder index |

Define the recursive function:

```text
build(left, right)

Construct the subtree represented by inorder[left:right + 1].
Consume exactly its nodes from preorder.
Return the constructed subtree root.
```

If:

```python
left > right
```

the section is empty, so return `None`.

Otherwise, consume the next preorder value, locate it in inorder, and split:

```text
Left subtree:  [left, middle - 1]
Right subtree: [middle + 1, right]
```

The middle position belongs to the current root. It must not appear in either child section.

**Why use a hashmap and boundaries?**

A straightforward implementation might repeatedly:

- Search inorder using `.index()`.
- Create smaller lists using slicing.

That is understandable, but repeated searches and copies can lead to `O(n²)` time on an unbalanced tree.

Instead:

```python
positions[value]  # Expected O(1) lookup
```

And pass integer boundaries rather than copying list sections.

This is the same optimization principle you have seen throughout DSA: **remember useful information and avoid repeated work**.

**Complete runnable Python**

Our function’s contract is:

- Accept two lists of unique integers.
- Return the reconstructed root.
- Return `None` for two empty lists.
- Raise `ValueError` for unequal lengths, duplicates, differing values, or inconsistent traversal order.
- Do not modify the input lists.

```python
class TreeNode:
    def __init__(self, value: int):
        self.value = value
        self.left: TreeNode | None = None
        self.right: TreeNode | None = None


def build_tree(
    preorder: list[int],
    inorder: list[int],
) -> TreeNode | None:
    n = len(preorder)

    if n != len(inorder):
        raise ValueError("Traversals must have equal lengths")

    if len(set(preorder)) != n or len(set(inorder)) != n:
        raise ValueError("Values must be unique")

    if set(preorder) != set(inorder):
        raise ValueError("Traversals must contain the same values")

    positions = {
        value: index
        for index, value in enumerate(inorder)
    }

    pre_index = 0

    def build(left: int, right: int) -> TreeNode | None:
        nonlocal pre_index

        if left > right:
            return None

        if pre_index >= n:
            raise ValueError("Inconsistent traversal order")

        value = preorder[pre_index]
        middle = positions[value]

        if not left <= middle <= right:
            raise ValueError("Inconsistent traversal order")

        pre_index += 1
        node = TreeNode(value)

        # Preorder consumes the entire left subtree first.
        node.left = build(left, middle - 1)
        node.right = build(middle + 1, right)

        return node

    root = build(0, n - 1)

    if pre_index != n:
        raise ValueError("Not all preorder values were consumed")

    return root


preorder = [3, 9, 20, 15, 7]
inorder = [9, 3, 15, 20, 7]

root = build_tree(preorder, inorder)

print(root.value)              # 3
print(root.left.value)         # 9
print(root.right.value)        # 20
print(root.right.left.value)   # 15
print(root.right.right.value)  # 7
```

`nonlocal pre_index` lets every recursive call advance the same cursor.

The cursor moves forward once per created node.

**Dry run**

Inorder indexes:

```text
Index:     0  1   2   3  4
Inorder:  [9, 3, 15, 20, 7]
```

The nonempty calls occur in this order:

| Inorder boundaries | Next preorder root | Root’s inorder index | Result |
|---|---:|---:|---|
| `[0, 4]` | 3 | 1 | Create root `3` |
| `[0, 0]` | 9 | 0 | Create `3.left` |
| `[2, 4]` | 20 | 3 | Create `3.right` |
| `[2, 2]` | 15 | 2 | Create `20.left` |
| `[4, 4]` | 7 | 4 | Create `20.right` |

Each leaf also creates two empty-section calls, which return `None` without consuming any preorder value.

At the end:

```python
pre_index == 5
```

All five values have been consumed.

**Why left must be built before right**

Preorder is:

```text
Root → entire left subtree → entire right subtree
```

With our single forward-moving cursor, the next value after the root belongs to the left subtree whenever that subtree exists.

For the example:

```text
After consuming 3, the next value is 9.
```

If we attempted to build the right subtree first, `9` would not belong within that subtree’s inorder boundaries.

The order is required by this implementation’s state contract.

**Why the algorithm works**

For each subtree, preorder identifies its root.

Because values are unique, the root has exactly one position in inorder. That position separates all nodes of the left subtree from all nodes of the right subtree.

Recursion applies the same reasoning to each section.

The empty section is the base case, and every nonempty call removes one root from its section before creating smaller child sections.

**A useful invalid-input example**

These lists contain the same unique values:

```python
preorder = [1, 2, 3]
inorder = [3, 1, 2]
```

But they are inconsistent.

From preorder, root is `1`. Inorder says:

```text
Left subtree:  [3]
Root:           1
Right subtree: [2]
```

The next preorder value is `2`, but the left subtree requires `3`.

Our boundary check catches that:

```python
if not left <= middle <= right:
    raise ValueError(...)
```

This demonstrates why checking equal lengths and matching sets is not enough.

**Complexity and common mistakes**

For `n` nodes and tree height `h`:

- **Expected time: `O(n)`**, assuming average constant-time set and dictionary operations.
- **Auxiliary space: `O(n)`** for validation sets, the index map, and up to `O(h)` recursion.
- **Output space: `O(n)`** for the newly created tree.
- A deep chain may exceed Python’s recursion limit.

| Mistake | Why it fails |
|---|---|
| Treat the first inorder value as the root | Inorder begins with the left subtree |
| Build right before left with the shared cursor | Consumes preorder values in the wrong order |
| Include `middle` in a child interval | Reuses the current root |
| Advance the cursor for an empty section | Skips values that belong to real nodes |
| Assume matching sets guarantee consistency | Traversal order may still contradict subtree boundaries |
| Use a value-to-index map with duplicates | One value may have several positions |
| Compare values to choose left or right | This is an ordinary binary tree, not necessarily a BST |

**Verify by traversing the reconstructed tree**

The strongest practical check is to regenerate both input traversals.

```python
def preorder_values(root):
    result = []

    def visit(node):
        if node is None:
            return
        result.append(node.value)
        visit(node.left)
        visit(node.right)

    visit(root)
    return result


def inorder_values(root):
    result = []

    def visit(node):
        if node is None:
            return
        visit(node.left)
        result.append(node.value)
        visit(node.right)

    visit(root)
    return result


assert preorder_values(root) == preorder
assert inorder_values(root) == inorder

assert build_tree([], []) is None

single = build_tree([7], [7])
assert single.value == 7
assert single.left is None
assert single.right is None

chain = build_tree([1, 2, 3], [3, 2, 1])
assert chain.left.left.value == 3

try:
    build_tree([1, 2, 3], [3, 1, 2])
except ValueError:
    pass
else:
    raise AssertionError("Expected inconsistent-order error")
```

Use the valid example lists when running the round-trip assertions; the invalid lists above are a separate example.

**Your assignment**

Reconstruct a tree from:

```python
preorder = [10, 5, 3, 7, 15, 12, 18]
inorder = [3, 5, 7, 10, 12, 15, 18]
```

Expected structure:

```text
          10
         /  \
        5    15
       / \   / \
      3   7 12 18
```

Then test an example that is **not a BST**:

```python
preorder = [1, 9, 2]
inorder = [9, 1, 2]
```

Expected:

```text
      1
     / \
    9   2
```

For your submission:

1. Implement reconstruction independently.
2. Trace the cursor and inorder boundaries for the first example.
3. Verify both traversals of the constructed tree.
4. Test empty lists, one node, a chain, and inconsistent input.
5. Explain why unique values matter.

As an extension, consider rebuilding from **inorder and postorder**. Postorder ends with the root. If you consume it backward using a shared cursor, which child subtree must you build first?

At **institute level**, explain how the two traversals contribute different information. A correct dry run should show both the cursor and the active inorder interval.

At **corporate level**, validate inputs before returning a result and use stable unique IDs when values can repeat. For very deep structures, consider an iterative implementation rather than relying on recursion.

At **client level**, clarify whether supplied labels are unique identifiers or display names. Two employees can share a name; reconstructing relationships from names alone may be ambiguous. Also confirm that the provided sequences really follow the agreed traversal conventions.

For GitHub:

```text
Day-45-DSA-Build-Tree-from-Preorder-and-Inorder.md
```

Before coding, answer this:

```python
preorder = [8, 4, 2, 6, 12]
inorder = [2, 4, 6, 8, 12]
```

**What is the root, and which inorder values belong to its left and right subtrees?**
