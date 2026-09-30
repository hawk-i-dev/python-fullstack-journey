# Day 27 DSA — Combination Sum II: Use Each Item Once

## 1. Problem

Given a list of positive numbers and a target:

- Each input position may be used only once.
- The input may contain duplicate values.
- Return only unique combinations.
- Every combination must total the target.

```python
candidates = [10, 1, 2, 7, 6, 1, 5]
target = 8
```

Expected output:

```python
[
    [1, 1, 6],
    [1, 2, 5],
    [1, 7],
    [2, 6],
]
```

The output must not contain duplicate combinations.

---

# 2. Difference Between Day 26 and Day 27

## Day 26 — Combination Sum I

```text
Input candidates are unique.
A candidate may be reused.
Recursive call uses index.
```

```python
backtrack(index, remaining - number)
```

## Day 27 — Combination Sum II

```text
Input may contain duplicates.
Each input position may be used once.
Recursive call uses index + 1.
Skip equal choices at the same recursion level.
```

```python
backtrack(index + 1, remaining - number)
```

Remember:

```text
Reuse allowed     → index
Use once          → index + 1
Duplicate input   → skip equal choices at the same level
```

---

# 3. Feynman Explanation

Imagine there are physical coupon cards:

```text
1, 1, 2, 5, 6, 7, 10
```

Even though two cards both contain `1`, they are two separate cards.

You may use each card only once.

This is allowed:

```text
1 + 1 + 6 = 8
```

But after using the first `1`, you cannot reuse that same card.

Also, these are the same combination:

```text
[1, 2, 5]
[1, 5, 2]
[2, 1, 5]
```

We should return only one of them.

---

# 4. The Important 20%

Four rules solve most of this problem:

```text
remaining == 0 → save path.copy()
number > remaining → break
recurse with index + 1 → use each position once
same value at same level → skip duplicate branch
```

The duplicate rule is:

```python
if index > start and numbers[index] == numbers[index - 1]:
    continue
```

---

# 5. Why Sort First?

Sort a copy of the input:

```python
numbers = sorted(candidates)
```

Example:

```python
[10, 1, 2, 7, 6, 1, 5]
```

becomes:

```python
[1, 1, 2, 5, 6, 7, 10]
```

Sorting helps us:

1. Place duplicate values beside each other.
2. Skip duplicate choices.
3. Stop when a number exceeds `remaining`.
4. Produce combinations in a consistent order.
5. Avoid modifying the caller’s list.

---

# 6. The Most Important Duplicate Rule

Use:

```python
if index > start and numbers[index] == numbers[index - 1]:
    continue
```

The condition has two parts.

## `numbers[index] == numbers[index - 1]`

The current value equals the previous value.

## `index > start`

Both equal values are being considered at the same recursion level.

Therefore, choosing the second one would create a duplicate branch.

---

# 7. Why Not Skip Every Duplicate Globally?

Input:

```python
[1, 1, 6]
```

Target:

```text
8
```

We need this valid result:

```python
[1, 1, 6]
```

After selecting the first `1`, the second `1` must still be available at the next recursion level.

Therefore, this would be wrong:

```python
if numbers[index] == numbers[index - 1]:
    continue
```

It would skip duplicates at every level.

Correct:

```python
if index > start and numbers[index] == numbers[index - 1]:
    continue
```

This skips duplicate choices only among siblings at the same level.

---

# 8. How to Think Before Coding

### What is the current choice?

Choose one candidate from `start` onward.

### Can the same input position be reused?

No.

Move forward:

```python
index + 1
```

### Can another duplicate-value position be selected later?

Yes, at a deeper recursion level.

That allows:

```python
[1, 1, 6]
```

### How do we prevent duplicate branches?

Skip an equal candidate when it appears after `start` at the current level.

### When is an answer complete?

```python
remaining == 0
```

### When can we prune?

```python
number > remaining
```

---

# 9. Variables Required

```python
numbers    # sorted copy of candidates
result     # completed unique combinations
path       # current combination
start      # first unused position allowed
remaining  # amount still needed
```

A separate `used` list is unnecessary because `start` ensures we only move forward.

---

# 10. Algorithm

```text
1. Sort a copy of candidates.
2. Create result and path.
3. Start with start=0 and remaining=target.
4. If remaining is zero:
      save path.copy()
      return
5. Loop from start to the final index.
6. Skip the current value if:
      it equals the previous value
      and both are choices at the same recursion level
7. If the value is larger than remaining:
      break
8. Add the value to path.
9. Recurse with index + 1.
10. Remove the value from path.
11. Return result.
```

---

# 11. Dry Run — Duplicate Handling

Sorted input:

```python
[1, 1, 2, 5, 6, 7, 10]
```

At the root:

```text
start = 0
path = []
```

Choose the first `1` at index `0`:

```text
path = [1]
```

The next call starts at index `1`.

