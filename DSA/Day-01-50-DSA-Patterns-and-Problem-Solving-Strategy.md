# Day 1–50 DSA: Patterns and Problem-Solving Strategy

Across these 50 days, the most important skill is not memorizing 50 solutions.

It is learning to answer:

> **“What information must I remember so I don’t repeat work—and why is my next step safe?”**

This guide connects your topics into one reusable approach.

## 1. What is common in almost every problem?

Think of solving a problem like searching for a toy in a house.

You need to know:

- **Goal:** Which toy are you looking for?
- **State:** Which room are you in? What have you already checked?
- **Next action:** Where should you look next?
- **Stopping rule:** When are you finished?
- **Correctness:** How do you know you did not miss it?

The programming equivalents are:

| Question | Programming concept | Example |
|---|---|---|
| What must I produce? | Output contract | Boolean, count, index, path |
| What must I remember? | State | Set, map, pointers, queue |
| What remains true? | Invariant | Window contains no duplicates |
| How does work advance? | Transition | Move a pointer, visit a child |
| When do I stop? | Termination | Queue empty, search interval empty |
| What did I spend? | Complexity | Time and extra memory |

**A useful definition:** An invariant is a promise that stays true while the algorithm runs.

For Day 50:

> `seen_gap` means a missing position has already been processed in BFS order.

That promise makes the next decision simple.

## 2. Your Day 1–50 pattern map

| Days | Pattern family | Main topics | Core question |
|---|---|---|---|
| 1–3 | Sets and hash maps | Duplicates, Two Sum, anagrams | What have I seen, and how often? |
| 4–5 | Two pointers | Palindrome, sorted Two Sum | Which end can I safely move? |
| 6–7 | Sliding window | Fixed and variable windows | What changes when one item enters or leaves? |
| 8–9 | Prefix sums | Range sums, subarray sum counts | Can an earlier total explain the current range? |
| 10–12 | Stacks | Parentheses, Min Stack, temperatures | Which earlier item is still waiting? |
| 13 | Queue | RecentCounter | Which old items have expired? |
| 14 | Linked lists | Reverse a list | Which reference must I save before changing links? |
| 15–17 | Binary search | Insert position, Koko, shipping | Can one decision eliminate half the candidates? |
| 18–23 | Recursion | Factorial, Fibonacci, array/string problems | What smaller problem should solve the rest? |
| 24–29 | Backtracking | Subsets, permutations, combinations, parentheses, Word Search | What can I choose, and what must I undo? |
| 30–32 | Tree traversal | Depth, DFS orders, BFS levels | What visiting order fits the question? |
| 33–37 | BST operations | Validate, search, insert, delete, kth smallest, LCA | What does BST ordering let me skip? |
| 38–41 | Tree information combining | General LCA, diameter, balance, maximum path sum | What must each child return? |
| 42–44 | Tree path problems | Path Sum I, II, III | Where may the path start and end? |
| 45–47 | Tree construction and transformation | Rebuild, serialize, flatten | Which structure or identity must be preserved? |
| 48–50 | Tree level and position problems | Right view, width, completeness | Do I care about nodes, positions, or gaps? |

## 3. The biggest shortcut: clarify the output first

Similar-looking problems often need different algorithms because their outputs differ.

| Required output | Typical state |
|---|---|
| Does something exist? | Boolean or set |
| How many exist? | Counter or frequency map |
| Where is it? | Index or node reference |
| What is the best value? | Running minimum/maximum |
| Return one path | Parent references or current path |
| Return every path | Backtracking plus copied results |
| Modify the structure | Carefully preserved references |

For example:

- **Two Sum:** Store number → index because the output needs indexes.
- **Subarray Sum Equals K:** Store prefix sum → frequency because the output needs a count.
- **Path Sum II:** Copy successful paths because the output needs all paths.
- **Path Sum III:** Count matching prefixes; do not copy every path.

**Ask “what must I return?” before asking “which data structure?”**

## 4. How to recognize the main patterns

### A. Sets and hash maps: remember instead of searching again

**Simple idea:** Keep a notebook of what you have already seen.

