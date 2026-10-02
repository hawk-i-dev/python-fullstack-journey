# Day 30 DSA — Binary Trees: Maximum Depth

## 1. Problem

Given the root of a binary tree, return its maximum depth.

Maximum depth is the number of nodes along the longest path from the root to a leaf.

Example:

```text
        3
       / \
      9   20
         /  \
        15   7
```

Expected answer:

```text
3
```

Longest paths include:

```text
3 → 20 → 15
3 → 20 → 7
```

Each contains three nodes.

---

# 2. What Is a Binary Tree?

A binary tree is a collection of nodes.

Each node can have at most:

```text
One left child
One right child
```

A basic node contains:

```python
value
left
right
```

Python representation:

```python
class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```

---

# 3. Important Tree Terms

## Root

The first node:

```text
3
```

## Parent

A node containing child references.

In the example:

```text
20 is the parent of 15 and 7
```

## Child

A node connected below another node.

## Leaf

A node with no children:

```text
9, 15 and 7
```

## Subtree

A node together with every node below it.

## Depth

Distance from the root to a node.

## Height

Longest downward path from a node to a leaf.

For today’s problem, the maximum depth of the whole tree is the height of the root when measured in nodes.

---

# 4. Feynman Explanation

Imagine a building made from connected rooms.

The root asks both children:

```text
How deep is your side?
```

Each child asks the same question below it.

The root compares both answers:

```text
left depth versus right depth
```

It keeps the larger answer and adds itself:

```text
1 + larger child depth
```

---

# 5. The Important 20%

You need only two rules:

```text
Empty node → depth 0

Real node →
1 + maximum(left depth, right depth)
```

Recurrence:

```python
depth(node) = 1 + max(
    depth(node.left),
    depth(node.right),
)
```

Base case:

```python
depth(None) = 0
```

---

# 6. How to Recognize This Pattern

Look for questions such as:

```text
Maximum depth
Tree height
Longest root-to-leaf path
Number of levels
Deepest node
```

This usually suggests:

```text
Tree DFS + recursion + combine child answers
```

---

# 7. How to Think Before Coding

For every node, ask:

### What should this function return?

The maximum depth of the subtree rooted at this node.

### What is the smallest input?

An empty subtree:

```python
node is None
```

Its depth is:

```text
0
```

### What smaller problems are available?

```python
max_depth(node.left)
max_depth(node.right)
```

### How do we combine their answers?

```python
1 + max(left_depth, right_depth)
```

The `1` counts the current node.

---

# 8. Why the Base Case Returns Zero

Consider a leaf node:

```text
7
```

Its children are empty:

```text
left depth = 0
right depth = 0
```

Calculation:

```text
1 + max(0, 0)
= 1
```

Therefore, a leaf has depth `1`.

If the empty-tree base case returned `1`, a leaf would incorrectly have depth `2`.

---

# 9. Recursive Call Tree

Example:

```text
        3
       / \
      9   20
         /  \
        15   7
```

Calls:

```text
depth(3)
├── depth(9)
│   ├── depth(None) → 0
│   └── depth(None) → 0
└── depth(20)
    ├── depth(15)
    │   ├── depth(None) → 0
    │   └── depth(None) → 0
    └── depth(7)
        ├── depth(None) → 0
        └── depth(None) → 0
```

---

# 10. Dry Run

## Node `9`

```text
left depth = 0
right depth = 0

depth(9) = 1 + max(0, 0)
         = 1
```

## Node `15`

```text
depth(15) = 1
```

## Node `7`

```text
depth(7) = 1
```

## Node `20`

```text
left depth = 1
right depth = 1

depth(20) = 1 + max(1, 1)
          = 2
```

## Root `3`

```text
left depth = 1
right depth = 2

depth(3) = 1 + max(1, 2)
         = 3
```

Answer:

```text
3
```

---

# 11. Python Implementation

```python
class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


def max_depth(root):
    # Base case: empty subtree
    if root is None:
        return 0

    # Solve the two smaller subproblems
    left_depth = max_depth(root.left)
    right_depth = max_depth(root.right)

    # Count the current node
    return 1 + max(left_depth, right_depth)
```

Build the example tree:

```python
root = TreeNode(
    3,
    left=TreeNode(9),
    right=TreeNode(
        20,
        left=TreeNode(15),
        right=TreeNode(7),
    ),
)

print(max_depth(root))
```

Output:

```text
3
```

---

# 12. Condensed Version

Once the idea is understood:

```python
def max_depth(root):
    if root is None:
        return 0

    return 1 + max(
        max_depth(root.left),
        max_depth(root.right),
    )
```

Use the expanded version while learning or debugging.

Use the concise version only when it remains clear to the team.

---

# 13. Why No Backtracking Undo Is Needed

In previous problems, we changed shared state:

```python
path.append(...)
path.pop()
```

For maximum depth, each recursive call only returns a number.

It does not modify shared state.

Therefore, no undo operation is needed.

This is tree recursion, not state-mutating backtracking.

---

# 14. Why the Algorithm Works

Assume the function correctly calculates the depth of the left and right subtrees.

The deepest path from the current node must travel through either:

```text
The left child
or
The right child
```

Therefore, choose the larger subtree depth:

```python
max(left_depth, right_depth)
```

Then count the current node:

```python
1 + max(left_depth, right_depth)
```

The base case correctly handles an empty tree, so the recursion works from leaves upward.

---

# 15. Edge Cases

## Empty tree

```python
max_depth(None)
```

Output:

```text
0
```

## One node

```python
root = TreeNode(10)
```

Output:

```text
1
```

## Only left children

```text
    1
   /
  2
 /
3
```

Output:

```text
3
```

## Only right children

```text
1
 \
  2
   \
    3
```

Output:

```text
3
```

## Balanced tree

```text
        1
       / \
      2   3
```

Output:

```text
2
```

---

# 16. Complexity

Let:

```text
n = number of nodes
h = height of the tree
```

Every node is visited once:

```text
Time: O(n)
```

The recursion stack follows one root-to-leaf path:

```text
Auxiliary space: O(h)
```

For a balanced tree:

```text
h ≈ log n
Space: O(log n)
```

For a completely skewed tree:

```text
h = n
Space: O(n)
```

---

# 17. Common Mistakes

## Returning `1` for an empty tree

Incorrect:

```python
if root is None:
    return 1
```

Correct:

```python
if root is None:
    return 0
```

## Forgetting the current node

Incorrect:

```python
return max(left_depth, right_depth)
```

Correct:

```python
return 1 + max(left_depth, right_depth)
```

## Adding both subtree depths

Incorrect:

```python
return 1 + left_depth + right_depth
```

Maximum depth follows only one path.

Correct:

```python
return 1 + max(left_depth, right_depth)
```

## Using `min()`

```python
1 + min(left_depth, right_depth)
```

This calculates something closer to the shortest depth, not maximum depth.

## Confusing nodes with edges

This lesson measures depth using nodes.

Some systems define depth using edges. Confirm the requirement.

---

# 18. Iterative BFS Alternative

Breadth-first search processes the tree one level at a time.

```python
from collections import deque


def max_depth_bfs(root):
    if root is None:
        return 0

    queue = deque([root])
    depth = 0

    while queue:
        level_size = len(queue)

        for _ in range(level_size):
            node = queue.popleft()

            if node.left is not None:
                queue.append(node.left)

            if node.right is not None:
                queue.append(node.right)

        depth += 1

    return depth
```

Use BFS when:

- Level-by-level processing is useful.
- Recursion depth may become unsafe.
- You need the width or nodes at each level.

---

# 19. DFS Versus BFS

| Requirement | Suitable approach |
|---|---|
| Simple tree-depth calculation | Recursive DFS |
| Extremely deep skewed tree | Iterative DFS or BFS |
| Process level by level | BFS |
| Find nodes at a particular level | BFS |
| Combine child results | Recursive DFS |

For untrusted or deeply skewed data, recursive Python code can exceed the recursion limit.

---

# 20. Real-World Assignment — Organization Hierarchy Depth

An organization hierarchy can be represented as a tree:

```text
CEO
├── Engineering Director
│   ├── Backend Manager
│   │   └── Senior Engineer
│   └── Frontend Manager
└── Sales Director
```

Create:

```python
def hierarchy_depth(root):
    pass
```

The result is the maximum number of employee levels from the CEO to the deepest employee.

For the example:

```text
CEO
Engineering Director
Backend Manager
Senior Engineer
```

Expected depth:

```text
4
```

For today’s assignment, model each employee with at most two direct reports so the structure remains a binary tree.

---

# 21. Institute-Level Assignment

Implement:

```python
def hierarchy_depth(root):
    pass
```

Requirements:

- Create a `TreeNode` class.
- Build at least three sample trees.
- Use recursive DFS.
- Draw one call tree.
- Perform a dry run.
- Explain time and space complexity.
- Include tests and a README.