The second `1` may now be chosen because it is at a deeper level:

```text
path = [1, 1]
```

Choose `6`:

```text
path = [1, 1, 6]
remaining = 0
```

Save:

```python
[1, 1, 6]
```

Later, after returning to the root, the loop reaches the second `1` at index `1`.

At that level:

```python
index > start
numbers[index] == numbers[index - 1]
```

So we skip it.

Otherwise, it would recreate every combination already explored using the first root-level `1`.

---

# 12. Python Implementation

```python
def combination_sum_once(candidates, target):
    numbers = sorted(candidates)
    result = []
    path = []

    def backtrack(start, remaining):
        if remaining == 0:
            result.append(path.copy())
            return

        for index in range(start, len(numbers)):
            number = numbers[index]

            # Skip duplicate choices at this recursion level
            if index > start and number == numbers[index - 1]:
                continue

            # Current and later numbers are too large
            if number > remaining:
                break

            # Choose
            path.append(number)

            # Explore: move forward because this position is used
            backtrack(index + 1, remaining - number)

            # Undo
            path.pop()

    backtrack(0, target)
    return result


print(
    combination_sum_once(
        [10, 1, 2, 7, 6, 1, 5],
        8,
    )
)
```

Output:

```python
[
    [1, 1, 6],
    [1, 2, 5],
    [1, 7],
    [2, 6],
]
```

---

# 13. Build the Code Step by Step

Sort a copy:

```python
numbers = sorted(candidates)
```

Create storage:

```python
result = []
path = []
```

Write the successful base case:

```python
if remaining == 0:
    result.append(path.copy())
    return
```

Try unused positions:

```python
for index in range(start, len(numbers)):
```

Skip same-level duplicates:

```python
if index > start and numbers[index] == numbers[index - 1]:
    continue
```

Prune:

```python
if numbers[index] > remaining:
    break
```

Choose:

```python
path.append(numbers[index])
```

Move forward so the position cannot be reused:

```python
backtrack(index + 1, remaining - numbers[index])
```

Undo:

```python
path.pop()
```

---

# 14. Decision Tree Insight

At one recursion level:

```text
              []
        choose 1   skip duplicate 1
           |
          [1]
```

After entering the `[1]` branch:

```text
            [1]
       choose second 1
              |
           [1, 1]
```

The second `1` is:

- Skipped as a duplicate sibling at the root.
- Allowed as a child after the first `1` has been selected.

That distinction is the heart of Combination Sum II.

---

# 15. Why the Algorithm Works

The algorithm moves only forward through the sorted array.

Therefore:

- Each input position is used at most once.
- Combinations remain in nondecreasing order.
- Reordered duplicates cannot occur.
- Same-level duplicate branches are skipped.
- Duplicate values can still be used when they come from different positions.
- Every valid unique combination is explored.

---

# 16. Edge Cases

## Empty candidates

```python
combination_sum_once([], 8)
```

Output:

```python
[]
```

## Target is zero

```python
combination_sum_once([1, 2], 0)
```

Output:

```python
[[]]
```

## No solution

```python
combination_sum_once([4, 6], 5)
```

Output:

```python
[]
```

## Duplicate values form a valid answer

```python
combination_sum_once([1, 1, 2], 2)
```

Output:

```python
[
    [1, 1],
    [2],
]
```

## Same value appears many times

```python
combination_sum_once([1, 1, 1], 2)
```

Output:

```python
[[1, 1]]
```

It should not return the same combination multiple times.

---

# 17. Complexity

With `n` input positions, every position may be selected or not selected.

The search can therefore explore up to:

```text
2ⁿ branches
```

Copying completed paths adds output-related work.

A useful description is:

```text
Time: O(2ⁿ × n) in the worst case
Auxiliary recursion space: O(n)
```

Sorting costs:

```text
O(n log n)
```

The exponential search dominates for large inputs.

---

# 18. Common Mistakes

## Using the same index

Incorrect:

```python
backtrack(index, remaining - number)
```

This allows the same position to be reused.

Correct:

```python
backtrack(index + 1, remaining - number)
```

## Skipping every duplicate

Incorrect:

```python
if number == numbers[index - 1]:
    continue
```

This can prevent valid results such as `[1, 1, 6]`.

Correct:

```python
if index > start and number == numbers[index - 1]:
    continue
```

## Forgetting to sort

Duplicate values may not be adjacent, and `break` pruning becomes unsafe.

## Using a set to repair duplicate output

Generating duplicate branches and converting results to a set wastes work.

Prevent duplicate branches during recursion.

## Forgetting `path.pop()`

Selections leak into unrelated branches.

## Modifying the caller’s input

Prefer:

```python
numbers = sorted(candidates)
```

instead of:

```python
candidates.sort()
```

---

# 19. Real-World Assignment — One-Time Coupon Bundle Builder

A customer owns these one-time coupon cards:

```python
coupons = [1000, 100, 200, 700, 600, 100, 500]
target_discount = 800
```

Each coupon card may be used once.

The two `100` coupons are separate physical coupons.

Expected unique bundles:

```python
[
    [100, 100, 600],
    [100, 200, 500],
    [100, 700],
    [200, 600],
]
```

Do not return different orderings or duplicate bundles.

---

# 20. Institute-Level Assignment

Implement:

```python
def coupon_bundles(coupons, target_discount):
    pass
```

Requirements:

- Use backtracking.
- Each input position may be used once.
- Duplicate coupon values are allowed.
- Output combinations must be unique.
- Do not modify the original list.
- Explain same-level duplicate skipping.
- Include a decision-tree dry run.

Submit:

```text
day-27-combination-sum-ii/
├── assignment.py
├── test_assignment.py
└── README.md
```

Viva questions:

1. Why do we sort?
2. Why do we use `index + 1`?
3. What does `index > start` mean?
4. Why can two equal values sometimes both be used?
5. Why is a set-based solution less efficient?

---

# 21. Corporate-Level Requirements

Add:

- Type hints
- Docstring
- Input validation
- Unit tests
- Target limit
- Candidate-count limit
- Maximum-result limit
- Clear exceptions
- Integer currency representation
- No mutation of caller input

Suggested signature:

```python
def coupon_bundles(
    coupons: list[int],
    target_discount: int,
    max_results: int = 1000,
) -> list[list[int]]:
    """Return unique one-time coupon bundles matching the target."""
    pass
```

Validate:

```text
coupons must be a list
every coupon must be a positive integer
target must be a non-negative integer
duplicate values are allowed
each input position may be selected once
```

---

# 22. Client-Level Questions

Before implementation, ask:

1. Is every coupon a separate physical coupon?
2. Can one coupon be reused?
3. Are duplicate coupon values allowed?
4. Should equivalent value combinations appear once?
5. Must the discount equal the target or stay below it?
6. Can coupons have expiry dates?
7. Are some coupon types incompatible?
8. Is there a maximum number of coupons per bundle?
9. Do we need every bundle or only the best one?
10. What is the maximum input size?
11. What should happen when no bundle exists?
12. Should results be paginated?

---

# 23. Requirement-Change Examples

## Client says coupons can be reused

Change:

```python
backtrack(index + 1, ...)
```

to a Day 26-style reusable search:

```python
backtrack(index, ...)
```

## Client says order matters

The problem becomes closer to permutations rather than combinations.

## Client wants only the fewest coupons

Do not blindly generate every result. Consider optimization, pruning or dynamic programming.

## Client adds incompatible coupon pairs

Validate the current path before exploring deeper.

---

# 24. Acceptance Criteria

```text
Given [1000,100,200,700,600,100,500] and target 800,
four unique bundles are returned.

Each input position is used at most once.

[100,100,600] is allowed because two 100 coupons exist.

Equivalent reordered bundles appear only once.

Every returned bundle totals exactly 800.

The original input remains unchanged.

No-solution input returns [].
```

---

# 25. Testing Checklist

Test:

```python
coupon_bundles([], 800)
coupon_bundles([100], 100)
coupon_bundles([100], 200)
coupon_bundles([100, 100], 200)
coupon_bundles([100, 100, 100], 200)
coupon_bundles([100, 200, 500, 700], 800)
coupon_bundles([100, 100, 200, 500, 600, 700, 1000], 800)
coupon_bundles([0, 100], 100)
coupon_bundles([-100, 200], 100)
```

Verify correctness:

```python
for bundle in result:
    assert sum(bundle) == target_discount
```

Verify uniqueness:

```python
normalized = {tuple(bundle) for bundle in result}
assert len(normalized) == len(result)
```

Verify input remains unchanged:

```python
original = coupons.copy()
coupon_bundles(coupons, target_discount)
assert coupons == original
```

---

# 26. Git and Review Expectations

Suggested commits:

```text
feat: implement one-time coupon bundle search
test: cover duplicate coupons and edge cases
refactor: add limits and validation
docs: explain same-level duplicate handling
```

During review, verify:

- Is sorting performed on a copy?
- Is `index + 1` used?
- Are same-level duplicates skipped?
- Can different duplicate positions still be selected?
- Are results unique?
- Can recursion always make progress?
- Are production limits enforced?

---

# 27. Your Day 27 Assignment

Complete three levels:

### Level 1 — Algorithm

Implement `coupon_bundles()`.

### Level 2 — Production quality

Add validation, tests, type hints and limits.

### Level 3 — Client analysis

Document:

- Clarification questions
- Acceptance criteria
- Requirement-change impacts
- Why duplicate prevention belongs inside the search

## Day 27 takeaway

```text
Use once → index + 1

Duplicate at same level →
if index > start and numbers[index] == numbers[index - 1]:
    continue

Valid duplicate at a deeper level → allow it
```
