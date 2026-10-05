# Day 31 DSA — Binary Tree Traversals

Today we will learn three depth-first tree traversals:

```text
Preorder
Inorder
Postorder
```

The recursion is almost identical. The only difference is when we process the current node.

---

# 1. Example Tree

```text
        1
       / \
      2   3
     / \
    4   5
```

The three traversal results are:

```text
Preorder:  [1, 2, 4, 5, 3]
Inorder:   [4, 2, 5, 1, 3]
Postorder: [4, 5, 2, 3, 1]
```

---

# 2. Feynman Explanation

Imagine visiting rooms in a building.

At every room, you have three possible moments to write down its number:

1. Before visiting its children
2. Between the left and right children
3. After visiting both children

These moments create the three traversal orders.

```text
Write before children  → Preorder
Write between children → Inorder
Write after children   → Postorder
```

---

# 3. The Important 20%

All three traversals use this same structure:

```python
def traverse(node):
    if node is None:
        return

    # Before left
    traverse(node.left)

    # Between left and right
    traverse(node.right)

    # After right
```

Move the processing line to select the traversal.

```python
result.append(node.value)
```

---

# 4. Preorder Traversal

Order:

```text
Root → Left → Right
```

Code placement:

```python
process(node)
traverse(node.left)
traverse(node.right)
```

For the example:

```text
Visit 1
Visit left subtree: 2, 4, 5
Visit right subtree: 3
```

Result:

```python
[1, 2, 4, 5, 3]
```

## Common uses

- Copying or serializing a tree
- Creating a hierarchy outline
- Processing a parent before its children
- Prefix expression generation

---

# 5. Inorder Traversal

Order:

```text
Left → Root → Right
```

Code placement:

```python
traverse(node.left)
process(node)
traverse(node.right)
```

For the example:

```text
Left subtree: 4, 2, 5
Visit root: 1
Right subtree: 3
```

Result:

```python
[4, 2, 5, 1, 3]
```

## Important rule

Inorder traversal produces sorted values only when the tree is a Binary Search Tree.

It does not sort an arbitrary binary tree.

## Common uses

- Reading BST values in sorted order
- Infix expression generation
- Finding ordered values in a BST

---

# 6. Postorder Traversal

Order:

```text
Left → Right → Root
```

Code placement:

```python
traverse(node.left)
traverse(node.right)
process(node)
```

For the example:

```text
Left subtree: 4, 5, 2
Right subtree: 3
Visit root: 1
```

Result:

```python
[4, 5, 2, 3, 1]
```

## Common uses

- Deleting children before their parent
- Calculating folder sizes
- Evaluating expression trees
- Combining child results
- Postfix expression generation

---

# 7. The One Template to Remember

```python
def dfs(node):
    if node is None:
        return

    # Preorder processing point

    dfs(node.left)

    # Inorder processing point

    dfs(node.right)

    # Postorder processing point
```

The traversal depends entirely on where you process the node.

---

# 8. Tree Node Implementation

```python
class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```

Build the example tree:

```python
root = TreeNode(
    1,
    left=TreeNode(
        2,
        left=TreeNode(4),
        right=TreeNode(5),
    ),
    right=TreeNode(3),
)
```

---

# 9. Recursive Preorder

```python
def preorder(root):
    result = []

    def traverse(node):
        if node is None:
            return

        result.append(node.value)
        traverse(node.left)
        traverse(node.right)

    traverse(root)
    return result
```

Output:

```python
[1, 2, 4, 5, 3]
```

---

# 10. Recursive Inorder

```python
def inorder(root):
    result = []

    def traverse(node):
        if node is None:
            return

        traverse(node.left)
        result.append(node.value)
        traverse(node.right)

    traverse(root)
    return result
```

Output:

```python
[4, 2, 5, 1, 3]
```

---

# 11. Recursive Postorder

```python
def postorder(root):
    result = []

    def traverse(node):
        if node is None:
            return

        traverse(node.left)
        traverse(node.right)
        result.append(node.value)

    traverse(root)
    return result
```

Output:

```python
[4, 5, 2, 3, 1]
```

---

# 12. Dry Run — Preorder

Tree:

```text
        1
       / \
      2   3
     / \
    4   5
```

Process the node before recursion:

```text
Process 1
  Process 2
    Process 4
    Process 5
  Process 3
```

Result:

```python
[1, 2, 4, 5, 3]
```

---

# 13. Dry Run — Inorder

Process between the two recursive calls:

```text
Go left from 1
  Go left from 2
    Process 4
  Process 2
    Process 5
Process 1
  Process 3
```

Result:

```python
[4, 2, 5, 1, 3]
```

---

