Day 43 DSA: **Path Sum II — Return All Matching Root-to-Leaf Paths**

Yesterday, you answered:

> “Does at least one matching path exist?”

Today, you’ll return **every matching path**.

That small requirement change brings back a pattern you already know:

**Choose → Explore → Undo.**

Imagine exploring routes through rooms. You carry a notebook containing the rooms on your current route. When you reach a valid exit, take a photograph of that page. When you return to a junction, erase the room you just left before trying another route.

The notebook is your working `path`. The photographs are copies saved in `results`.

Use this tree:

```text
           5
          / \
         4   8
        /   / \
       11  13  4
      / \     / \
     7   2   5   1
```

For target `22`, there are two matching root-to-leaf paths:

```text
5 → 4 → 11 → 2 = 22
5 → 8 → 4 → 5  = 22
```

Expected output:

```python
[
    [5, 4, 11, 2],
    [5, 8, 4, 5],
]
```

Notice that this tree includes a left child `5` beneath the right-side `4`. That creates the second matching path.

**What changes from Day 42?**

| Decision | Day 42: existence | Day 43: all paths |
|---|---|---|
| Output | `True` or `False` | List of matching paths |
| Stop after first match? | Yes | No |
| Remember current route? | Not necessary | Yes |
| Empty tree | `False` | `[]` |
| Combining branches | `or` can short-circuit | Explore both branches |

The path rules remain the same:

- Start at the root.
- End at a leaf.
- Follow actual tree connections.
- Match the target exactly.
- Negative values and duplicate values are allowed.

We assume valid tree structure and return paths as lists of values, in left-first DFS order.

**The 80/20 idea**

At every node:

1. **Choose:** append its value to the working path.
2. Subtract its value from the remaining target.
3. If it is a matching leaf, save a copy of the path.
4. Otherwise, explore its children.
5. **Undo:** remove its value before returning.

```python
path.append(node.value)       # Choose
# Explore or save a result.
path.pop()                    # Undo
```

Each call must leave the shared path exactly as it found it.

**Define the state before coding**

| Variable | Meaning |
|---|---|
| `node` | Current node being explored |
| `remaining` | Sum still needed before processing this node |
| `path` | Values along the current route |
| `results` | Copies of completed matching paths |

Our helper follows this rule:

> On entry, `path` contains the route to the current node’s parent. On exit, that same path is restored.

For example, when entering node `11`:

```text
path = [5, 4]
```

After choosing `11`:

```text
path = [5, 4, 11]
```

After finishing both children and undoing `11`:

```text
path = [5, 4]
```

This restoration keeps sibling routes separate.

**Why saving a copy matters**

This is correct:

```python
results.append(path.copy())
```

This is usually wrong:

```python
results.append(path)
```

The second version stores a reference to the same working list. Later `append()` and `pop()` operations change that list, including what saved results appear to contain.

Think of the difference as:

```text
path          → the notebook you keep editing
path.copy()   → a photograph of the notebook at that moment
```

With proper backtracking, the working path is empty when traversal finishes. If every result references that same list, your answers may end up looking like:

```python
[[], []]
```

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


def path_sum(
    root: TreeNode | None,
    target: int,
) -> list[list[int]]:
    results = []
    path = []

    def dfs(node: TreeNode | None, remaining: int) -> None:
        if node is None:
            return

        # Choose this node.
        path.append(node.value)
        remaining -= node.value

        is_leaf = node.left is None and node.right is None

        if is_leaf:
            if remaining == 0:
                results.append(path.copy())
        else:
            # Explore both sides: we need every matching path.
            dfs(node.left, remaining)
            dfs(node.right, remaining)

        # Undo before returning to the parent.
        path.pop()

    dfs(root, target)
    return results


root = TreeNode(
    5,
    TreeNode(
        4,
        left=TreeNode(11, TreeNode(7), TreeNode(2)),
    ),
    TreeNode(
        8,
        TreeNode(13),
        TreeNode(4, TreeNode(5), TreeNode(1)),
    ),
)

