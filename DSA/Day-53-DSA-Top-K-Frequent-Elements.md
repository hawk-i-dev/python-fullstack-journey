# Day 53 DSA: Top K Frequent Elements

Day 51 ranked values by **size**.  
Day 52 maintained that ranking as values arrived.

Today, we rank values by **how often they appear**.

> Given a list of integers, return the `k` most frequent distinct values.

We’ll combine two familiar tools: **a frequency map and a bounded min-heap**.

## 1. Understand it like voting for snacks

Children vote for their favourite snacks:

```text
Apple, Banana, Apple, Mango, Banana, Apple
```

Count the votes:

```text
Apple  → 3
Banana → 2
Mango  → 1
```

The two most popular snacks are Apple and Banana.

Notice: we are not choosing the “largest” snack. We are choosing the snacks with the **largest counts**.

With numbers:

```python
nums = [4, 4, 4, 7, 7, 9]
k = 2
```

Answer:

```python
[4, 7]
```

Although `9` is the largest number, it is the least frequent.

**Main idea:**

> Count first. Then keep the `k` strongest frequency candidates.

## 2. Clarify ties before coding

What if two values appear equally often?

```python
nums = [8, 8, 3, 3, 5]
k = 1
```

Both `8` and `3` have frequency `2`.

Different problems allow different answers. For this lesson, we’ll use a precise rule:

> Higher frequency wins. If frequencies tie, the smaller number wins.

Therefore:

```python
[3]
```

Our output will also follow that ordering:

1. Frequency descending.
2. Value ascending when frequencies tie.

This makes results predictable and tests unambiguous.

## 3. Our contract

Implement:

```python
def top_k_frequent(nums, k):
    ...
```

Rules:

- `nums` contains integers.
- `k` is an integer.
- Return exactly `k` **distinct values**.
- Valid range: `0 <= k <= number_of_distinct_values`.
- For `k = 0`, return `[]`.
- Raise `ValueError` for an invalid range.
- Do not modify the input.
- Use the frequency and tie rules above.

Examples:

```python
top_k_frequent([4, 4, 4, 7, 7, 9], 2)
# [4, 7]

top_k_frequent([8, 8, 3, 3, 5], 1)
# [3]

top_k_frequent([], 0)
# []
```

Unlike Day 51, duplicates do not occupy separate output positions. They increase a value’s frequency.

## 4. Start with the straightforward solution

First count occurrences:

```python
counts = {}

for value in nums:
    counts[value] = counts.get(value, 0) + 1
```

Then sort the distinct values:

```python
ranked = sorted(
    counts,
    key=lambda value: (-counts[value], value)
)
```

Why this sorting key?

```text
-counts[value] → higher frequencies come first
value          → smaller values win ties
```

Return:

```python
ranked[:k]
```

If there are `n` input values and `u` unique values, this approach takes expected:

```text
O(n + u log u) time
O(u) extra space
```

This is a useful baseline—and a good reference implementation for testing.

But when `k` is much smaller than `u`, we can avoid sorting every distinct value.

## 5. How to think about the heap

Ask:

> “Which retained candidate should be easiest to remove?”

The weakest one:

- Lowest frequency.
- If frequencies tie, largest numeric value.

A min-heap exposes its smallest entry.

We therefore need to encode a candidate so that:

```text
Smaller heap entry = weaker candidate
```

Use this tuple:

```python
(frequency, -value)
```

Python compares tuples from left to right.

### Why negate the value?

Consider:

```text
Value 3, frequency 2 → (2, -3)
Value 8, frequency 2 → (2, -8)
```

Because:

```python
(2, -8) < (2, -3)
```

value `8` becomes the weaker candidate—the one to remove first.

That matches our rule: smaller values win frequency ties.

**Remember:**

> Design the heap key around the candidate you want to evict.

## 6. Two phases—not one

### Phase 1: Count everything

```text
Value → final frequency
```