Submission:

```text
day-30-binary-tree-depth/
├── assignment.py
├── test_assignment.py
└── README.md
```

Viva questions:

1. Why does an empty tree have depth zero?
2. Why do we use `max()`?
3. Why do we add one?
4. What is the difference between depth and height?
5. When can recursion become unsafe?

---

# 22. Corporate-Level Requirements

In a real employee system, one manager may have many reports. That structure is an N-ary tree, not necessarily a binary tree.

Production concerns include:

- Duplicate employee IDs
- Missing manager IDs
- Cyclic reporting relationships
- Multiple root employees
- Extremely deep hierarchies
- Inactive employees
- Concurrent hierarchy updates
- Database query count
- Authorization and privacy

For externally supplied relationships, validate that the structure is actually a tree.

A cycle such as:

```text
A manages B
B manages C
C manages A
```

would cause infinite recursion without cycle detection.

---

# 23. Corporate Implementation Expectations

Add:

- Type hints
- Docstrings
- Iterative fallback for deep structures
- Duplicate-ID validation
- Cycle detection
- Missing-reference validation
- Unit tests
- Performance tests
- Structured logging
- Metrics for node count and depth
- Clear error messages

Do not execute one database query for every node. That creates the N+1 query problem.

Load the required hierarchy efficiently before traversal.

---

# 24. Client-Level Questions

Before implementation, ask:

1. Is depth measured using nodes or edges?
2. Is an empty hierarchy valid?
3. Can there be multiple CEOs?
4. Can a manager have more than two reports?
5. Should inactive employees be counted?
6. Should contractors be included?
7. Can reporting relationships contain cycles?
8. What should happen when a manager record is missing?
9. Do we need only the depth or also the deepest path?
10. What is the maximum organization size?
11. Is the hierarchy updated during calculation?
12. Should permissions hide some employees?
13. Is the result required in real time?
14. Can the result be cached?

---

# 25. Requirement-Change Examples

## Client wants the deepest employee path

Return both:

```python
depth, path
```

Example:

```text
4
["CEO", "Engineering Director", "Backend Manager", "Senior Engineer"]
```

## Client allows many direct reports

Use an N-ary node:

```python
class EmployeeNode:
    def __init__(self, employee_id):
        self.employee_id = employee_id
        self.reports = []
```

Combine child results with:

```python
1 + max(depth(child) for child in node.reports)
```

## Client wants depth by department

Group or filter the hierarchy before calculating.

## Client wants live repeated queries

Consider caching and invalidating the cached depth after hierarchy changes.

---

# 26. Acceptance Criteria

```text
An empty tree returns 0.
A one-node tree returns 1.
A balanced three-level tree returns 3.
A skewed four-node tree returns 4.
Every node is visited at most once.
The input tree remains unchanged.
Cycles in external relationship data are rejected.
The node-versus-edge definition is documented.
```

---

# 27. Testing Checklist

Test:

```python
max_depth(None)
max_depth(TreeNode(1))
```

Balanced tree:

```text
        1
       / \
      2   3
```

Skewed tree:

```text
1
 \
  2
   \
    3
     \
      4
```

Uneven tree:

```text
        1
       / \
      2   3
         /
        4
       /
      5
```

Verify both implementations:

```python
assert max_depth(root) == max_depth_bfs(root)
```

---

# 28. Git and Review Expectations

Suggested commits:

```text
feat: implement binary-tree maximum depth
test: cover empty, balanced and skewed trees
refactor: add iterative traversal option
docs: explain hierarchy assumptions and risks
```

During review, verify:

- Is the base case correct?
- Does the function use `max()`, not sum?
- Is the current node counted?
- Can recursion overflow?
- Is the hierarchy guaranteed to be acyclic?
- Are database queries performed efficiently?
- Is the depth definition documented?

---

# 29. Your Day 30 Assignment

Complete three levels.

### Level 1 — Algorithm

Implement `hierarchy_depth()` using recursive DFS.

### Level 2 — Production quality

Add tests, type hints, an iterative alternative and cycle protection for external relationship data.

### Level 3 — Requirement analysis

Document:

- Client clarification questions
- Acceptance criteria
- Node-versus-edge definition
- Binary-tree versus N-ary-tree differences
- Risks involving cycles, missing records and deep recursion

## Day 30 takeaway

```text
Empty node → 0

Current node →
1 + max(left subtree depth, right subtree depth)

Time → O(n)
Space → O(tree height)
```
