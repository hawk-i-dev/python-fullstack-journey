Day 32 DSA: **Binary Tree Level Order Traversal — BFS Using a Queue**

Yesterday, you practiced DFS: preorder, inorder, and postorder. Today, you will visit a tree **one level at a time** and return each level as a separate list.

By the end, you should be able to explain why a queue works, write the code independently, and adapt it to a client’s reporting requirement.

Imagine children standing in rows for a school photograph. You finish calling everyone in the first row, then the second row, then the third. You also keep a separate attendance list for each row.

That is level order traversal:

```text
          1             Level 1
         / \
        2   3           Level 2
       / \
      4   5             Level 3
```

Expected output:

```python
[[1], [2, 3], [4, 5]]
```

The inner lists matter: `[1, 2, 3, 4, 5]` gives the visit order but loses the level boundaries.

**The 80/20 ideas to remember**

Most of today’s problem comes down to four rules:

1. Use a **queue**: the first node added is the first node processed.
2. Before processing a level, save `level_size = len(queue)`.
3. Process exactly that many nodes, adding their children to the queue.
4. Save the current level’s values, then repeat.

This method is called **breadth-first search**, or **BFS**.

A queue behaves like a ticket counter:

```text
Front → [person A, person B, person C] ← Back

Serve A first.
New arrivals join behind C.
```

In Python, use `deque`:

```python
from collections import deque

queue = deque()
queue.append(10)       # Add at the back
queue.append(20)

first = queue.popleft()

print(first)          # 10
print(list(queue))    # [20]
```

`append()` and `popleft()` take `O(1)` time. A list’s `pop(0)` shifts the remaining elements, so it is a poor choice for this queue.

**Think through the problem before writing code**

The input is a root node. Each node has a value and up to two children. We assume the input is a valid tree.

The output is a list of lists, ordered from top to bottom and left to right within each level.

Now decide what information you must remember:

| Variable | Meaning |
|---|---|
| `queue` | Nodes waiting to be processed |
| `level_size` | Number of nodes belonging to the current level |
| `level` | Values collected for this level |
| `result` | All completed levels |

The empty-tree case is straightforward: no root means no levels, so return `[]`.

The key reasoning step is this:

> At the beginning of each outer loop, the queue contains exactly the nodes in the next level to process.

During processing, it will temporarily contain a mixture of remaining current-level nodes and newly added next-level nodes. Saving `level_size` tells us where to stop.

For example:

```text
Current level starts:
queue = [2, 3]
level_size = 2

Process 2, then add its children:
queue = [3, 4, 5]

Process 3:
queue = [4, 5]

We processed exactly 2 nodes.
Therefore, this level is complete.
```

Although `4` and `5` are now waiting, they belong to the next level.

Here is the complete runnable program:

```python
from collections import deque


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


def level_order(root: TreeNode | None) -> list[list[int]]:
    if root is None:
        return []

    result = []
    queue = deque([root])

    while queue:
        # Freeze the number of nodes in this level.
        level_size = len(queue)
        level = []

        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.value)

            if node.left is not None:
                queue.append(node.left)

            if node.right is not None:
                queue.append(node.right)

        result.append(level)

    return result


root = TreeNode(
    1,
    left=TreeNode(
        2,
        left=TreeNode(4),
        right=TreeNode(5),
    ),
    right=TreeNode(3),
)

print(level_order(root))
```

Output:

```text
[[1], [2, 3], [4, 5]]
```

Notice that `queue` stores **node objects**, while `level` stores **values**. We need the objects in the queue because we must access their children later.

Also, `_` means we do not need the loop counter. We simply want to repeat the operation `level_size` times.

Follow the execution one operation at a time:

| Action | Queue afterward, shown as values | Current level | Completed result |
|---|---|---|---|
| Add root | `[1]` | — | `[]` |
| Process `1`; add `2`, `3` | `[2, 3]` | `[1]` | `[]` |
| Finish level 1 | `[2, 3]` | — | `[[1]]` |
| Process `2`; add `4`, `5` | `[3, 4, 5]` | `[2]` | `[[1]]` |
| Process `3` | `[4, 5]` | `[2, 3]` | `[[1]]` |
| Finish level 2 | `[4, 5]` | — | `[[1], [2, 3]]` |
| Process `4` | `[5]` | `[4]` | `[[1], [2, 3]]` |
| Process `5` | `[]` | `[4, 5]` | `[[1], [2, 3]]` |
| Finish level 3 | `[]` | — | `[[1], [2, 3], [4, 5]]` |