print(path_sum(root, 22))
```

Output:

```python
[[5, 4, 11, 2], [5, 8, 4, 5]]
```

The function changes its working list but does **not** modify the tree.

`results` and `path` are created inside each public function call, so separate calls do not share answers.

**Dry run: save, undo, continue**

Follow the first matching route:

| Action | Working path | Remaining |
|---|---|---:|
| Enter `5` | `[5]` | 17 |
| Enter left `4` | `[5, 4]` | 13 |
| Enter `11` | `[5, 4, 11]` | 2 |
| Enter `7` | `[5, 4, 11, 7]` | -5 |
| Leaf fails; pop `7` | `[5, 4, 11]` | Parent still has 2 |
| Enter `2` | `[5, 4, 11, 2]` | 0 |
| Save a copy; pop `2` | `[5, 4, 11]` | Parent still has 2 |

After finishing the left subtree, the working path returns to:

```python
[5]
```

Then the right subtree begins:

```text
[5, 8]
```

The route through `13` fails because its sum is `26`. Later, we reach:

```text
[5, 8, 4, 5]
```

Its sum is `22`, so we save another copy.

At the end:

```text
Working path: []
Saved results: [[5, 4, 11, 2], [5, 8, 4, 5]]
```

The working path can become empty without damaging the saved copies.

**Why do we undo the list but not the remaining target?**

`path` is one mutable list shared by recursive calls. Appending changes that same object, so we must remove the appended value.

`remaining` is an integer parameter local to each call. This statement:

```python
remaining -= node.value
```

rebinds the local variable. It does not change the parent’s integer variable.

Therefore:

```text
Shared list change → explicitly undo with pop().
Local integer change → no explicit undo needed.
```

This distinction appears in many backtracking problems.

**Do not accidentally stop after the first result**

Day 42 used:

```python
left_succeeds or right_succeeds
```

That was appropriate because only one successful path was needed.

Today, both branches must be explored. Also, avoid returning immediately after saving a result if that return skips cleanup:

```python
# Bug: skips path.pop()
if is_leaf and remaining == 0:
    results.append(path.copy())
    return
```

Our implementation reaches `path.pop()` for every existing node, whether the path matched or failed.

**Negative values and duplicate routes**

The negative-value rule from Day 42 still applies:

```text
5 → -3, target 2 → [[5, -3]]
```

A negative remainder is not enough to reject a branch.

Distinct node paths can also have identical value sequences:

```text
      1
     / \
    2   2
```

For target `3`, our contract returns:

```python
[[1, 2], [1, 2]]
```

These are two different root-to-leaf routes. We do not silently deduplicate them.

If a client needs to distinguish them, return node IDs alongside or instead of values.

**Complexity: include the cost of returning answers**

Let:

- `n` be the number of nodes.
- `h` be the tree height.
- `S` be the total number of values across all returned paths.

Then:

- **Time: `O(n + S)`**.
- **Auxiliary space excluding results: `O(h)`**.
- **Output space: `O(S)`**.

Traversal processes each node once. Every saved path must also be copied, and those copies account for `S`.

For example, returning 100 matching paths of length 20 requires storing 2,000 values. That work cannot be ignored.

In the worst case, `S` can be `O(n²)`, so a blanket claim of `O(n)` total time would be incorrect.

**Tests to run**

```python
assert path_sum(None, 0) == []

assert path_sum(TreeNode(7), 7) == [[7]]
assert path_sum(TreeNode(7), 8) == []

assert path_sum(root, 22) == [
    [5, 4, 11, 2],
    [5, 8, 4, 5],
]

# Matching an internal prefix is insufficient.
assert path_sum(root, 9) == []

# Negative values are allowed.
negative_tree = TreeNode(5, left=TreeNode(-3))
assert path_sum(negative_tree, 2) == [[5, -3]]

# Distinct routes can have identical values.
duplicates = TreeNode(1, TreeNode(2), TreeNode(2))
assert path_sum(duplicates, 3) == [[1, 2], [1, 2]]

# Calls must not share results.
assert path_sum(TreeNode(0), 0) == [[0]]
assert path_sum(None, 0) == []
```

**Your assignment**

Build this tree from Day 42:

```text
           10
          /  \
         5   -3
        / \    \
       3   2    11
      /     \
     -2      1
```

Implement `path_sum()` independently.

Expected outputs:

```python
path_sum(root, 18)
# [[10, 5, 2, 1], [10, -3, 11]]

path_sum(root, 16)
# [[10, 5, 3, -2]]

path_sum(root, 15)
# []

path_sum(root, 100)
# []
```

For your submission:

1. Write the tree construction and function.
2. Trace every `append()` and `pop()` for target `18`.
3. Explain why `path.copy()` is required.
4. Include a test with two matching routes that share the same prefix.
5. State complexity using the total output size.

As an extension, implement:

```python
def count_path_sums(root, target) -> int:
    ...
```

Count matching root-to-leaf paths without storing their value lists. For this version, sum the child counts. You can achieve `O(n)` time and `O(h)` recursive space because no output paths need copying.

At **institute level**, connect this problem to subsets and combinations: the shared pattern is choose, explore, and undo. Explain exactly which state is shared and which state is local.

At **corporate level**, consider output size before promising to return every route in one response. If a limit is introduced, report clearly whether results are truncated. A generator can stream paths, but each yielded mutable path still needs an independent snapshot.

At **client level**, clarify whether “all paths” means distinct node routes or distinct sequences of values. Agree on ordering, identifiers, and whether the client actually needs the paths or only their count.

For GitHub:

```text
Day-43-DSA-Path-Sum-II-All-Root-to-Leaf-Paths.md
```

Before coding, answer this:

> After saving `[5, 4, 11, 2]`, we remove `2` from the working path. Why does the saved answer still contain `2`?
