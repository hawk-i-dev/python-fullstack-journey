Day 44 DSA: **Path Sum III — Count Downward Paths Using Prefix Sums**

Today, we combine **Day 9’s prefix sums and hashmap** with **tree DFS and backtracking**.

The requirement changes again:

> Count all downward paths whose values add up to the target. A path can start and end anywhere, but it must move from parent to child.

Unlike Days 42–43, the path does **not** need to start at the root or end at a leaf.

```text
           10
          /  \
         5   -3
        / \    \
       3   2    11
      / \   \
     3  -2   1
```

For target `8`, there are three matching paths:

```text
5 → 3       = 8
5 → 2 → 1   = 8
-3 → 11     = 8
```

Expected result:

```python
count_path_sum(root, 8)  # 3
```

The route `3 → 5 → 2` is not allowed because it moves upward and then downward. Today’s paths always move downward.

**Keep the three path-sum problems separate**

| Day | Allowed start | Allowed end | Output |
|---|---|---|---|
| 42 | Root | Leaf | Boolean |
| 43 | Root | Leaf | All matching paths |
| 44 | Any node | Any descendant, including itself | Count |

We count nonempty paths. Different node routes count separately, even if their values are identical.

**Begin with the straightforward solution**

One correct approach is:

1. Choose every node as a potential starting point.
2. Explore downward from it.
3. Count routes that reach the target.

But this repeatedly visits the same descendants. On a chain, the work can be:

```text
n + (n - 1) + ... + 1 = O(n²)
```

We can avoid those repeated searches by remembering prefix sums on the **current ancestor path**.

**The Feynman explanation: subtract an earlier total**

Imagine recording a running total as you walk down the tree.

For this route:

```text
10 → 5 → 3
```

The running totals are:

```text
After 10: 10
After 5:  15
After 3:  18
```

At node `3`, the total from the root is `18`.

We want a path totaling `8`:

```text
18 - earlier_total = 8
earlier_total = 10
```

An earlier total of `10` exists just after the root. Removing that initial portion leaves:

```text
5 → 3 = 8
```

The same identity from Day 9 works here:

```python
earlier_prefix = current_prefix - target
```

The important difference is that a tree has branches. We must remember prefixes from **ancestors on the active route**, not nodes from unrelated branches.

**The 80/20 rules**

At each node:

1. Add its value to the running prefix sum.
2. Count earlier prefixes equal to `current_prefix - target`.
3. Add the current prefix to the frequency map.
4. Explore both children.
5. Remove the current prefix when returning.

```text
Count → Add → Explore → Undo
```

The order matters.

We count **before** adding the current prefix. Otherwise, when the target is zero, the current prefix could match itself and incorrectly count an empty path.

**Define the state precisely**

| Variable | Meaning |
|---|---|
| `running` | Sum from the root through the current node after addition |
| `prefix_count` | Frequencies of prefix sums before the current node on its ancestor route |
| `total` | Matching paths found by the current call and its descendants |

Before traversal, initialize:

```python
prefix_count = {0: 1}
```

This represents the empty prefix **before the root**.

It lets us count a path that starts at the root. For example:

```text
Current root-to-node sum = 8
Target = 8
Needed earlier prefix = 0
```

The stored zero contributes one match.

The zero prefix is a boundary marker, not an empty path being counted.

**Why frequencies are needed**

Several ancestor prefixes can have the same sum.

Consider:

```text
0 → 0 → 0
```

For target `0`, the matching paths are:

```text
Three single-node paths
Two two-node paths
One three-node path

Total = 6
```

At the last node, three earlier zero prefixes are available, representing three possible starting positions.

A set would remember only “zero exists.” A frequency map remembers **how many valid starting boundaries exist**.

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


def count_path_sum(root: TreeNode | None, target: int) -> int:
    prefix_count = {0: 1}

    def dfs(node: TreeNode | None, running: int) -> int:
        if node is None:
            return 0

        running += node.value

        # Count paths ending at this node.
        needed = running - target
        total = prefix_count.get(needed, 0)

        # Make this prefix available to descendants.
        prefix_count[running] = prefix_count.get(running, 0) + 1

        total += dfs(node.left, running)
        total += dfs(node.right, running)

        # Undo this node's contribution before visiting other branches.
        prefix_count[running] -= 1

        if prefix_count[running] == 0:
            del prefix_count[running]

        return total

    return dfs(root, 0)


root = TreeNode(
    10,
    TreeNode(
        5,
        TreeNode(3, TreeNode(3), TreeNode(-2)),
        TreeNode(2, right=TreeNode(1)),
    ),
    TreeNode(-3, right=TreeNode(11)),
)

print(count_path_sum(root, 8))  # 3
```

The function does not modify the tree. Each public call creates a fresh map.

Deleting entries whose frequency reaches zero keeps the map limited to prefixes still relevant to the current route.

**Dry run: where the three matches appear**

The map below shows only the relevant matching prefix, not every entry.

| Route to current node | Running sum | Needed prefix: `running - 8` | Matches added |
|---|---:|---:|---|
| `10` | 10 | 2 | 0 |
| `10 → 5` | 15 | 7 | 0 |
| `10 → 5 → 3` | 18 | 10 | 1: `5 → 3` |
| `10 → 5 → 2 → 1` | 18 | 10 | 1: `5 → 2 → 1` |
| `10 → -3` | 7 | -1 | 0 |
| `10 → -3 → 11` | 18 | 10 | 1: `-3 → 11` |

Other visited nodes contribute zero matches. The total is `3`.

**Backtracking prevents paths from crossing branches**

Consider:

```text
       1
      / \
     2   3