Use:

- A **set** for membership.
- A **map** for associated information.
- A **frequency map** for counts.

Two Sum:

```python
def two_sum(nums, target):
    seen = {}

    for index, value in enumerate(nums):
        needed = target - value

        if needed in seen:
            return [seen[needed], index]

        seen[value] = index

    return None
```

**Invariant:** Before processing the current value, `seen` contains information from earlier indexes.

**Why check before inserting?** To avoid using the current element twice.

**Remember:** Hash-table lookup is typically `O(1)` on average, not an unconditional worst-case guarantee.

### B. Two pointers: use an ordering rule

For sorted Two Sum:

```text
Sum too small → move left forward.
Sum too large → move right backward.
```

Why is this safe?

If the smallest remaining number plus the largest remaining number is too small, pairing that smallest number with anything smaller cannot help.

**Pattern recognition is not enough. Explain why the discarded candidates cannot be answers.**

For palindrome checking, pointers move inward because you compare matching positions from opposite ends.

### C. Sliding window: maintain a contiguous region

**Simple idea:** Move a frame across a row of items.

Fixed window:

```text
Add the entering item.
Remove the leaving item.
Evaluate a window of exactly k items.
```

Variable window:

```text
Expand right.
While invalid, shrink left.
Record the valid result.
```

**Critical warning:** Variable sliding windows require a condition that supports safe expansion and shrinking.

For example, sum-based shrinking commonly relies on nonnegative values. With negative numbers, removing an element can increase the sum.

Do not use a window merely because the problem says “subarray.”

### D. Prefix sums: subtract the past

Define:

```text
prefix[0] = 0
prefix[i + 1] = nums[0] + ... + nums[i]
```

Then:

```python
range_sum = prefix[right + 1] - prefix[left]
```

For counting subarrays with sum `k`:

```text
current_prefix - earlier_prefix = k

Therefore:

earlier_prefix = current_prefix - k
```

That equation explains the map lookup.

**Remember these three details:**

1. Start with `{0: 1}`.
2. Query before inserting the current prefix.
3. Store frequencies, not just membership.

### E. Stack: handle the most recent unresolved item

| Problem | What the stack remembers |
|---|---|
| Valid Parentheses | Unmatched opening brackets |
| Min Stack | Values and minimum information |
| Daily Temperatures | Indexes waiting for a warmer day |

For Daily Temperatures, a new warmer day resolves earlier colder days.

**Why indexes?** The answer is a distance between days.

**Why can nested loops still be `O(n)`?** Each index is pushed once and popped at most once.

Count total operations—not just visible loops.

### F. Queue: oldest work first

Use a queue when processing order matters:

- RecentCounter removes expired calls from the front.
- Tree BFS processes earlier levels before deeper ones.

For RecentCounter at time `t`, retain timestamps in:

```text
[t - 3000, t]
```

Remove timestamps **strictly less than** `t - 3000`.

For Python queue operations, use `deque.popleft()` rather than repeatedly removing the first element of a list.

### G. Binary search: find a boundary

There are two major forms:

| Form | Search space |
|---|---|
| Search a sorted array | Indexes |
| Search for the smallest feasible answer | Speeds, capacities, other candidate values |

For Koko and shipping capacity:

```text
Smaller candidates: may fail.
Larger candidates: once feasible, stay feasible.
```

That is a **monotonic feasibility condition**.

A smallest-feasible-answer template:

```python
while low < high:
    mid = (low + high) // 2

    if feasible(mid):
        high = mid
    else:
        low = mid + 1
```

This template assumes the bounds contain a feasible answer and feasibility changes from false to true.

**Do not mix binary-search templates without checking their interval rules.**

## 5. Recursion: define the promise before writing the calls

Before implementing a recursive function, complete this sentence:

> “For this input, my function returns ______.”

Examples:

```text
height(node)
→ height of this subtree

reverse_list(head)
→ root of the reversed list

build(left, right)
→ tree represented by this inorder interval
```

Then answer:

1. What is the smallest input?
2. What does it return?
3. How does the input become smaller?
4. How do I combine smaller answers?

