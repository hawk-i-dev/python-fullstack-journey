# Day 51 DSA: Binary Heap — Find the Kth Largest Element

Today we move from **complete binary trees** to a structure that uses that shape: the **binary heap**.

Our problem:

> Given an unsorted list of integers, return the kth largest value without sorting the entire list.

We’ll use a **min-heap of size `k`**.

## 1. Understand it like a winners’ club

Imagine a club with room for only **three winners**.

People arrive with scores:

```text
3, 2, 1, 5, 6, 4
```

The club keeps the three highest scores seen so far.

Whenever it becomes full, the **lowest score inside** is the entry requirement:

- A higher score replaces the lowest member.
- A lower or equal score does not improve the top three values.

At the end:

```text
Top three: 4, 5, 6
Lowest among them: 4
```

Therefore, the **third-largest value is `4`**.

**The key idea:**

> Keep the largest `k` values. Their smallest value is the kth largest overall.

## 2. What is a binary heap?

A binary heap has two separate rules.

**Shape rule:** It is a complete binary tree.

**Ordering rule:** In a min-heap, each parent is less than or equal to its children.

Example:

```text
        2
       / \
      5   3
     / \
    9   7
```

This is a valid min-heap.

Notice:

- The smallest value is at the root.
- The left and right subtrees are **not** arranged like a BST.
- The whole structure is **not sorted**.

The same heap can be stored as an array:

```python
[2, 5, 3, 9, 7]
```

For zero-based index `i`:

```python
left_child = 2 * i + 1
right_child = 2 * i + 2
parent = (i - 1) // 2  # Only when i > 0
```

Because the tree is complete, we do not need gaps or explicit child pointers in its array representation.

### Connect to Day 50

Day 50 checked the **complete shape**.

A min-heap requires that shape **plus heap ordering**.

A complete tree alone is not necessarily a heap.

## 3. Min-heap versus max-heap

| Structure | Value at the root |
|---|---|
| Min-heap | Smallest |
| Max-heap | Largest |

A natural question is:

> “If we need kth largest, why use a min-heap?”

Because we are maintaining a group of the **largest `k` values**, and we need quick access to the **weakest member of that group**.

That weakest member is the one to replace.

**Think about what must be removed—not just what the final answer is called.**

## 4. Define the problem precisely

Implement:

```python
def kth_largest(nums, k):
    ...
```

Contract:

- `nums` is a list of integers.
- `k` is an integer with `1 <= k <= len(nums)`.
- Duplicate values count separately.
- Do not modify `nums`.
- Raise `ValueError` if `k` is outside the valid range.

Example:

```text
nums = [3, 2, 1, 5, 6, 4]
k = 2

Descending order: [6, 5, 4, 3, 2, 1]
Answer: 5
```

Duplicates:

```text
nums = [5, 5, 4]
k = 2

Answer: 5, not 4
```

**Kth largest is not automatically kth distinct largest.**

## 5. Start with the simple solution

Sorting gives a straightforward baseline:

```python
def kth_largest_by_sorting(nums, k):
    if not 1 <= k <= len(nums):
        raise ValueError("k must be between 1 and len(nums)")

    return sorted(nums, reverse=True)[k - 1]
```

Time: `O(n log n)`.

This is often a perfectly reasonable solution. But it orders every value even though we only need one rank.

Can we remember just the strongest `k` candidates?

Yes—a bounded heap.

## 6. Derive the algorithm step by step

For every value:

**Case 1: The heap has fewer than `k` values.**

Add the value.

We have not filled all available places yet.

**Case 2: The heap is full and the value is greater than its root.**

Replace the root.

The incoming value is better than the weakest retained value.

**Case 3: The heap is full and the value is no greater than its root.**

Ignore it.

It cannot improve the largest `k` values retained so far.

At the end:

```python
return heap[0]
```

### The invariant

> After each processed value, the heap contains the largest `min(k, processed_count)` values seen so far, counting duplicates.

The heap’s internal array need not be sorted.

## 7. Complete runnable Python

Python’s `heapq` functions operate on an ordinary list as a min-heap.

```python
import heapq


def kth_largest(nums, k):
    if not 1 <= k <= len(nums):
        raise ValueError("k must be between 1 and len(nums)")

    heap = []

    for value in nums:
        if len(heap) < k:
            heapq.heappush(heap, value)

        elif value > heap[0]:
            heapq.heapreplace(heap, value)

    return heap[0]


nums = [3, 2, 1, 5, 6, 4]

print(kth_largest(nums, 2))  # 5
print(kth_largest(nums, 3))  # 4
print(nums)                 # Original list unchanged
```

### Understand the operations

```python
heapq.heappush(heap, value)
```

Adds a value and restores heap ordering.

```python
heap[0]
```

Reads the smallest retained value without removing it.

```python
heapq.heapreplace(heap, value)
```

Removes the smallest value and inserts the new value, restoring heap ordering.

**Important:** `heapreplace` always replaces the root. That is why our algorithm checks:

```python
value > heap[0]
```

before calling it.

## 8. Dry run

Input:

```python
nums = [3, 2, 1, 5, 6, 4]
k = 3
```

The table shows retained values in **sorted order for explanation**. The actual heap array may have a different order.

| Incoming value | Action | Retained values | Root |
|---|---|---|---|
| `3` | Fill available place | `[3]` | `3` |
| `2` | Fill available place | `[2, 3]` | `2` |
| `1` | Fill available place | `[1, 2, 3]` | `1` |
| `5` | Replace `1` | `[2, 3, 5]` | `2` |
| `6` | Replace `2` | `[3, 5, 6]` | `3` |
| `4` | Replace `3` | `[4, 5, 6]` | `4` |

