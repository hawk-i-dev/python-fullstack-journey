Day 33 DSA: **Validate a Binary Search Tree — Thinking with Bounds**

Today’s goal is to determine whether a binary tree follows the binary search tree rules. You’ll learn how to carry constraints through recursion—a useful idea in many problems beyond trees.

Imagine a school arranging students by roll number. A student numbered `10` stands at the entrance:

- Everyone in the left section must have a number smaller than `10`.
- Everyone in the right section must have a number greater than `10`.
- Each section follows the same rule at every smaller division.

That is a **binary search tree**, or **BST**.

For today’s problem, use a strict rule: **duplicate values are not allowed**.

```text
         10
        /  \
       5    15
      / \   / \
     2   7 12  20
```

This tree is valid. Every value in the left subtree of `10` is smaller than `10`, and every value in its right subtree is greater than `10`.

The word **every** is essential.

Consider this tree:

```text
         10
        /  \
       5    15
           / \
          6   20
```

Checking only parents and their immediate children makes everything appear correct:

```text
5 < 10
15 > 10
6 < 15
20 > 15
```

But `6` is in the **right subtree of `10`**, so it must also be greater than `10`. It fails that inherited rule.

Expected answer:

```python
False
```

**The 80/20 idea: each node has an allowed range**

Instead of asking only, “Is this node smaller or larger than its parent?”, ask:

> “Does this value fit inside all the limits inherited from its ancestors?”

Represent those limits as:

```text
lower < node.value < upper
```

At the root, there are no limits.

When moving left, the current value becomes the new upper limit. When moving right, it becomes the new lower limit.

```text
Current node: value x, allowed range (lower, upper)

Left child:   allowed range (lower, x)
Right child:  allowed range (x, upper)
```

The parentheses mean the endpoints are excluded. A value equal to a bound is invalid under our no-duplicates rule.

Follow the invalid example:

| Node | Allowed values | Result |
|---|---|---|
| `10` | No lower or upper limit | Valid |
| `5` | Less than `10` | Valid |
| `15` | Greater than `10` | Valid |
| `6` | Greater than `10` **and** less than `15` | Invalid |

Notice that moving left from `15` does not erase the lower limit inherited from `10`.

**Turn the reasoning into code**

Before coding, define the helper function’s job:

```text
check(node, lower, upper)

Return True only if the entire subtree rooted at node
satisfies the inherited bounds and the BST rules.
```

Its decisions are:

1. If the node is missing, return `True`: an empty subtree breaks no rules.
2. If its value violates either bound, return `False`.
3. Check the left subtree with a tighter upper bound.
4. Check the right subtree with a tighter lower bound.
5. Both subtrees must be valid.

Here is a runnable implementation. We use `None` to represent an absent bound.

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


def is_valid_bst(root: TreeNode | None) -> bool:
    def check(
        node: TreeNode | None,
        lower: int | None,
        upper: int | None,
    ) -> bool:
        if node is None:
            return True

        if lower is not None and node.value <= lower:
            return False

        if upper is not None and node.value >= upper:
            return False

        return (
            check(node.left, lower, node.value)
            and check(node.right, node.value, upper)
        )

    return check(root, None, None)


valid_tree = TreeNode(
    10,
    TreeNode(5, TreeNode(2), TreeNode(7)),
    TreeNode(15, TreeNode(12), TreeNode(20)),
)

invalid_tree = TreeNode(
    10,
    TreeNode(5),
    TreeNode(15, TreeNode(6), TreeNode(20)),
)

print(is_valid_bst(valid_tree))    # True
print(is_valid_bst(invalid_tree))  # False
```

The conditions use `<=` and `>=` because equality is also a violation.

The `and` operator means **both subtrees must pass**. Python also short-circuits: if checking the left subtree returns `False`, it does not evaluate the right subtree.

Here is a smaller successful dry run:

```text
        8
       / \
      3  10
       \
        6
```

```text
check(8, None, None)
│
├── check(3, None, 8)
│   ├── Missing left child → True
│   └── check(6, 3, 8)
│       ├── 3 < 6 < 8 → allowed
│       └── Both missing children → True
│
└── check(10, 8, None)
    ├── 10 > 8 → allowed
    └── Both missing children → True

Final result → True
```

Why does this work? The bounds preserve every relevant ancestor restriction. Tightening the upper bound when going left and the lower bound when going right ensures that a node fits in its entire subtree location. Recursion applies that same check to every node.

**Connect this to Day 31**

You learned that inorder traversal visits:

```text
Left → Root → Right
```

For a valid strict BST, those values are **strictly increasing**.

```text
Valid example inorder:
[2, 5, 7, 10, 12, 15, 20]

