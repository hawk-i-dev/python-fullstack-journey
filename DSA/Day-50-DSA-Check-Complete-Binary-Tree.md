# Day 50 DSA: Check Whether a Binary Tree Is Complete

Yesterday, we counted the **positions between the first and last nodes** at each level. Today, we ask:

**Are the tree’s nodes packed from left to right without leaving a gap?**

We’ll learn the concept, turn it into Python, and finish with an assignment.

## 1. Understand it like classroom seating

Imagine a classroom where students must:

1. Fill the front row before using the next row.
2. Sit from left to right.
3. Never leave an empty seat before an occupied seat.

A **complete binary tree** follows the same rules:

- Every level except possibly the last is completely filled.
- The last level is filled from left to right.

**Complete:**

```text
        1
       / \
      2   3
     / \  /
    4   5 6
```

The final row contains `4, 5, 6`, followed by one empty position. That is allowed.

**Not complete:**

```text
        1
       / \
      2   3
     /   / \
    4   6   7
```

There is an empty position where `2.right` belongs, but occupied positions appear after it.

**Remember:** Missing positions at the end are fine. Missing positions before another node are not.

## 2. The 80/20 idea

You need only three things:

| Tool | Why we need it |
|---|---|
| BFS queue | Visit positions level by level, left to right |
| `None` entries | Make missing positions visible |
| `seen_gap` flag | Remember whether an empty position has appeared |

The entire decision is:

> Once BFS encounters an empty position, every remaining position must also be empty.

If a real node appears afterward, return `False`.

## 3. Clarify the problem before coding

**Input:** The root of a valid binary tree.

**Output:** `True` if it is complete; otherwise, `False`.

Our contract:

- An empty tree is complete.
- A single-node tree is complete.
- Values can repeat.
- Values do not need to follow BST ordering.
- The tree is not modified.
- The input has no cycles or shared child nodes.

This problem checks **shape**, not values.

## 4. How to think your way to the solution

**Question 1: In what order should we inspect seats?**

Row by row, left to right.

That means **BFS**.

**Question 2: How do we notice an empty seat?**

For every real node, enqueue both children—even when a child is `None`.

**Question 3: What must we remember?**

Only whether we have already seen a missing position.

```python
seen_gap = False
```

**Question 4: What happens at each queue entry?**

```text
Missing position?
    Remember the gap.

Real node after a gap?
    Tree is not complete.

Real node before any gap?
    Add its left and right child positions.
```

**Important:** Do not enqueue children for `None`. An empty position has nothing to explore.

## 5. Complete runnable Python

```python
from collections import deque


class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


def is_complete(root):
    if root is None:
        return True

    queue = deque([root])
    seen_gap = False

    while queue:
        node = queue.popleft()

        if node is None:
            seen_gap = True
            continue

        if seen_gap:
            return False

        # Include missing children to expose gaps.
        queue.append(node.left)
        queue.append(node.right)

    return True


complete_tree = Node(
    1,
    Node(2, Node(4), Node(5)),
    Node(3, Node(6))
)

incomplete_tree = Node(
    1,
    Node(2, Node(4)),
    Node(3, Node(6), Node(7))
)

print(is_complete(complete_tree))    # True
print(is_complete(incomplete_tree))  # False
```

### Translate the code into plain English

```python
if node is None:
    seen_gap = True
    continue
```

“I found an empty seat. Remember it and inspect the next position.”

```python
if seen_gap:
    return False
```

“This position contains a node, but an earlier seat was empty. The seating rule is broken.”

## 6. Dry run: find the exact failure

Consider:

```text
        1
       / \
      2   3
     /   /
    4   6
```

BFS processes these entries until failure:

```text
1 → 2 → 3 → 4 → None → 6
```

| Entry removed | `seen_gap` before | Action |
|---|---:|---|
| `1` | False | Enqueue `2, 3` |
| `2` | False | Enqueue `4, None` |
| `3` | False | Enqueue `6, None` |
| `4` | False | Enqueue `None, None` |
| `None` | False | Set `seen_gap = True` |
| `6` | True | Return `False` |

**The failure is not simply that a child is missing. It is that a real node appears after the missing position.**

## 7. Why it works

BFS checks positions in the order completeness requires: top to bottom, left to right.

- If an occupied position appears after an empty one, the tree has a gap and is not complete.
- If no occupied position appears after the first empty one, all occupied positions form a continuous left-packed sequence.

That is exactly the definition of a complete binary tree.

**Invariant to remember:**

> `seen_gap` is true exactly when an earlier processed position was empty.

## 8. Complete, full, perfect, and balanced are different

