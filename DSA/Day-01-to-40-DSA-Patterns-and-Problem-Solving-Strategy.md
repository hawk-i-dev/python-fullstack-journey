**Days 1–40 DSA: the common ideas, what to remember, and how to approach a new problem**

You have practiced many problems, but they reuse a much smaller set of ideas. The goal now is to recognize those ideas and explain why they work—not memorize 40 solutions.

Think of solving a puzzle on a table. Before moving anything, you decide:

- What am I trying to find?
- What information must I remember?
- Which choices can I safely skip?
- When am I finished?
- How will I know the answer is correct?

That is the common foundation of your DSA journey.

**1. Your 40 days fit into these families**

| Days | Family | Problems you practiced | Main idea |
|---|---|---|---|
| 1–3 | Sets and hashmaps | Duplicates, Two Sum, anagrams | Remember what you have already seen |
| 4–5 | Two pointers | Palindrome, sorted Two Sum | Use two positions to eliminate unnecessary work |
| 6–7 | Sliding window | Fixed-size sum, longest unique substring | Maintain information about a moving contiguous section |
| 8–9 | Prefix sums | Range sum, subarray sum equals K | Reuse totals from earlier positions |
| 10–12 | Stacks | Parentheses, Min Stack, Daily Temperatures | Remember unfinished work |
| 13 | Queue | RecentCounter | Process or remove items in arrival order |
| 14 | Linked list | Reverse a list | Preserve the next connection before changing it |
| 15–17 | Binary search | Search Insert, Koko, shipping capacity | Eliminate half a valid search range |
| 18–23 | Recursion and memoization | Factorial, Fibonacci, array sum/max, string reversal, palindrome | Define a smaller problem and a stopping condition |
| 24–29 | Backtracking | Subsets, permutations, combinations, parentheses, Word Search | Choose, explore, undo |
| 30–32 | Tree traversal | Depth, DFS orders, level order | Visit nodes in an order suited to the question |
| 33–37 | BST operations | Validate, search/insert, delete, kth smallest, LCA | Use ordering to constrain the search |
| 38–40 | General tree DFS | LCA, diameter, balance | Combine useful answers from child subtrees |

**2. The biggest common idea: avoid repeating work**

Most improvements come from remembering something useful.

For Two Sum, you remember earlier numbers:

```text
Current number = 7
Target = 10
Need = 3

Have I already seen 3?
```

For a sliding window, you update the previous window instead of rebuilding it:

```text
New sum = old sum - outgoing value + incoming value
```

For Fibonacci, memoization saves answers to repeated subproblems.

For tree diameter, each child returns its height once, so the parent does not recalculate it.

Whenever a brute-force solution repeats a scan, ask:

> “What small piece of information could I keep so I don’t repeat this work?”

That information might be a set, frequency map, running sum, stack, or subtree height.

**3. Match the information you need to the right data structure**

| What you need to remember | Useful structure | Example |
|---|---|---|
| Whether something has appeared | Set | Contains Duplicate |
| A value’s index | Hashmap | Two Sum |
| How many times something appeared | Frequency map | Anagram, prefix-sum counting |
| Most recent unfinished item | Stack | Parentheses |
| Unresolved items in a useful order | Monotonic stack | Daily Temperatures |
| Oldest waiting item | Queue / deque | RecentCounter, BFS |
| Previous answer to a subproblem | Memo table | Fibonacci |
| Current choices and used positions | List + set/boolean array | Permutations |
| Parent-child relationships | Tree nodes | Tree traversal |

A data structure is a tool for remembering information. Choose it because of the operations you need.

For example:

```text
“Have I seen this?”             → set
“Where did I see this?”         → map to index
“How often did I see this?”     → map to count
“What was the last unfinished?” → stack
“What has waited longest?”     → queue
```

**4. Before coding, define what every variable means**

This is one of the strongest ways to prevent bugs.

Weak description:

```text
left is an index.
```

Useful description:

```text
left is the first position of the current valid window.
```

Weak description:

```text
stack stores nodes.
```

Useful description:

```text
stack stores ancestors waiting for their inorder visit.
```

For recursion, define the return value just as precisely:

```text
height(node) returns the maximum downward height in nodes.
```

That immediately tells you why returning a two-branch diameter would be wrong.

The condition that stays true as the algorithm runs is called an **invariant**. You do not need complicated language to use one:

- Two pointers: the remaining answer, if any, is between the pointers.
- Sliding window: after shrinking, the window satisfies the required condition.
- Binary search: the answer remains within the maintained search interval.
- BFS: at the start of a level, the queue contains that level’s nodes.
- BST validation: the current node must satisfy inherited lower and upper bounds.

If you cannot explain what your state means, pause before adding more code.

**5. Learn the clues—but check their conditions**