### The recursive leap of faith

When calculating a tree’s height, do not mentally expand the entire tree.

Assume:

```python
left_height = height(node.left)
right_height = height(node.right)
```

correctly solve the smaller subproblems.

Then combine:

```python
return 1 + max(left_height, right_height)
```

**Important distinction:**

- Ordinary recursion breaks work into smaller calls.
- Memoization saves results of repeated states.
- Backtracking explores choices and restores state.

They are related, but not interchangeable.

## 6. Backtracking: choose, explore, undo

**Simple idea:** Try a route through a maze. If you return to a junction, restore your state before trying another route.

```python
def backtrack(state):
    if finished(state):
        answers.append(path.copy())
        return

    for choice in available_choices(state):
        # Choose
        path.append(choice)

        # Explore
        backtrack(next_state)

        # Undo
        path.pop()
```

This is a conceptual template; the state and available choices depend on the problem.

### The details that change between problems

| Problem | Key decision |
|---|---|
| Subsets | Include or skip |
| Permutations | Which unused item comes next? |
| Combination Sum | Reuse the same candidate index |
| Combination Sum II | Advance the index; skip same-depth duplicates |
| Generate Parentheses | Add `)` only when fewer closes than opens |
| Word Search | Mark the current cell, then restore it |

**Two major mistakes:**

- Saving `path` instead of `path.copy()`.
- Forgetting to undo a choice before exploring a sibling branch.

**Pruning rule:** Skip a branch only when you can explain why it cannot produce a valid answer.

## 7. Trees: distinguish what you return from what you record

This is one of the most important ideas from Days 30–50.

A function may:

- **Return information the parent can extend.**
- **Record a complete answer found inside the subtree.**

These are not always the same value.

| Problem | Return to parent | Record/check locally |
|---|---|---|
| Height | `1 + max(left, right)` | Nothing extra |
| Diameter | Height | `left_height + right_height` |
| Balance | Height or failure sentinel | Height difference at every node |
| Maximum path sum | Best one-branch gain | Node value + both useful child gains |

### Why maximum path sum returns only one branch

A parent can extend a path through one child branch.

If a child returns a path already using both branches, extending it to the parent creates a three-way fork—not a simple path.

**Memory aid:**

> Return what can continue upward. Record what is already complete.

### Units matter

For our lessons:

```text
Height: number of nodes on the longest downward path.
Empty height: 0.
Leaf height: 1.
Diameter: number of edges on the longest path.
```

Therefore:

```python
diameter_through_node = left_height + right_height
```

Write down units before deriving formulas.

## 8. BSTs: use ordering, but do not invent guarantees

BST ordering helps with:

- Searching.
- Inserting.
- Deleting.
- Finding kth smallest by inorder traversal.
- Finding LCA using comparisons.

But a BST is **not automatically balanced**.

Operations following one root-to-leaf route take `O(h)`:

```text
Balanced tree: h = O(log n)
Skewed tree:   h = O(n)
```

### Three rules to remember

**Validation:** Check ancestor bounds, not only immediate children.

**Deletion:** Return the updated subtree root and reconnect it.

```python
root.left = delete_bst(root.left, key)
```

**Two-child deletion:** Replace with an inorder successor or predecessor, then remove its original occurrence.

Copying a value can affect node-identity semantics in real applications. Know whether the caller cares about values, objects, or complete records.

## 9. Path problems: define the legal path

Before solving any tree path problem, ask:

- Must it start at the root?
- Must it end at a leaf?
- Can it start anywhere?
- Can it turn through a parent?
- Are we checking existence, listing paths, counting paths, or maximizing a sum?

| Problem | Legal path | Output |
|---|---|---|
| Path Sum I | Root to leaf | Boolean |
| Path Sum II | Root to leaf | All matching paths |
| Path Sum III | Any downward ancestor-to-descendant path | Count |
| Maximum Path Sum | Any simple connected path | Maximum sum |
| Diameter | Any simple connected path | Maximum edge count |

### Why Path Sum III needs cleanup

Its prefix map represents prefixes on the **current ancestor path**.

When leaving a node:

```python
prefix_count[current_sum] -= 1
```

Otherwise, a sibling branch can accidentally use prefixes from an unrelated branch.

**General principle:**

> State must live exactly as long as the information it represents is valid.

This explains backtracking cleanup, sliding-window removal, and expired queue entries too.

## 10. Days 45–50: preserve structure and interpret positions

| Problem | Main idea | Common trap |
|---|---|---|
| Rebuild from preorder + inorder | Preorder selects root; inorder splits subtrees | Ignoring unique-value assumptions |
| Serialize/deserialize | Preserve values and missing-child structure | Omitting null markers |
| Flatten | Preserve preorder while reconnecting existing nodes | Losing the original right subtree |
| Right-side view | Last real node at each BFS level | Following only right-child links |
| Maximum width | Track positional distance | Counting only existing nodes |
| Completeness | No real node after the first BFS gap | Skipping missing children |

### Node count, width, and completeness are different

Consider a level whose occupied positions are:

```text
Positions:  0   1   2   3
Nodes:      A   B   ·   C
```

- Existing node count: **3**
- Positional width: **4**
- Left-packed: **No**

For whole-tree completeness, all earlier levels must also satisfy the filling rule.

### Mutation needs special care

Before changing a reference:

> “Will I still be able to reach everything I need afterward?”

For reversing a linked list, save `next_node`.

For flattening a tree, reconnect the original right subtree before replacing the right link.

The same reasoning appears in both problems.

## 11. A reusable problem-solving workflow

Use this checklist for every new problem.

### Step 1 — Restate the contract

Write:

```text
Input:
Output:
Constraints:
Empty-input behavior:
Duplicates allowed?
Mutation allowed?
```

### Step 2 — Work through a tiny example

Use three to six elements.

Include one example that breaks a tempting shortcut.

For completeness:

```text
Root with only a right child.
```

For tree balance:

```text
Root looks balanced, but a lower subtree is not.
```

### Step 3 — Describe the straightforward solution

Ask:

> “What would I do if efficiency did not matter?”

This gives you a correct baseline and exposes repeated work.

### Step 4 — Identify the waste

Examples:

```text
Searching earlier values repeatedly → map.
Recomputing overlapping sums → prefix sum/window.
Recomputing Fibonacci states → memoization.
Recalculating subtree heights → combine results in one traversal.
```

### Step 5 — Choose state and write the invariant

Do this before coding.

```text
State: seen set
Invariant: contains all previously processed values
```

or:

```text
State: path + visited cells
Invariant: represents the current candidate route
```

### Step 6 — Explain why each move is safe

Why can you:

- Move that pointer?
- Discard half the search interval?
- Pop that stack entry?
- Prune that branch?
- Return early?

If you cannot explain it, the algorithm may be a guess.

### Step 7 — Implement the smallest clear version

Use names that describe roles:

```python
remaining
left_height
seen_gap
prefix_count
```

### Step 8 — Test and analyze

Check correctness first, then time and space.

**Memorable sequence:**

> Contract → Example → Baseline → Waste → State → Safe move → Code → Tests.

## 12. Complexity: count the actual work

Avoid these common shortcuts in reasoning.

| Misleading shortcut | Better reasoning |
|---|---|
| “Two loops means `O(n²)`” | Count how many times each item is processed |
| “Recursion means `O(log n)` space” | Measure maximum recursion depth |
| “BST means `O(log n)`” | Check tree height |
| “Every tree algorithm is `O(n)`” | Look for repeated subtree traversals or output copying |
| “Returning every result is cheap” | Count the total output size |
| “Hash maps are always constant time” | State the average-case assumption |

Examples:

- Monotonic stack: `O(n)` because each item enters and leaves once.
- Naive Fibonacci: repeated recursive states produce exponential work.
- Memoized Fibonacci: `O(n)` distinct states under a unit-cost arithmetic model.
- Returning all subsets: output itself requires `Θ(n × 2ⁿ)` element storage/copying.
- Returning all permutations of distinct items: output requires `Θ(n × n!)`.

**Separate auxiliary space from output space.**

## 13. Tests that expose real mistakes