| Term | Meaning |
|---|---|
| **Complete** | All levels filled except possibly the last, which is left-packed |
| **Full** | Every node has either zero or two children |
| **Perfect** | Every internal node has two children, and all leaves share one depth |
| **Height-balanced** | At every node, left and right subtree heights differ by at most one |

For example:

```text
    1
   /
  2
```

This tree is **complete**, but not full or perfect.

```text
    1
     \
      2
```

This tree is height-balanced, but **not complete**: the left position is empty before the occupied right position.

**An ordinary BST does not have to be complete.**

## 9. Complexity

For `n` real nodes:

- **Time: `O(n)`**
- **Auxiliary space: `O(n)` worst case**

Why is time still linear when we enqueue missing children?

Each real node adds exactly two entries. If the traversal finishes, it processes `2n + 1` entries, including the initial root entry.

The queue can hold a level-sized collection of real and missing positions, giving `O(n)` worst-case space.

## 10. Common mistakes

**Mistake 1: Skip missing children.**

```python
# This hides the gaps needed by this algorithm.
if node.left:
    queue.append(node.left)
```

Here, enqueue both children unconditionally.

**Mistake 2: Return `False` immediately at the first `None`.**

Every finite tree has missing children. Only a **later real node** proves incompleteness.

**Mistake 3: Reset the flag for every level.**

A gap in an earlier level matters. Keep the flag for the whole traversal.

**Mistake 4: Use node values to decide.**

A complete tree can contain unsorted or repeated values.

**Mistake 5: Reuse yesterday’s queue rules unchanged.**

Day 49 enqueued only real nodes and used indexes to represent gaps.

Today’s method enqueues `None` explicitly to detect gaps.

## 11. Tests you should run

```python
# Empty tree
assert is_complete(None) is True

# Single node
assert is_complete(Node(1)) is True

# Left child only: allowed at the last level
assert is_complete(Node(1, Node(2))) is True

# Right child only: gap before an occupied position
assert is_complete(Node(1, right=Node(2))) is False

# Perfect tree
assert is_complete(
    Node(1, Node(2), Node(3))
) is True

# Going deeper before the previous level is filled
assert is_complete(
    Node(1, Node(2, Node(3)))
) is False

# Duplicate values do not matter
assert is_complete(
    Node(7, Node(7), Node(7))
) is True

assert is_complete(complete_tree) is True
assert is_complete(incomplete_tree) is False

print("All tests passed!")
```

## 12. Assignment: implement without copying

### Part A — Predict and explain

For each tree:

1. State `True` or `False`.
2. If false, identify the first gap and the later node that violates the rule.

**Tree A**

```text
        10
       /  \
      5    15
     / \   /
    3   7 12
```

**Tree B**

```text
        10
       /  \
      5    15
     / \     \
    3   7     20
```

**Tree C**

```text
        10
       /
      5
     /
    3
```

**Expected results:**

```text
A → True
B → False
C → False
```

### Part B — Build the function yourself

Implement:

```python
def is_complete(root):
    ...
```

Requirements:

- Use a queue.
- Do not modify the tree.
- Handle an empty tree.
- Pass all tests above.
- Explain why `None` must remain visible.

### Part C — Add a useful diagnostic

Implement:

```python
def completeness_report(root):
    ...
```

Return:

```python
(True, None)
```

for a complete tree, or:

```python
(False, offending_node.value)
```

for the first real node encountered after a gap.

Expected:

```text
Tree A → (True, None)
Tree B → (False, 20)
Tree C → (False, 3)
```

In applications with duplicate values, return a unique node ID or location instead of relying on the value alone.

## 13. Institute, corporate, and client-level thinking

| Setting | What to focus on |
|---|---|
| **Institute/interview** | Define completeness, justify BFS, explain the gap invariant, demonstrate edge cases |
| **Corporate implementation** | Validate the actual data contract, write tests, avoid mutation, provide useful failure details |
| **Client discussion** | Confirm whether gaps are truly invalid and whether they should be rejected, reported, or repaired |

**A direct practical connection: binary heaps.**

A binary heap requires both:

1. A complete binary-tree shape.
2. Heap ordering—for example, each parent is no greater than its children in a min-heap.

Our function checks **only the shape**, not heap ordering.

If a client asks, “Is this a valid min-heap?”, completeness alone is not enough.

Also, if real input may contain cycles or shared children, this lesson’s valid-tree assumption no longer holds. Validate those separately before using this traversal.

**GitHub filename**

```text
Day-50-DSA-Check-Complete-Binary-Tree.md
```

**Explain it back in one sentence:**

> “I scan positions level by level; after the first empty position, finding any real node means the tree is not complete.”

**Your checkpoint:** Why is a root with only a left child complete, but a three-node left-only chain incomplete?