Final answer:

```text
Third largest = 4
```

Notice how we never needed to sort all six values in the algorithm.

## 9. Why the algorithm is correct

Suppose the heap already contains the largest `k` values from the processed input.

Let its minimum be `m`.

For a new value `x`:

- If `x <= m`, there are already `k` retained values at least as large as `x`. Keeping the heap unchanged preserves the top-k values.
- If `x > m`, replacing `m` with `x` preserves the largest `k` values of the expanded input.

The initial filling stage retains every value while fewer than `k` exist.

Therefore, the invariant holds throughout. At the end, the smallest of the largest `k` values is exactly the kth largest.

### What about equal values?

If the heap is full and `x == heap[0]`, replacing the root with an equal value would not change the retained values.

Ignoring it is safe **for this value-ranking problem**.

If you were ranking records with tie-breaking rules, those rules would need to be part of the comparison.

## 10. Complexity

Let:

```text
n = number of input values
k = number of retained values
```

- Root lookup: `O(1)`.
- Heap insertion or replacement: `O(log k)` for `k >= 2`.
- Auxiliary space: `O(k)`.
- Total worst-case time: commonly written `O(n log k)` for `k >= 2`.

A bound that also handles `k = 1` cleanly is:

```text
O(n log(k + 1))
```

When `k = 1`, we effectively track the maximum in linear time.

**Why is the heap shallow?** Its complete-tree shape gives it logarithmic height.

## 11. Mistakes to avoid

**Mistake 1: Treat the heap list as fully sorted.**

```python
heap[0]  # Guaranteed minimum
heap[1]  # Not guaranteed second smallest
```

**Mistake 2: Replace the root for every incoming value.**

That could throw away a stronger candidate for a weaker one.

**Mistake 3: Convert the input to a set.**

That changes the meaning from ranking values with duplicates to ranking distinct values.

**Mistake 4: Allow invalid `k`.**

An empty list has no valid kth-largest value under this contract.

**Mistake 5: Forget that the heap must stay bounded.**

If you keep every value, you lose the `O(k)` memory benefit.

**Mistake 6: Assume a heap is a BST.**

A heap orders parents against children. A BST imposes ordering across whole left and right subtrees.

## 12. Tests to run

```python
assert kth_largest([3, 2, 1, 5, 6, 4], 2) == 5
assert kth_largest([3, 2, 1, 5, 6, 4], 3) == 4

# Duplicates count separately
assert kth_largest([5, 5, 4], 2) == 5

# Negative values
assert kth_largest([-5, -1, -3], 2) == -3

# One value
assert kth_largest([8], 1) == 8

# Boundary ranks
assert kth_largest([4, 1, 9], 1) == 9
assert kth_largest([4, 1, 9], 3) == 1

# All equal
assert kth_largest([7, 7, 7], 2) == 7

# Input is not mutated
values = [3, 1, 2]
before = values.copy()
assert kth_largest(values, 2) == 2
assert values == before

# Invalid requests
for values, k in [([], 1), ([1, 2], 0), ([1, 2], 3)]:
    try:
        kth_largest(values, k)
    except ValueError:
        pass
    else:
        raise AssertionError("Expected ValueError")

print("All tests passed!")
```

## 13. Assignment

### Part A — Implement without copying

Implement `kth_largest(nums, k)` using a min-heap of at most `k` values.

Before coding, write:

```text
Output contract:
State:
Invariant:
When do I replace the root?
Time:
Space:
```

Test:

```python
nums = [8, 3, 5, 12, 7, 12, 1]
k = 3
```

Expected:

```text
8
```

Why?

```text
Descending: [12, 12, 8, 7, 5, 3, 1]
```

### Part B — Return the top k values

Implement:

```python
def top_k_largest(nums, k):
    ...
```

Use the same validation rules. Return the retained values in descending order.

Expected:

```python
top_k_largest([8, 3, 5, 12, 7, 12, 1], 3)
# [12, 12, 8]
```

Hint: Sorting only the final `k` retained values adds `O(k log k)` work.

### Part C — Explain a decision

Suppose:

```text
Retained values: [7, 8, 12]
k = 3
```

For each incoming value, consider it **independently**:

```text
Incoming 6  → ignore
Incoming 7  → ignore; retained values unchanged
Incoming 10 → replace 7
```

Explain why each decision preserves the invariant.

## 14. Institute, corporate, and client-level thinking

| Setting | What to focus on |
|---|---|
| Institute/interview | Explain why kth largest uses a min-heap, state the invariant, analyze `O(k)` memory |
| Corporate | Test invalid ranks, duplicates, negative values, tie-breaking, and input preservation |
| Client | Clarify whether “top k” means values, distinct values, records, or everyone tied at the boundary |

### Practical example: highest transaction amounts

A client asks:

> “Show the top three transactions.”

Before implementing, clarify:

- Three transaction records or three distinct amounts?
- How should equal amounts be ordered?
- All-time transactions or a rolling time window?
- Can transactions be corrected or deleted?

A bounded heap works well for retaining top-k values from append-only data.

**But a rolling window is different:** old winners expire, so this simple algorithm is not sufficient by itself. Requirement changes can change the necessary data structure.

## 15. Remember this

> **To keep the strongest `k` candidates, make the weakest retained candidate easy to remove.**

That is why this problem uses a min-heap.

GitHub filename:

```text
Day-51-DSA-Min-Heap-Kth-Largest-Element.md
```

**Explain-back checkpoint:** If you wanted the **k smallest** values instead, which retained value would you need quick access to—and why?