Invalid example inorder:
[5, 10, 6, 15, 20]
        ↑
        6 appears after 10, so the order decreases.
```

Checking inorder order is another valid solution. With duplicates forbidden, each value must be **greater than** the preceding value; equality fails too.

For today, practice the bounds solution first because it directly explains which ancestor restriction a node violates.

**Complexity and mistakes**

For `n` nodes and tree height `h`:

- Worst-case time is **`O(n)`** because each node is checked once.
- Auxiliary space is **`O(h)`** for recursive calls.
- A balanced tree has `h = O(log n)`.
- A chain-shaped tree has `h = O(n)` and can exceed Python’s recursion limit.

An invalid tree may return early, but the worst case still visits every node.

| Mistake | Why it fails | Correct approach |
|---|---|---|
| Check only immediate children | Misses ancestor violations | Carry lower and upper bounds |
| Reset an inherited bound | Forgets earlier restrictions | Preserve the opposite bound |
| Use `or` between subtree checks | Accepts a tree with one invalid subtree | Use `and` |
| Allow equality accidentally | Accepts duplicates under a strict rule | Reject `<= lower` and `>= upper` |
| Write `if lower:` | Treats the valid bound `0` as absent | Use `if lower is not None:` |
| Return `False` for missing children | Rejects ordinary leaves | Return `True` for `None` |

That zero-bound mistake is particularly easy to overlook:

```python
# Incorrect: zero is falsy.
if lower and node.value <= lower:
    return False

# Correct: zero is a real bound.
if lower is not None and node.value <= lower:
    return False
```

Use these tests:

```python
assert is_valid_bst(None) is True
assert is_valid_bst(TreeNode(8)) is True

assert is_valid_bst(valid_tree) is True
assert is_valid_bst(invalid_tree) is False

# Duplicate value violates the strict rule.
assert is_valid_bst(
    TreeNode(5, TreeNode(5), TreeNode(7))
) is False

# Zero must be preserved as a lower bound.
assert is_valid_bst(
    TreeNode(0, right=TreeNode(-1))
) is False

# Negative numbers are allowed.
assert is_valid_bst(
    TreeNode(-5, TreeNode(-8), TreeNode(-2))
) is True
```

**Your assignment**

Implement `is_valid_bst(root)` independently, then evaluate these trees. Write the expected answer before running your code.

```text
Tree A                 Tree B

       8                      8
      / \                    / \
     3  10                  3  10
       / \                    / \
      9  14                  6  14
```

Expected:

```python
Tree A: True
Tree B: False
```

For Tree B, explain both parts of the failure: `6` is less than its parent `10`, but it violates the lower bound `8`.

```text
Tree C                 Tree D

       0                     5
      / \                   / \
    -3   4                 2   7
        /                     /
       2                     5
```

Expected:

```python
Tree C: True
Tree D: False
```

Tree D contains a value equal to the root inside its right subtree, where every value must be strictly greater than `5`.

Your submission should include:

1. The function and construction code for all four trees.
2. Tests for an empty tree, one node, duplicates, and negative values.
3. A bounds dry run for Tree B.
4. Time and space complexity in your own words.

For an extra challenge, write an **iterative** version using a stack of `(node, lower, upper)` entries. This avoids recursive calls while preserving the same logic.

**How this applies at institute, corporate, and client levels**

At **institute level**, be ready to explain why comparing a node with only its parent is insufficient. A good demonstration is Tree B: all immediate parent-child comparisons look valid, but an ancestor constraint fails.

At **corporate level**, separate two checks: whether the input is structurally a tree, and whether its values satisfy BST ordering. This implementation assumes valid tree structure and integer values. External data may need validation for cycles, shared children, or invalid value types before traversal. For very deep trees, use the iterative approach to avoid recursion-depth failures.

At **client level**, settle the ordering contract before coding. Confirm the field being compared, whether duplicate keys are allowed, and whether the response should be a boolean or an explanation of the first violation. A hierarchy does not automatically need to be a BST; apply this rule only when ordered placement is part of the requirement.

Suppose the client changes the requirement to:

> “Allow duplicate keys, but place them only in right subtrees.”

That changes the rule to:

```text
Left subtree:  values < node.value
Right subtree: values >= node.value
```

Now bound inclusivity must be handled consistently, and tests must cover duplicates on both sides and deeper descendants. Simply accepting any sorted inorder sequence would not enforce which side equal values occupy.

Before coding today’s assignment, say this aloud:

> “A node must obey its ancestors too. Going left tightens the upper bound; going right tightens the lower bound.”
