Day 35 DSA: **Delete a Node from a Binary Search Tree**

Yesterday, you learned to search and insert. Today, you’ll remove a value **while keeping the remaining tree a valid BST**.

We’ll continue using integer values and a strict BST: no duplicates.

Imagine removing a person from a family photograph arranged by number. Removing someone at the edge is easy. Removing someone who connects two groups requires choosing a replacement that keeps everyone correctly ordered.

BST deletion has exactly **three cases**:

| Node being deleted | What to do |
|---|---|
| No children | Remove it |
| One child | Connect its parent directly to its child |
| Two children | Replace its value with its inorder successor, then delete that successor from the right subtree |

The difficult part is preserving connections. The useful question is:

> “After deletion, what node should become the root of this subtree?”

That question guides both the algorithm and the return values.

**Start with the easy cases**

Consider:

```text
          8
         / \
        3   10
       / \    \
      1   6    14
```

Delete `1`, a leaf:

```text
          8
         / \
        3   10
         \    \
          6    14
```

The subtree rooted at `1` becomes empty, so deletion returns `None` to its parent.

Now consider deleting `10` from the original tree. It has one child, `14`:

```text
Before:             After:

    8                   8
     \                   \
     10                  14
       \
       14
```

The subtree previously rooted at `10` should now be rooted at `14`. Return that child so the parent can reconnect to it.

This gives two compact rules:

```python
if root.left is None:
    return root.right

if root.right is None:
    return root.left
```

These also cover a leaf: if both children are missing, the first condition returns `None`.

**The two-child case**

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

Suppose we delete `3`.

We cannot simply replace `3` with either child and discard the other subtree. Both sides contain values we must preserve.

Instead, choose the **inorder successor**: the smallest value in the node’s right subtree.

For node `3`:

```text
Right subtree:

      6
     / \
    4   7

Smallest value = 4
```

Find it by moving right once, then moving left as far as possible:

```text
3 → right to 6 → left to 4
```

Why is `4` a suitable replacement?

- It is greater than every value in `3`’s left subtree.
- It is the smallest value in `3`’s right subtree.
- After removing its original occurrence, all remaining right-subtree values are greater than it.

Deletion has two steps:

```text
1. Replace 3's value with 4.
2. Delete the original 4 from the right subtree.
```

Result:

```text
           8
         /   \
        4     10
       / \      \
      1   6      14
           \     /
            7   13
```

The successor has **no left child**, because it is already the leftmost node. It may have a right child, so removing it reduces to one of our easy cases.

The largest value in the left subtree—the inorder predecessor—is also a valid replacement strategy. Today, use the successor consistently.

**The 80/20 rules**

Remember these four decisions:

1. Smaller target → delete from the left subtree.
2. Larger target → delete from the right subtree.
3. Found, with zero or one child → return the remaining child or `None`.
4. Found, with two children → copy the successor’s value and remove its original occurrence.

The function’s contract is:

```text
delete_bst(root, key)

Delete key if present.
Return the root of the resulting subtree.
If key is absent, leave the tree unchanged.
```

This means callers must reconnect the returned subtree:

```python
root.left = delete_bst(root.left, key)
```

Calling the function without that assignment can lose an important connection update.

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


def delete_bst(
    root: TreeNode | None,
    key: int,
) -> TreeNode | None:
    if root is None:
        return None

    if key < root.value:
        root.left = delete_bst(root.left, key)

    elif key > root.value:
        root.right = delete_bst(root.right, key)

    else:
        # No left child: covers a leaf or right-only child.
        if root.left is None:
            return root.right

        # Only a left child.
        if root.right is None:
            return root.left

        # Two children: find the inorder successor.
        successor = root.right

        while successor.left is not None:
            successor = successor.left

        # Replace the value, then remove its old occurrence.
        root.value = successor.value
        root.right = delete_bst(root.right, successor.value)

    return root


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


def make_tree() -> TreeNode:
    return TreeNode(
        8,
        TreeNode(
            3,
            TreeNode(1),
            TreeNode(6, TreeNode(4), TreeNode(7)),
        ),
        TreeNode(
            10,
            right=TreeNode(14, left=TreeNode(13)),
        ),
    )


root = make_tree()

print(inorder(root))
# [1, 3, 4, 6, 7, 8, 10, 13, 14]

root = delete_bst(root, 3)