# 14. Dry Run — Postorder

Process after both recursive calls:

```text
Visit children of 1
  Visit children of 2
    Process 4
    Process 5
  Process 2
  Process 3
Process 1
```

Result:

```python
[4, 5, 2, 3, 1]
```

---

# 15. Unified Implementation

A practical API can support all three orders:

```python
def traverse_tree(root, order):
    valid_orders = {"preorder", "inorder", "postorder"}

    if order not in valid_orders:
        raise ValueError(
            "order must be preorder, inorder or postorder"
        )

    result = []

    def traverse(node):
        if node is None:
            return

        if order == "preorder":
            result.append(node.value)

        traverse(node.left)

        if order == "inorder":
            result.append(node.value)

        traverse(node.right)

        if order == "postorder":
            result.append(node.value)

    traverse(root)
    return result
```

Examples:

```python
print(traverse_tree(root, "preorder"))
print(traverse_tree(root, "inorder"))
print(traverse_tree(root, "postorder"))
```

---

# 16. Edge Cases

## Empty tree

```python
preorder(None)
inorder(None)
postorder(None)
```

All return:

```python
[]
```

## One node

```text
10
```

All three return:

```python
[10]
```

## Left-skewed tree

```text
    1
   /
  2
 /
3
```

Results:

```text
Preorder:  [1, 2, 3]
Inorder:   [3, 2, 1]
Postorder: [3, 2, 1]
```

## Right-skewed tree

```text
1
 \
  2
   \
    3
```

Results:

```text
Preorder:  [1, 2, 3]
Inorder:   [1, 2, 3]
Postorder: [3, 2, 1]
```

---

# 17. Complexity

Let:

```text
n = number of nodes
h = tree height
```

Every traversal visits every node once:

```text
Time: O(n)
```

The recursion stack contains at most one root-to-leaf path:

```text
Auxiliary space: O(h)
```

Balanced tree:

```text
O(log n)
```

Skewed tree:

```text
O(n)
```

The returned result requires:

```text
O(n)
```

---

# 18. Iterative Preorder

Recursive traversal can exceed Python’s recursion limit on a deep tree.

Preorder can be implemented with a stack:

```python
def preorder_iterative(root):
    if root is None:
        return []

    result = []
    stack = [root]

    while stack:
        node = stack.pop()
        result.append(node.value)

        # Push right first because the stack is LIFO
        if node.right is not None:
            stack.append(node.right)

        if node.left is not None:
            stack.append(node.left)

    return result
```

Why push right first?

```text
Stack removes the most recently pushed node.
Push right, then left.
Left is therefore processed first.
```

---

# 19. Iterative Inorder

```python
def inorder_iterative(root):
    result = []
    stack = []
    current = root

    while current is not None or stack:
        while current is not None:
            stack.append(current)
            current = current.left

        current = stack.pop()
        result.append(current.value)
        current = current.right

    return result
```

Mental model:

```text
Go left as far as possible.
Process the node.
Move into its right subtree.
```

---

# 20. Common Mistakes

## Confusing the orders

```text
Preorder:  Root, Left, Right
Inorder:   Left, Root, Right
Postorder: Left, Right, Root
```

## Assuming inorder always sorts values

It produces sorted output only for a valid BST.

## Forgetting the base case

```python
if node is None:
    return
```

Without it, recursion attempts to access children of `None`.

## Using a mutable default result

Avoid:

```python
def preorder(root, result=[]):
```

The same list is reused between calls.

Create the result inside the function instead.

## Processing both children before preorder

That becomes postorder, not preorder.

## Pushing left before right in iterative preorder

Because a stack is LIFO, that processes the right child first.

---

# 21. Traversal Comparison

| Traversal | Order | Process node | Typical use |
|---|---|---|---|
| Preorder | Root, Left, Right | Before children | Copy or serialize |
| Inorder | Left, Root, Right | Between children | Sorted BST values |
| Postorder | Left, Right, Root | After children | Delete or combine |

---

# 22. Real-World Assignment — Decision Tree Audit Exporter

A company uses a binary decision tree:

```text
                 Check Account
                 /           \
          Existing         New User
          /     \           /     \
       Approve Review    Signup   Reject
```

Build:

```python
def export_decision_tree(root, order):
    pass
```

Use cases:

```text
Preorder:
Export a parent before its rules.

Inorder:
Inspect the left decision, current rule and right decision.

Postorder:
Process child outcomes before summarizing the parent rule.
```

---

# 23. Institute-Level Assignment

Implement:

```python
preorder(root)
inorder(root)
postorder(root)
traverse_tree(root, order)
```

Requirements:

- Build at least three test trees.
- Write recursive versions.
- Write iterative preorder and inorder.
- Dry-run every traversal.
- Compare their outputs.
- Explain complexity.
- Include tests and README documentation.

Submission:

```text
day-31-tree-traversals/
├── assignment.py
├── test_assignment.py
└── README.md
```

Viva questions:

1. What changes between the three recursive algorithms?
2. Why does inorder produce sorted BST values?
3. Why is right pushed before left in iterative preorder?
4. What does the recursion stack contain?
5. When should an iterative version be preferred?

---

# 24. Corporate-Level Requirements

A production traversal service should consider:

- Type hints and docstrings
- Valid traversal-order validation
- Iterative fallback for deep trees
- Cycle protection for external data
- Duplicate node IDs
- Missing child references
- Result-size limits
- Lazy iteration for large trees
- Authorization filtering
- Deterministic output
- Structured logging and metrics
- No mutation of the tree

Suggested interface:

```python
from typing import Literal


TraversalOrder = Literal[
    "preorder",
    "inorder",
    "postorder",
]


def export_decision_tree(
    root: TreeNode | None,
    order: TraversalOrder,
) -> list[int]:
    pass
```

---

# 25. Client-Level Questions

Ask before implementation:

1. Which traversal order is required?
2. Should the result contain values, IDs or complete records?
3. Is the tree guaranteed to be binary?
4. Is it guaranteed to have no cycles?
5. Can node values be duplicated?
6. Is the tree a Binary Search Tree?
7. Should hidden or inactive nodes be excluded?
8. If a node is hidden, should its children remain visible?
9. Do we need a list, stream or downloadable file?
10. What is the maximum tree size and depth?
11. Is result ordering part of the contract?
12. Should missing child references produce errors?
13. Do we need one subtree or the entire tree?
14. Is pagination required?
15. Should traversal stop when a condition is met?

---

# 26. Requirement-Change Examples

## Client wants sorted values

Inorder works only if the tree is a valid BST.

Otherwise, traverse and explicitly sort:

```python
sorted(values)
```

## Client wants to stop at the first match

Return early instead of constructing the complete traversal.

## Client wants large results streamed

Use a generator:

```python
yield node.value
```

instead of storing every value in a list.

## Client supplies graph-like relationships

Add a visited set and cycle detection.

Do not assume external relationships form a valid tree.

---

# 27. Acceptance Criteria

For:

```text
        1
       / \
      2   3
     / \
    4   5
```

Expected:

```text
Preorder:  [1, 2, 4, 5, 3]
Inorder:   [4, 2, 5, 1, 3]
Postorder: [4, 5, 2, 3, 1]
```

Additional criteria:

```text
Empty tree returns [].
One-node tree returns its single value.
Invalid order raises a clear ValueError.
Every node appears exactly once.
The input tree remains unchanged.
Deep-tree risks are documented.
External cycles are rejected.
```

---

# 28. Testing Checklist

```python
assert preorder(None) == []
assert inorder(None) == []
assert postorder(None) == []
```

Single node:

```python
single = TreeNode(10)

assert preorder(single) == [10]
assert inorder(single) == [10]
assert postorder(single) == [10]
```

Main example:

```python
assert preorder(root) == [1, 2, 4, 5, 3]
assert inorder(root) == [4, 2, 5, 1, 3]
assert postorder(root) == [4, 5, 2, 3, 1]
```

Compare recursive and iterative versions:

```python
assert preorder(root) == preorder_iterative(root)
assert inorder(root) == inorder_iterative(root)
```

---

# 29. Git and Review Expectations

Suggested commits:

```text
feat: implement recursive tree traversals
feat: add iterative preorder and inorder
test: cover empty, balanced and skewed trees
docs: explain traversal use cases and risks
```

Review checklist:

- Is the processing line in the correct location?
- Does every function handle `None`?
- Is the tree unchanged?
- Are deep trees handled safely?
- Is the requested order validated?
- Is inorder correctly described as sorted only for BSTs?
- Are external cycles considered?

---

# 30. Your Day 31 Assignment

Complete three levels.

### Level 1 — Algorithm

Implement all three recursive traversals.

### Level 2 — Production quality

Add:

- Unified API
- Type hints
- Tests
- Iterative alternatives
- Order validation
- Cycle protection strategy

### Level 3 — Requirement analysis

Document:

- Client clarification questions
- Acceptance criteria
- Streaming approach
- Deep-tree risks
- When each traversal is useful

## Day 31 takeaway

```text
Preorder:
Root → Left → Right

Inorder:
Left → Root → Right

Postorder:
Left → Right → Root
```

The recursion remains the same.

Only the processing position changes.
