Day 34 DSA: **Search and Insert in a Binary Search Tree**

Yesterday, you checked whether a tree obeys BST rules. Today, you’ll use those rules to **find a value and insert a new value in the correct place**.

We’ll use a strict BST: values in the left subtree are smaller, values in the right subtree are greater, and duplicates are not allowed.

Imagine finding a numbered classroom. At every junction, a sign tells you:

- Smaller numbers → left.
- Larger numbers → right.
- Matching number → you’ve arrived.

You choose one direction at each step because the BST’s ordering tells you where the answer could be.

Use this tree:

```text
           8
         /   \
        3     10
       / \      \
      1   6      14
         / \     /
        4   7   13
```

**The 80/20 rule for today**

Both operations use the same three comparisons:

```text
target < current.value  → go left
target > current.value  → go right
target == current.value → found
```

Their difference is what happens when you reach a missing child:

```text
Search: missing child means the value is absent.
Insert: missing child is where the new node belongs.
```

This approach assumes the input is already a valid BST. If its ordering is broken, choosing just one branch can miss a value.

**Think through search before coding**

Suppose you want to find `7`.

```text
At 8: 7 < 8 → left
At 3: 7 > 3 → right
At 6: 7 > 6 → right
At 7: match → return this node
```

Visited values:

```text
8 → 3 → 6 → 7
```

We never need to inspect `10`, `14`, or `13`.

Now search for `5`:

```text
At 8: 5 < 8 → left
At 3: 5 > 3 → right
At 6: 5 < 6 → left
At 4: 5 > 4 → right
Missing child → return None
```

The reasoning is precise: if `5` existed, the BST rules would place it along this path. Reaching an empty position proves it is absent.

We’ll return the **node object** when found and `None` otherwise. Returning a node lets a caller access its value or other fields later.

```python
class TreeNode:
    def __init__(self, value: int):
        self.value = value
        self.left: TreeNode | None = None
        self.right: TreeNode | None = None


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
```

Only one variable changes: `current`, the node we are inspecting. Search does not modify the tree.

**Turn the same thinking into insertion**

Now insert `5` into the original tree.

Follow the search path:

```text
8 → 3 → 6 → 4
```

At node `4`, the target `5` is larger. Its right child is missing, so attach the new node there:

```text
Before:             After:

    6                   6
   / \                 / \
  4   7               4   7
                       \
                        5
```

Why is this position valid?

Along the path, `5` obeyed these restrictions:

```text
5 < 8
5 > 3
5 < 6
5 > 4
```

These are the ancestor constraints you learned yesterday. Following comparisons preserves them automatically.

Before implementing insertion, define its behavior:

- Empty tree: create and return the root.
- New value: attach a new leaf and return the original root.
- Duplicate value: leave the tree unchanged and return the original root.

Other duplicate policies are possible, but we’ll use this one consistently.

```python
def insert_bst(
    root: TreeNode | None,
    value: int,
) -> TreeNode:
    if root is None:
        return TreeNode(value)

    current = root

    while True:
        if value == current.value:
            # Duplicate: no change.
            return root

        if value < current.value:
            if current.left is None:
                current.left = TreeNode(value)
                return root

            current = current.left

        else:
            if current.right is None:
                current.right = TreeNode(value)
                return root

            current = current.right
```

The `while True` loop stops when it finds a duplicate or attaches a new node. For a finite valid tree, following children eventually reaches one of those conditions.

Notice the difference between these statements:

```python
current = current.right
```

This moves our local reference to another node.

```python
current.right = TreeNode(value)
```

This changes the tree by connecting a new child.

Also, insertion returns `root`, not `current`. The caller needs the entry point to the entire tree.

**Run both operations together**

Place this below the class and functions:

```python
def inorder(root: TreeNode | None) -> list[int]:
    result = []

    def visit(node: TreeNode | None) -> None:
        if node is None:
            return

        visit(node.left)
        result.append(node.value)
        visit(node.right)

    visit(root)
    return result


root = None

for value in [8, 3, 10, 1, 6, 14, 4, 7, 13]:
    root = insert_bst(root, value)

found = search_bst(root, 7)
print(found.value if found is not None else "Not found")
# 7

print(search_bst(root, 5))
# None

root = insert_bst(root, 5)

print(inorder(root))
# [1, 3, 4, 5, 6, 7, 8, 10, 13, 14]

root = insert_bst(root, 6)  # Duplicate: unchanged.

print(inorder(root))
# [1, 3, 4, 5, 6, 7, 8, 10, 13, 14]
```