print(inorder(root))
# [1, 4, 6, 7, 8, 10, 13, 14]
```

Unlike search, this function **modifies the tree**.

Always capture its return value:

```python
root = delete_bst(root, key)
```

The root can change when deleting a root that has one child, or become `None` when deleting the only node.

**Dry run: delete `3`**

| Step | Decision | Connection or value update |
|---|---|---|
| At `8` | `3 < 8`, go left | Assign the result back to `8.left` |
| At `3` | Found; two children | Find minimum in right subtree |
| At `6`, then `4` | `4` is leftmost | Successor is `4` |
| At the matched node | Copy successor value | Value changes from `3` to `4` |
| Delete original `4` below `6` | It is a leaf | `6.left` becomes `None` |
| Return upward | Preserve the updated subtree | `8.left` points to the subtree rooted at `4` |

There is briefly a repeated value during the internal replacement steps. Removing the original successor restores the strict BST rule before the operation finishes.

**Why the algorithm is correct**

When the target differs from the current value, BST ordering identifies the only subtree that can contain it.

When the target matches:

- A leaf can be replaced by `None`.
- A node with one child can be replaced by that child because all descendants already obey the ancestor restrictions.
- A node with two children can take its successor’s value because that value fits between the left subtree and the remaining right subtree.

Returning the updated subtree root lets each parent preserve the correct connection.

**Complexity and mistakes**

Let `h` be the tree height.

- **Time: `O(h)`**. Searching for the target and locating/removing its successor involve paths bounded by the tree height.
- **Auxiliary space: `O(h)`** for recursion.
- Balanced tree: `O(log n)` time and stack space.
- Chain-shaped tree: `O(n)` time and stack space.

The successor search does not make this `O(h²)`: it occurs at the matched node, and its removal follows the same subtree path.

| Mistake | What goes wrong |
|---|---|
| Call deletion without assigning its return value | Parent or root may retain an outdated connection |
| Return `None` for every matched node | Discards its children |
| Use the immediate right child as the successor without checking left descendants | Can violate BST ordering |
| Copy the successor value but never delete its original occurrence | Leaves a duplicate |
| Assume the successor is always a leaf | Can lose its right child |
| Assume deletion balances the tree | Performance may remain linear on a chain |

**Tests that catch the important cases**

Each test starts with a fresh tree because deletion changes it.

```python
# Empty tree.
assert delete_bst(None, 5) is None

# Delete the only node.
single = TreeNode(5)
single = delete_bst(single, 5)
assert single is None

# Delete a root with one child.
one_child = TreeNode(5, left=TreeNode(2))
one_child = delete_bst(one_child, 5)
assert inorder(one_child) == [2]

# Missing key: contents remain unchanged.
root = make_tree()
before = inorder(root)
root = delete_bst(root, 99)
assert inorder(root) == before

# Two children.
root = make_tree()
root = delete_bst(root, 3)
assert inorder(root) == [1, 4, 6, 7, 8, 10, 13, 14]

# Successor has a right child: that child must survive.
root = TreeNode(
    5,
    TreeNode(3),
    TreeNode(9, left=TreeNode(7, right=TreeNode(8))),
)
root = delete_bst(root, 5)
assert inorder(root) == [3, 7, 8, 9]
```

Reuse Day 33’s validator after deletion as another check:

```python
assert is_valid_bst(root) is True
```

**Your assignment**

Build this tree using your Day 34 insertion function:

```python
[20, 10, 30, 5, 15, 25, 35, 27]
```

```text
          20
         /  \
       10    30
      / \   / \
     5  15 25 35
             \
              27
```

Perform these deletions **in sequence**:

| Operation | Expected inorder output |
|---|---|
| Delete `5` | `[10, 15, 20, 25, 27, 30, 35]` |
| Delete `10` | `[15, 20, 25, 27, 30, 35]` |
| Delete `20` | `[15, 25, 27, 30, 35]` |
| Delete `99` | `[15, 25, 27, 30, 35]` |

For the third operation, explain why `25` replaces `20` and why `27` must remain connected.

Submit your deletion function, tests, and a drawing after each operation. Verify both the expected values and BST validity; a valid tree that accidentally loses extra nodes is still an incorrect deletion result.

At **institute level**, explain all three cases without memorizing the code. Your most important sentence is: “The function returns the new root of this subtree.”

At **corporate level**, consider whether other code keeps references to node objects. Our two-child implementation changes the matched object’s value and removes the successor from the tree. If node identity matters, physically transplanting nodes may be more appropriate. Deep trees also require an iterative approach or a different tree implementation to avoid recursion-depth failures.

At **client level**, clarify whether “delete” means permanent removal or marking a record inactive. If nodes contain an ID plus other record fields, copying only the successor’s ID could associate it with the wrong data. Record replacement or node transplantation must preserve the complete record consistently.

A practical requirement change might be:

> “Users should be able to restore deleted products.”

That requires a recovery design, such as an inactive flag or retained deletion history. This basic BST deletion function alone does not provide undo.

Before coding, answer this:

> “If the successor has a right child, which connection will preserve that child when the successor is removed?”