For every problem, choose relevant cases from this list:

1. Empty input.
2. One element or node.
3. Smallest successful case.
4. Smallest failing case.
5. Duplicates.
6. Zero and negative values.
7. No answer.
8. Multiple valid answers.
9. Boundary values.
10. Deep or skewed structure.
11. Repeated calls to detect leaked state.
12. Input unchanged when mutation is forbidden.

Examples:

- Maximum path sum: all values negative.
- Path Sum III: overlapping paths and sibling branches.
- BST validation: violation caused by a distant ancestor.
- Codec: missing tokens and extra tokens.
- Complete tree: a gap followed by a real node.
- Width: very few real nodes with large positional gaps.

## 14. Institute, corporate, and client-level strategy

| Setting | What success looks like |
|---|---|
| Institute/interview | Explain the pattern, invariant, correctness, complexity, and edge cases |
| Corporate | Implement against a clear contract, add tests, handle invalid data, preserve required behavior |
| Client | Translate the requirement into observable outcomes and agree on acceptance examples |

### Example: “Check whether our tree is valid”

That requirement is incomplete.

Ask:

- Valid as a binary tree?
- Valid as a BST?
- Complete?
- Height-balanced?
- Valid as a min-heap?
- Should invalid input be rejected or repaired?

Each question defines a different implementation.

### When requirements suddenly change

Suppose:

```text
Old: Return whether the tree is complete.
New: Explain where completeness fails.
```

Approach:

1. Write the new output contract.
2. Preserve the original invariant.
3. Add the required diagnostic state.
4. Add tests for the new output.
5. Check whether existing callers still need the boolean interface.

**Do not rewrite the algorithm merely because the output format changed.**

For production errors, reproduce the failing input, identify which assumption broke, add a regression test, and then change the implementation.

## 15. Practice assignment: prove you can choose the pattern

For each task below, write these four things **before coding**:

```text
Pattern:
State:
Invariant:
Time and space:
```

| Task | Input/example | Expected result |
|---|---|---|
| Contains Duplicate | `[4, 1, 4]` | `True` |
| Two Sum | `[2, 7, 11, 15]`, target `9` | `[0, 1]` |
| Maximum sum of exactly 3 consecutive values | `[2, 1, 5, 1, 3, 2]` | `9` |
| Count subarrays summing to 2 | `[1, 1, 1]` | `2` |
| Daily Temperatures | `[73, 74, 75, 71, 69, 72, 76, 73]` | `[1, 1, 4, 2, 1, 1, 0, 0]` |
| Minimum shipping capacity | Weights `[1, 2, 3, 1, 1]`, days `4` | `3` |
| All subsets | `[1, 2]` | `[]`, `[1]`, `[2]`, `[1, 2]`, any order |
| Tree report below | Given tree | Results below |

For the tree report, use:

```text
        1
       / \
      2   3
     / \
    4   5
```

Implement separate functions and verify:

```text
Maximum depth:     3
Diameter in edges: 3
Height-balanced:   True
Right-side view:   [1, 3, 5]
Maximum width:     2
Complete:          True
```

Then move node `5` from `2.right` to `3.right`:

```text
        1
       / \
      2   3
     /     \
    4       5
```

Predict before running:

```text
Maximum depth:     3
Diameter in edges: 4
Height-balanced:   True
Right-side view:   [1, 3, 5]
Maximum width:     4
Complete:          False
```

This exercise shows why **balanced, wide, visible, and complete describe different properties**.

## 16. What to remember permanently

You do not need to remember every line of every solution.

Remember these questions:

1. **What exactly must I return?**
2. **Which information prevents repeated work?**
3. **What promise does my state maintain?**
4. **Why is the next move safe?**
5. **What must be restored or preserved?**
6. **Which small input breaks my assumption?**
7. **How many times can each item be processed?**

Your next milestone is not another perfect MCQ score. It is being able to **choose a pattern, explain it, implement it without copying, and test it on an unfamiliar variation**.

Suggested GitHub filename:

```text
Day-1-50-DSA-Patterns-and-Problem-Solving-Strategy.md
```