```

Target: `1`.

The only valid matching path is the root by itself:

```python
answer = 1
```

During the left traversal, prefix `3` appears:

```text
1 + 2 = 3
```

Before exploring the right child, that prefix must be removed.

At the right child:

```text
running = 1 + 3 = 4
needed = 4 - 1 = 3
```

If the left branch’s prefix `3` remained in the map, it would create a false match. That prefix is not an ancestor boundary of the right child.

The map must describe the current route, not the entire traversal history.

**Connect this to yesterday’s `path.pop()`**

Yesterday, undo meant:

```python
path.pop()
```

That removed the current node from the working route.

Today, undo means:

```python
prefix_count[running] -= 1
```

That removes the current node’s prefix contribution from the working frequency map.

Neither operation erases answers already found.

```text
Working state changes during exploration.
Completed answers remain preserved.
```

We decrement by one rather than deleting immediately because an ancestor might have the same prefix sum.

For example, if the count is `3`, returning from one node should leave it at `2`.

**Why the algorithm works**

Any downward path ending at the current node can be described as:

```text
Root-to-current sum
minus
the prefix just before the path starts
```

Therefore, it matches the target exactly when:

```python
earlier_prefix == running - target
```

The map contains precisely those valid earlier prefixes on the current ancestor route.

Each matching path is counted once, when its endpoint is processed. Backtracking removes prefixes that would become invalid after leaving a branch.

**Complexity and mistakes**

Let `n` be the number of nodes and `h` the tree height.

- **Expected time: `O(n)`**, assuming average constant-time dictionary operations.
- **Auxiliary space: `O(h)`** for recursion and active prefix entries.
- A deep chain may exceed Python’s recursion limit.

| Mistake | Why it fails |
|---|---|
| Require a root start or leaf end | Misses paths allowed by today’s contract |
| Use a set instead of frequencies | Undercounts repeated prefix sums |
| Add the current prefix before counting | Can count an empty path when target is zero |
| Forget `{0: 1}` | Misses root-starting paths unless handled separately |
| Forget to undo after exploring children | Can count invalid cross-branch paths |
| Delete a repeated prefix without decrementing | Removes prefixes still needed by ancestors |
| Prune when the running sum exceeds the target | Negative values can change the sum later |

**Tests to run**

```python
assert count_path_sum(root, 8) == 3
assert count_path_sum(None, 0) == 0

assert count_path_sum(TreeNode(8), 8) == 1
assert count_path_sum(TreeNode(8), 0) == 0

# Repeated prefixes: three + two + one matching paths.
zeros = TreeNode(0, left=TreeNode(0, left=TreeNode(0)))
assert count_path_sum(zeros, 0) == 6

# Must not count across sibling branches.
siblings = TreeNode(1, TreeNode(2), TreeNode(3))
assert count_path_sum(siblings, 1) == 1

# A negative value can complete the target.
negative = TreeNode(5, left=TreeNode(-3))
assert count_path_sum(negative, 2) == 1

# Calls must not share prefix state.
assert count_path_sum(TreeNode(0), 0) == 1
assert count_path_sum(None, 0) == 0
```

**Your assignment**

Build:

```text
          1
         / \
       -1   1
       /     \
      1      -1
```

For target `0`, the four matching paths are:

```text
Root 1 → left -1
Left -1 → its child 1
Right 1 → its child -1
Root 1 → right 1 → right -1   # Not a match: sum is 1
```

Be careful: that last route is not valid for target zero. The fourth actual match is:

```text
Root 1 → left -1 → left 1   # Also sum 1, not zero
```

Rather than guessing from the picture, enumerate each endpoint carefully. This tree has **three**, not four, matching paths:

```python
count_path_sum(root, 0)  # 3
```

Your task is to demonstrate that result using the prefix map:

1. Implement the function independently.
2. Record `running`, `needed`, and matches added at each node.
3. Show the frequency map after each undo.
4. Test target `1`; the expected count is `6`.
5. Explain why a prefix from the left branch must not remain during the right traversal.

As an extension, write the slower “start DFS from every node” solution and compare its outputs with the prefix-sum version on small trees. Agreement across carefully chosen cases is a useful correctness check.

At **institute level**, explain the subtraction identity and the meaning of `{0: 1}`. Dry-run repeated zeros to demonstrate why counts matter.

At **corporate level**, keep traversal state local to each request and restore it when leaving branches. For an iterative version, explicit entry and exit events can perform the add and undo steps without recursive calls.

At **client level**, clarify whether paths must move downward, whether single-node paths count, and whether distinct node routes with identical values count separately. If the request changes to returning every route, the output size and implementation requirements also change.

For GitHub:

```text
Day-44-DSA-Path-Sum-III-Prefix-Sums-and-Backtracking.md
```

Before coding, answer this: **if `running = 18`, `target = 8`, and `prefix_count[10] = 2`, how many matching paths end at the current node?**