| Problem clue | Consider | Check before using it |
|---|---|---|
| Duplicates, counts, complements | Set or hashmap | Do you need presence, count, or index? |
| Sorted input and a pair target | Two pointers | Can each comparison safely eliminate a side? |
| Contiguous section | Sliding window or prefix sum | Does the condition support predictable expansion/shrinking? |
| Static range-sum queries | Prefix sums | Is the data unchanged between queries? |
| Nested matching | Stack | Does the newest opening need to close first? |
| Next warmer/greater item | Monotonic stack | What makes a pending item resolved? |
| Minimum feasible speed/capacity | Binary search on answer | Is feasibility monotonic? |
| All combinations or arrangements | Backtracking | What choices are legal, and what must be undone? |
| Hierarchy, subtree, ancestry | Tree DFS/BFS | Is the structure a tree, and is it a BST? |

A clue suggests a candidate approach; it does not prove that approach is correct.

For example, “subarray” does not automatically mean sliding window. For **Subarray Sum Equals K with arbitrary negative numbers**, prefix sums plus a frequency map work because increasing a window can either increase or decrease its sum.

**6. Remember these pattern-specific rules**

For **hashmaps**, think about what the map represents.

In Two Sum:

```python
needed = target - current
```

Check whether `needed` appeared earlier before storing the current index. This avoids using the same element twice.

For anagrams, equal length is insufficient: character frequencies must match.

For **two pointers**, explain why moving a pointer cannot discard a valid answer.

In sorted Two Sum:

```text
Sum too small → move left pointer right.
Sum too large → move right pointer left.
```

Sorted order is what makes those moves safe.

For **sliding windows**, distinguish fixed from variable size:

```text
Fixed: maintain exactly k elements.
Variable: expand, then shrink until the condition is restored.
```

A variable window often needs a `while`, not just an `if`, because removing one element may not restore validity.

For **prefix sums**, keep one convention:

```python
prefix[0] = 0
prefix[i + 1] = prefix[i] + nums[i]

# Inclusive range [left, right]:
range_sum = prefix[right + 1] - prefix[left]
```

For Subarray Sum Equals K:

```text
current_prefix - earlier_prefix = k
earlier_prefix = current_prefix - k
```

Store **counts** of earlier prefixes because the same prefix can occur multiple times. Initialize the frequency of prefix `0` to `1`.

For **stacks and queues**:

```text
Stack: last in, first out.
Queue: first in, first out.
```

In Daily Temperatures, store indexes so you can calculate waiting days. In RecentCounter, keep timestamps in the inclusive interval `[t - 3000, t]`, assuming increasing timestamps.

For **linked-list reversal**, save the next node before reversing the link:

```python
next_node = current.next
current.next = previous
previous = current
current = next_node
```

Otherwise, you can lose access to the unprocessed list.

For **binary search**, remember the reason it works:

> A decision lets you discard a region that cannot contain the answer.

For answer-space problems:

```text
Koko: larger speed cannot require more hours.
Shipping: larger capacity cannot require more days.
```

That predictable feasibility change is the essential condition.

**7. Recursion and backtracking share a foundation, but serve different purposes**

For every recursive function, write these four statements before coding:

```text
1. What does this call mean?
2. What does it return?
3. What is the base case?
4. How does each recursive call make progress?
```

Example:

```text
array_sum(index) returns the sum from index to the end.
Base case: index == len(nums) → 0.
Progress: recurse with index + 1.
Combine: nums[index] + remaining_sum.
```

Memoization helps when the **same subproblem repeats**, as in naive Fibonacci.

Backtracking explores choices:

```python
choose()
explore()
undo()
```

Remember the differences:

| Problem | Key rule |
|---|---|
| Subsets | Advance the starting index |
| Permutations | Track used positions |
| Combination Sum with reuse | Recurse with the same chosen index |
| Combination Sum without reuse | Recurse with the next index |
| Duplicate combinations | Skip equivalent choices at the same depth |
| Generate Parentheses | Never close more parentheses than have opened |
| Word Search | Do not reuse a cell within the current path; restore state afterward |

When saving a mutable path:

```python
answers.append(path.copy())
```

Saving the same list object repeatedly can make earlier answers change as backtracking continues.

**8. For trees, the most important question is: what should a child return?**

This connects your recent lessons.

| Problem | Information returned or passed |
|---|---|
| Maximum depth | Height of the subtree |
| BST validation | Whether the subtree satisfies inherited bounds |
| BST insertion/deletion | Updated subtree root |
| General-tree LCA | A target found or an LCA already discovered |
| Diameter | Height upward; track best path separately |
| Balance | Height if balanced; `-1` otherwise |

Keep these formulas separate:

```python
# Height in nodes:
1 + max(left_height, right_height)

# Diameter candidate in edges:
left_height + right_height

# Local balance:
abs(left_height - right_height) <= 1
```

For `left_height = 3` and `right_height = 1`:

```text
Ordinary height:     4 nodes
Diameter candidate: 4 edges
Locally balanced:   False
```

The numbers happen to match for height and diameter here, but they measure different things.

Also remember:

- **BST ordering** concerns values.
- **Height balance** concerns shape.
- A tree can satisfy one without satisfying the other.
- Inorder is sorted only when the tree satisfies the relevant BST ordering rules.
- A diameter or imbalance may occur below the root.
- Ordinary binary-tree LCA cannot use the BST comparison shortcut.

**9. Use this strategy for every new problem**

Follow these steps in order.

1. **Restate the requirement.**  
   “Return the length,” “return a node,” and “return the path” are different tasks.

2. **Write a tiny example with expected output.**  
   Include one normal case and one boundary case.

3. **Clarify the contract.**  
   Sorted? Duplicates? Negative numbers? Empty input? Missing target? May the input change?

4. **Describe a correct straightforward solution.**  
   Understand the repeated work before optimizing it.

5. **Choose a pattern and justify it.**  
   Explain the property that makes it valid.

6. **Define state and stopping conditions.**  
   Give each variable and return value a clear meaning.

7. **Dry-run before coding.**  
   Track the state that changes—not just the final output.

8. **Code, test, and explain complexity.**  
   Include the worst case and auxiliary memory.

For example, before implementing balance checking:

```text
Output: boolean.
Empty tree: True.
Input mutation: none.
Helper: height or -1.
Base: None returns 0.
Combine: propagate failures, check difference, return height.
```

Once those decisions are clear, the code becomes much easier to write.

**10. Test the places where assumptions break**

You do not need dozens of random examples first. Start with purposeful ones:

| Situation | Useful test |
|---|---|
| No data | Empty list/tree/string |
| Smallest valid input | One element or node |
| Boundary position | First, last, `k = 1`, `k = n` |
| Invalid request | Missing target, `k = 0`, `k > n` |
| Repetition | Duplicate values or repeated prefixes |
| Sign assumptions | Zero and negative values |
| Shape assumptions | Chain, wide tree, hidden imbalance |
| State leakage | Call the function twice with different inputs |
| Mutation | Verify input is preserved when required |

For tree deletion, checking BST validity alone is insufficient: your code could accidentally remove extra nodes and still leave a valid BST.

Check both:

```text
Correct structure/property
AND
Correct remaining contents
```

**11. Remember complexity as work performed, not a label**

Ask:

- How many times can each element enter or leave the structure?
- How many nodes can I visit?
- Am I recalculating the same result?
- How much state remains alive at once?
- How large is the output itself?

Useful anchors:

```text
One scan                         → O(n)
Repeated halving                 → O(log n)
Two full nested scans            → often O(n²)
Tree traversal                   → O(n)
One BST path                     → O(h)
Recursive tree stack             → O(h)
BFS queue                        → O(w)
All subsets                      → 2^n outputs
All permutations                 → n! outputs
```

Here, `h` is height and `w` is maximum level width.

Two pointers or a sliding window can still be `O(n)` despite containing a loop inside a loop: if each pointer advances at most `n` times, the total movement is linear.

For backtracking, storing complete outputs adds copying costs. Generating all subsets is not generally an `O(n)` task.

**12. Apply the same discipline at institute, corporate, and client levels**

| Level | What you should be able to do |
|---|---|
| Institute/interview | Explain the pattern, dry-run, implement, test, and justify complexity |
| Corporate | Define input contracts, handle realistic sizes, avoid unintended mutation, review failure cases |
| Client | Clarify what “rank,” “balanced,” “delete,” “distance,” or “same node” actually means |

A requirement change often changes more than one line:

- “Third-smallest” → third **distinct** value changes duplicate handling.
- “Delete” → allow restoration changes data retention.
- “Longest route” → greatest travel time introduces weights.
- “Common manager” → multiple managers changes a tree into a graph.
- “Balanced” → equal workload is different from equal height.

Write a concrete acceptance example before implementing the change.

**Your consolidation assignment**

Choose these six problems as a checkpoint:

| Problem | What to explain before coding |
|---|---|
| Two Sum | What does the hashmap store? |
| Longest unique substring | When and why must the window shrink? |
| Subarray Sum Equals K | Why are prefix frequencies needed? |
| Search Insert Position | What does the maintained interval mean? |
| Subsets | What is chosen, and what gets undone? |
| Balanced Binary Tree | What does the helper return, and why? |

For each one:

1. Write the input/output contract.
2. Solve a small example by hand.
3. Implement without copying.
4. Add three purposeful tests.
5. Explain time and space complexity.
6. Record any mistake and its underlying cause.

Your GitHub revision file can be named:

```text
Day-01-to-40-DSA-Patterns-and-Problem-Solving-Strategy.md
```

When you get stuck, return to this question:

> **“What information do I need to remember, what must remain true, and what can I safely stop considering?”**