The algorithm works because the queue keeps parents ahead of the children they add. Processing the saved number of nodes finishes the current level, leaving precisely the next level waiting. Adding left before right preserves the required order.

**Time and memory**

Let `n` be the number of nodes and `w` be the maximum number of nodes in any one level.

- **Time: `O(n)`** — every node enters and leaves the queue once.
- **Auxiliary space: `O(w)`** — the queue and current-level list grow with the tree’s width.
- **Returned output: `O(n)`** — every node’s value appears in the answer.

The queue can temporarily hold parts of two adjacent levels; its size is still bounded by a constant multiple of `w`.

Compare this with yesterday’s recursive DFS:

| Tree shape | BFS queue | Recursive DFS stack |
|---|---:|---:|
| A long chain | `O(1)` | `O(n)` |
| A balanced, wide tree | Can be `O(n)` | `O(log n)` |

Choose based on the required output and the tree’s shape. BFS does not always use less memory.

**Mistakes to watch for**

| Mistake | What goes wrong | Correction |
|---|---|---|
| Use `queue.pop()` | Processes the newest node first | Use `popleft()` |
| Keep processing until the queue empties inside one level | Children get mixed into their parents’ level | Process the saved `level_size` |
| Add missing children | Later code tries to read attributes from `None` | Check each child before adding it |
| Create `level = []` outside the outer loop | Values accumulate across levels | Create a fresh list for each level |
| Add the right child first | Changes the required left-to-right order | Add left, then right |
| Deduplicate by node value | Distinct nodes with equal values disappear | Process every node in the tree |

For a valid tree, you do not need a visited set: every non-root node has exactly one parent. If external data contains cycles or shared child references, clarify and validate that structure before treating it as a tree.

Try these checks after the program:

```python
assert level_order(None) == []

assert level_order(TreeNode(7)) == [[7]]

assert level_order(root) == [[1], [2, 3], [4, 5]]

chain = TreeNode(1, right=TreeNode(2, right=TreeNode(3)))
assert level_order(chain) == [[1], [2], [3]]

duplicates = TreeNode(5, TreeNode(5), TreeNode(5))
assert level_order(duplicates) == [[5], [5, 5]]

# Calling the function again should produce the same result.
assert level_order(root) == [[1], [2, 3], [4, 5]]
```

**Your assignment: build a tree-level report**

Use this tree:

```text
           10
          /  \
         6    15
        / \     \
       3   8     20
```

Complete these three tasks. Write your first attempt without copying the reference implementation.

1. Implement `level_order(root)`.

   Expected:

   ```python
   [[10], [6, 15], [3, 8, 20]]
   ```

2. Implement `level_sums(root)` to return one total per level.

   Expected:

   ```python
   [10, 21, 31]
   ```

   Hint: reset a total at the beginning of each level and add each processed value.

3. Implement `max_level_node_count(root)` to return the largest number of **actual nodes** in a level. Do not count gaps or missing positions.

   Expected:

   ```python
   3
   ```

   Hint: the saved `level_size` already gives the number of nodes in that level.

For an empty tree, the expected results are `[]`, `[]`, and `0`, respectively. None of these functions should modify the tree.

At **institute level**, submit the three functions, test cases, and a handwritten dry run. Be ready to explain why the level size is saved before processing and why children must wait until the next iteration.

At **corporate level**, treat the output order and empty-input behavior as part of the function’s contract. Test missing children and duplicate values, review memory usage for wide trees, and agree on how external data becomes a valid tree. Keep traversal independent of database fetching so you can test it without database access.

At **client level**, confirm what a “level report” means before implementation. Does the root count as level `0` or `1`? Should the report contain names, IDs, or complete records? Should siblings retain their existing order or be sorted? Can a parent have more than two children? Real organization hierarchies commonly require an N-ary tree, where each node holds a list of children.

Suppose the client suddenly says, “Return only level 3.”

First, make the acceptance example concrete:

```text
Root is level 1.
For the assignment tree, level 3 returns [3, 8, 20].
A level beyond the tree's depth returns [].
```

Then adapt the traversal to track the current level and stop once the requested level has been collected. Add tests for the root level, the requested level, and a level that does not exist. If an existing API already returns all levels, preserve its agreed behavior or coordinate the API change with its consumers.

Before considering today complete, explain this sentence in your own words:

> “I save the queue’s size because adding children changes the queue, but it must not change how many nodes belong to the level I am currently processing.”

Then implement the three assignment functions and send your code for review.