### Phase 2: Rank distinct values

For each `(value, frequency)`:

```text
Heap has room?
    Push the candidate.

Candidate is stronger than the root?
    Replace the root.

Otherwise?
    Ignore the candidate.
```

Why count first?

A value’s frequency changes while input is being read. Today’s heap processes each distinct value **once, using its final count**.

This is a batch solution, not a live frequency tracker.

## 7. Complete runnable Python

```python
import heapq


def top_k_frequent(nums, k):
    if type(k) is not int:
        raise TypeError("k must be an integer")

    if k < 0:
        raise ValueError("k must be nonnegative")

    counts = {}

    for value in nums:
        if type(value) is not int:
            raise TypeError("values must be integers")

        counts[value] = counts.get(value, 0) + 1

    if k > len(counts):
        raise ValueError("k exceeds the number of distinct values")

    if k == 0:
        return []

    heap = []

    for value, frequency in counts.items():
        candidate = (frequency, -value)

        if len(heap) < k:
            heapq.heappush(heap, candidate)

        elif candidate > heap[0]:
            heapq.heapreplace(heap, candidate)

    # Strongest first: frequency descending,
    # then original value ascending.
    ranked = sorted(heap, reverse=True)

    return [-negative_value for frequency, negative_value in ranked]


print(top_k_frequent([4, 4, 4, 7, 7, 9], 2))
# [4, 7]

print(top_k_frequent([8, 8, 3, 3, 5], 1))
# [3]

print(top_k_frequent([-1, -1, -2, -2, 5], 2))
# [-2, -1]
```

The type checks intentionally reject booleans as well as non-integer values.

## 8. Dry run: watch the tie-breaker

Input:

```python
nums = [8, 8, 3, 3, 5, 5, 5, 9]
k = 2
```

Frequency map:

```text
8 → 2
3 → 2
5 → 3
9 → 1
```

Process distinct values in their first-appearance order:

| Value | Candidate key | Decision | Retained values |
|---|---|---|---|
| `8` | `(2, -8)` | Push | `{8}` |
| `3` | `(2, -3)` | Push | `{8, 3}` |
| `5` | `(3, -5)` | Replace weakest: `8` | `{3, 5}` |
| `9` | `(1, -9)` | Ignore | `{3, 5}` |

Why was `8` removed instead of `3`?

Both have frequency `2`, but `3` wins the tie.

Final ranked output:

```python
[5, 3]
```

The braces in the table show retained membership, not the heap’s internal array order.

## 9. The invariant and why it works

During the second phase:

> The heap contains the best `min(k, candidates_processed)` distinct values under our ranking rule.

When full:

- A candidate no stronger than the root cannot improve the retained top `k`.
- A candidate stronger than the root replaces the weakest retained candidate.

Our tuple key consistently represents the ranking, so ordinary tuple comparisons make the correct decision.

Finally, sorting the retained entries in descending tuple order produces:

```text
Highest frequency first.
Smallest original value first when tied.
```

## 10. Complexity

Let:

```text
n = number of input elements
u = number of distinct values
k = number of results requested
```

| Step | Cost |
|---|---|
| Build frequency map | Expected `O(n)` |
| Process distinct candidates | `O(u log(k + 1))` |
| Sort retained results | `O(k log(k + 1))` |
| Frequency-map storage | `O(u)` |
| Heap storage | `O(k)` |

For `k >= 1`, since `k <= u`, total expected time simplifies to:

```text
O(n + u log(k + 1))
```

For `k = 0`, this implementation still counts and validates input, taking expected `O(n)` time.

**Total auxiliary space is `O(u)`, not just `O(k)`.** The frequency map also uses memory.

## 11. Common mistakes

**Ranking by numeric value**

```text
Largest value ≠ most frequent value
```

**Using `(frequency, value)` with our tie rule**

That makes the smaller value easier to remove on a frequency tie—the opposite of what we want.