Always capture the returned root:

```python
root = insert_bst(root, value)
```

This matters especially when `root` starts as `None`. The function creates a node, but your variable receives it only when you assign the return value.

The inorder helper uses recursion for demonstration. The search and insertion functions themselves are iterative.

**Complexity: height controls the work**

Let `h` be tree height and `n` the number of nodes.

| Operation | Time | Auxiliary space |
|---|---:|---:|
| Iterative search | `O(h)` | `O(1)` |
| Iterative insertion | `O(h)` | `O(1)` |

Insertion also creates one new node when the value is absent.

For a balanced tree, `h = O(log n)`. For a chain, `h = O(n)`.

A BST is **not automatically balanced**. Insert these values in order:

```python
[1, 2, 3, 4, 5]
```

You get:

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
```

Finding `5` requires visiting every node. Building a tree from `n` sorted values with this insertion method takes `O(n²)` total time because successive insertions follow increasingly long paths.

Balanced search trees address this problem; today’s implementation does not rebalance itself.

**Common mistakes**

| Mistake | Consequence | Correction |
|---|---|---|
| Search both children every time | Loses the benefit of BST ordering | Choose one branch using comparison |
| Assume every BST operation is `O(log n)` | Misses the chain-shaped worst case | State the cost as `O(h)` |
| Ignore the returned root | An initially empty tree stays unassigned | Use `root = insert_bst(...)` |
| Overwrite an existing child during insertion | Disconnects an existing subtree | Attach only where the child is `None` |
| Forget the equality case | Duplicate behavior becomes incorrect or unclear | Handle duplicates explicitly |
| Change a stored value arbitrarily | Can break BST ordering | Use an operation that preserves ordering |

For example, changing the root from `8` to `100` would leave smaller values in its right subtree, violating the BST rule.

**Your assignment: build a number lookup tool**

Starting from an empty tree, insert:

```python
[20, 10, 30, 5, 15, 25, 35]
```

The result should be:

```text
         20
        /  \
       10   30
      / \   / \
     5  15 25 35
```

Complete these tasks independently:

1. Search for `15`. Return the node containing `15`.
2. Search for `17`. Return `None`.
3. Insert `17`. It should become the right child of `15`.
4. Insert `10` again. The tree should remain unchanged.
5. Return the inorder values.

Expected final inorder output:

```python
[5, 10, 15, 17, 20, 25, 30, 35]
```

Add a function:

```python
def search_path(root, target) -> list[int]:
    ...
```

It should return the values inspected during the search, whether the target is found or absent.

After inserting `17`:

```python
search_path(root, 17)
# [20, 10, 15, 17]

search_path(root, 26)
# [20, 30, 25]

search_path(None, 26)
# []
```

Hint: append the current value before checking equality or choosing the next branch.

Test an empty tree, a single node, a missing target, a duplicate insertion, and negative values. Reuse yesterday’s `is_valid_bst()` to confirm that your insertions preserve the strict BST rule.

At **institute level**, submit the functions, expected outputs, and a dry run of inserting `17`. Explain why the node belongs below `15` and why search returns `None` for an absent value.

At **corporate level**, define the duplicate policy and return values clearly. Consider tree height and expected data volume before selecting this structure. If multiple threads can modify the same tree, a search-then-attach sequence needs coordination to avoid conflicting updates.

At **client level**, clarify what “insert an existing key” should mean: ignore it, reject it, or update its associated record. Those are different requirements. Also confirm whether ordered operations are needed; a plain BST is not automatically the best choice for every lookup feature.

Suppose the client changes the request to:

> “If the product ID already exists, update its price.”

The product ID becomes the ordering key, while price is separate data stored in the node. On an equal ID, update the price without changing the key or creating another node. Changing the ordering key in place could invalidate the tree.

Before writing your assignment, explain these two points in your own words:

> “Search stops at a missing child because the target has no other valid location.”

> “Insertion uses that missing child position to connect a new leaf while preserving ancestor rules.”