**Returning the heap directly**

A heap is not fully sorted, and its entries contain encoded keys rather than the requested values.

**Validating `k` against `len(nums)`**

The output contains distinct values. Compare `k` with:

```python
len(counts)
```

**Claiming `O(k)` total space**

That ignores the frequency map.

**Updating a stored heap entry’s frequency in place**

Changing a priority without restoring heap order can break the heap invariant. Live frequency tracking requires additional design.

## 12. Tests to run

```python
assert top_k_frequent(
    [1, 1, 1, 2, 2, 3], 2
) == [1, 2]

# Ties: smaller value first
assert top_k_frequent(
    [8, 8, 3, 3, 5], 1
) == [3]

# Negative values and ties
assert top_k_frequent(
    [-1, -1, -2, -2, 5], 2
) == [-2, -1]

# All values appear once
assert top_k_frequent(
    [9, 1, 5], 2
) == [1, 5]

# All occurrences belong to one distinct value
assert top_k_frequent(
    [7, 7, 7], 1
) == [7]

assert top_k_frequent([], 0) == []
assert top_k_frequent([1, 2], 0) == []

# Input unchanged
values = [3, 3, 1, 2]
before = values.copy()

top_k_frequent(values, 2)

assert values == before

# Invalid: only one distinct value exists
try:
    top_k_frequent([7, 7, 7], 2)
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError")

print("All tests passed!")
```

## 13. Assignment

### Part A — Predict before running

```python
nums = [6, 6, 2, 2, 2, 9, 9, 4, 4]
k = 3
```

Write:

1. The frequency map.
2. The tuple key for each distinct value.
3. Which candidate is weakest when the heap is full.
4. The final ordered result.

Expected:

```python
[2, 4, 6]
```

Explanation:

```text
2 appears 3 times.
4, 6, and 9 each appear twice.
Among the tied values, 4 and 6 win.
```

### Part B — Implement without copying

Implement `top_k_frequent()` with:

- The same tie rule.
- A heap of at most `k` candidates.
- Input validation.
- No input mutation.

Before coding, write:

```text
What does my tuple mean?
Which candidate should be at the root?
What invariant am I preserving?
What memory does the frequency map require?
```

### Part C — Verify against sorting

For valid input, compare your result with:

```python
expected = sorted(
    counts,
    key=lambda value: (-counts[value], value)
)[:k]
```

Try cases with many ties. Tie-heavy inputs expose ranking mistakes quickly.

## 14. Institute, corporate, and client-level thinking

| Setting | What to focus on |
|---|---|
| Institute/interview | Explain the count-map + heap combination and derive the tuple key |
| Corporate | Define deterministic ties, validate inputs, measure distinct-value memory, and test against a reference |
| Client | Clarify what counts as an occurrence, which period to analyze, and whether ties should extend the result |

### Real-world example: most common error codes

Suppose logs contain error-code events.

The client asks:

> “Show the top five errors.”

Clarify:

- Top five by event count or by number of affected users?
- All-time or during a specific time window?
- Count repeated log delivery once or multiple times?
- Return exactly five codes or include all codes tied at fifth?

Each choice changes the result.

### Sudden requirement change: “Make it live”

Do not assume Day 52’s tracker can be reused unchanged.

In Day 52, each score has a fixed value.

Here, an existing candidate’s **frequency changes** when another occurrence arrives. A previously unpopular value can become the most frequent.

That requires maintained counts plus a strategy for changing heap priorities or recomputing the ranking. Today’s implementation deliberately counts first, then ranks.

## 15. Remember this

> **Top K by size:** rank values.  
> **Top K by frequency:** count first, then rank distinct values.

GitHub filename:

```text
Day-53-DSA-Top-K-Frequent-Elements.md
```

**Explain-back checkpoint:** Why does `(frequency, -value)` put the correct candidate at the root when smaller values must win frequency ties?
