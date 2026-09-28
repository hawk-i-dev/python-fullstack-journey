# Day 26 DSA — Backtracking: Combination Sum

## 1. Problem

Given a list of unique positive numbers and a target, find every combination whose sum equals the target.

A number may be used more than once.

```python
candidates = [2, 3, 6, 7]
target = 7
```

Expected output:

```python
[
    [2, 2, 3],
    [7],
]
```

The following should not appear separately:

```python
[2, 3, 2]
[3, 2, 2]
```

They contain the same numbers as `[2, 2, 3]`. In this problem, order does not create a new combination.

---

# 2. Feynman Explanation

Imagine you have coins worth:

```text
₹2  ₹3  ₹6  ₹7
```

You must fill a box with exactly:

```text
₹7
```

For each coin, you can:

```text
Use it
Use it again
Try the next coin
```

If the total becomes exactly ₹7, save the box.

If the total exceeds ₹7, stop exploring that path.

Possible answers:

```text
₹2 + ₹2 + ₹3 = ₹7
₹7 = ₹7
```

---

# 3. The Important 20%

Remember these four rules:

```text
remaining == 0 → save the answer
candidate > remaining → stop that branch
recurse with same index → reuse allowed
use start index → avoid reordered duplicates
```

Core pattern:

```python
for index in range(start, len(candidates)):
    number = candidates[index]

    path.append(number)
    backtrack(index, remaining - number)
    path.pop()
```

Notice this:

```python
backtrack(index, ...)
```

We use `index`, not `index + 1`, because the current number may be reused.

---

# 4. How to Recognize This Pattern

Combination Sum usually contains phrases such as:

```text
Find all combinations
Sum must equal target
A value may be used repeatedly
Order does not matter
Return every valid combination
```

That suggests:

```text
Backtracking + remaining target + start index
```

---

# 5. How to Think Before Coding

Ask these questions.

### What is the current choice?

Choose one candidate from `start` onward.

### What represents the current combination?

```python
path
```

### How much more do we need?

```python
remaining
```

### When is an answer complete?

```python
remaining == 0
```

### When is a branch impossible?

```python
candidate > remaining
```

### Can we reuse the selected number?

Yes, so recurse using the same index:

```python
backtrack(index, remaining - candidate)
```

### How do we avoid reordered duplicates?

Never move backward in the candidate list.

Use:

```python
for index in range(start, len(candidates)):
```

---

# 6. Variables Required

```python
result      # every valid completed combination
path        # combination currently being built
start       # first candidate index allowed
remaining   # amount still required
```

Example state:

```text
path = [2, 2]
remaining = 3
start = 0
```

---

# 7. Why Sort the Candidates?

Sorting enables early stopping.

```python
candidates = sorted(candidates)
```

If:

```text
remaining = 4
candidate = 6
```

then `6` is too large.

Because later candidates are even larger, we can stop the loop:

```python
if candidate > remaining:
    break
```

Using `break` is valid only because the candidates are sorted.

---

# 8. Algorithm

```text
1. Sort a copy of the candidates.
2. Create result and path.
3. Start with:
      start = 0
      remaining = target
4. If remaining == 0:
      save a copy of path
      return
5. Loop from start to the end.
6. If the current candidate is larger than remaining:
      stop the loop
7. Add the candidate to path.
8. Recursively reduce remaining.
9. Use the same index so the candidate can be reused.
10. Remove the candidate from path.
11. Return all valid combinations.
```

---

# 9. Dry Run

Input:

```python
candidates = [2, 3, 6, 7]
target = 7
```

Start:

```text
path = []
remaining = 7
```

Choose `2`:

```text
path = [2]
remaining = 5
```

Choose `2` again:

```text
path = [2, 2]
remaining = 3
```

Choose `2` again:

```text
path = [2, 2, 2]
remaining = 1
```

No candidate fits, so backtrack:

```text
path = [2, 2]
remaining = 3
```

Try `3`:

```text
path = [2, 2, 3]
remaining = 0
```

Save:

```python
[2, 2, 3]
```

Later, the root branch chooses `7`:

```text
path = [7]
remaining = 0
```

Save:

```python
[7]
```

Final result:

```python
[[2, 2, 3], [7]]
```

---

# 10. Python Implementation

```python
def combination_sum(candidates, target):
    numbers = sorted(candidates)
    result = []
    path = []

    def backtrack(start, remaining):
        # A valid combination is complete
        if remaining == 0:
            result.append(path.copy())
            return

        for index in range(start, len(numbers)):
            number = numbers[index]

            # This and every later number are too large
            if number > remaining:
                break

            # Choose
            path.append(number)

            # Explore: same index allows reuse
            backtrack(index, remaining - number)

            # Undo
            path.pop()

    backtrack(0, target)
    return result


print(combination_sum([2, 3, 6, 7], 7))
```

Output:

```text
[[2, 2, 3], [7]]
```

---

# 11. Build the Code Step by Step

Sort without changing the caller’s list:

```python
numbers = sorted(candidates)
```

Create storage:

```python
result = []
path = []
```

Create the recursive function:

```python
def backtrack(start, remaining):
```

Save a complete answer:

```python
if remaining == 0:
    result.append(path.copy())
    return
```

Try allowed candidates:

```python
for index in range(start, len(numbers)):
```

Prune impossible choices:

```python
if numbers[index] > remaining:
    break
```

Choose:

```python
path.append(numbers[index])
```

Explore while allowing reuse:

```python
backtrack(index, remaining - numbers[index])
```

Undo:

```python
path.pop()
```

Start the search:

```python
backtrack(0, target)
```

---

# 12. Why Use the Same Index?

After choosing `2`, it may be selected again:

```text
[2]
[2, 2]
[2, 2, 2]
```

Therefore:

```python
backtrack(index, remaining - number)
```

If each number could be used only once, we would use:

```python
backtrack(index + 1, remaining - number)
```

Remember:

```text
Reuse allowed     → index
Reuse not allowed → index + 1
```

---

# 13. Why Use a Start Index?

Without a start index, the algorithm may generate:

```python
[2, 2, 3]
[2, 3, 2]
[3, 2, 2]
```

The start index prevents the recursion from returning to earlier candidates.

This produces combinations, not permutations.

---

# 14. Why the Algorithm Works

At every step, the algorithm tries each allowed candidate.

After choosing a candidate:

- `remaining` becomes smaller.
- The same candidate remains available.
- Earlier candidates are no longer considered.
- Invalid branches are stopped.
- Valid paths are copied into `result`.

Therefore, every valid combination is found without generating different orderings of the same combination.

---

# 15. Edge Cases

### Target is zero

```python
combination_sum([2, 3], 0)
```

Output:

```python
[[]]
```

The empty combination has a sum of zero.

### No solution

```python
combination_sum([4, 6], 5)
```

Output:

```python
[]
```

### One exact candidate

```python
combination_sum([2, 7], 7)
```

Output:

```python
[[7]]
```

### Repeated use

```python
combination_sum([2], 6)
```

Output:

```python
[[2, 2, 2]]
```

---

# 16. Important Input Assumptions

The basic algorithm assumes:

```text
Candidates are positive integers.
Candidates are unique.
Target is non-negative.
```

Zero must not be accepted as a reusable candidate:

```python
[0, 2]
```

Choosing `0` does not reduce `remaining`, causing infinite recursion.

Negative reusable candidates can also prevent progress.

Production code must validate these assumptions.

---

# 17. Complexity

Combination Sum has exponential search growth.

If:

```text
n = number of candidates
T = target
m = smallest candidate
```

the maximum recursion depth is approximately:

```text
T / m
```

A practical description is:

```text
Time: exponential and output-dependent
Auxiliary space: O(T / m)
```

Copying and storing every valid combination requires additional output space.

Avoid claiming that this algorithm is simply `O(n)` or `O(n²)`.

---

# 18. Common Mistakes

### Using `index + 1` when reuse is allowed

Incorrect:

```python
backtrack(index + 1, remaining - number)
```

This prevents results such as:

```python
[2, 2, 3]
```

### Starting every call from zero

Incorrect:

```python
for index in range(0, len(numbers)):
```

This produces reordered duplicates.

### Forgetting to undo

Incorrect:

```python
path.append(number)
backtrack(index, remaining - number)
```

Correct:

```python
path.append(number)
backtrack(index, remaining - number)
path.pop()
```

### Using `break` without sorting

If candidates are unsorted, a later value might still fit.

Sort before using:

```python
if number > remaining:
    break
```

### Accepting zero or negative reusable values

They can cause infinite recursion or invalid search behavior.

---

# 19. Real-World Assignment — Budget Package Builder

A client has services with integer prices:

```python
prices = [200, 300, 600, 700]
budget = 700
```

Find every package whose total price is exactly `700`.

Expected combinations:

```python
[
    [200, 200, 300],
    [700],
]
```

For this assignment, the same service may be selected more than once.

---

# 20. Institute-Level Assignment

Implement:

```python
def build_packages(prices, budget):
    pass
```

Requirements:

- Use backtracking.
- Do not use a library that generates combinations.
- Prices are unique positive integers.
- Repeated use is allowed.
- Do not return reordered duplicates.
- Do not modify the original list.
- Explain the dry run.

Submit:

```text
day-26-combination-sum/
├── assignment.py
├── test_assignment.py
└── README.md
```

Be ready to explain:

1. Why do we need `remaining`?
2. Why is the same index passed recursively?
3. How does `start` prevent duplicates?
4. Why do we sort?
5. Why must reusable values be positive?

---

# 21. Corporate-Level Requirements

Upgrade the implementation with:

- Type hints
- Docstring
- Input validation
- Clear exceptions
- Unit tests
- Maximum budget protection
- Maximum result protection
- Logging only at the service boundary
- No modification of caller input

Suggested signature:

```python
def build_packages(
    prices: list[int],
    budget: int,
    max_results: int = 1000,
) -> list[list[int]]:
    """Return packages whose prices total exactly to budget."""
    pass
```

Validate:

```text
prices must be a list
every price must be a positive integer
prices must be unique
budget must be a non-negative integer
max_results must be positive
```

---

# 22. Client-Level Questions

Before implementation, ask:

1. Can the same service be selected repeatedly?
2. Does order matter?
3. Must the sum equal the budget or remain under it?
4. Are prices integers or decimal currency values?
5. Are duplicate prices possible?
6. Do we need every package or only the cheapest/best package?
7. What is the maximum number of services?
8. What is the maximum budget?
9. How many results may be returned?
10. Should results be paginated?
11. What should happen when there is no solution?
12. Are some combinations forbidden?

A small requirement change can produce a different algorithm.

---

# 23. Currency Warning

Never use normal floating-point values for exact financial calculations:

```python
0.1 + 0.2 != 0.3
```

Prefer integer minor units:

```text
₹2.00 → 200 paise
$2.00 → 200 cents
```

Or use Python’s `Decimal` when required.

For the assignment, use integers.

---

# 24. Acceptance Criteria

```text
Given [200, 300, 600, 700] and budget 700,
exactly [200, 200, 300] and [700] are returned.

No reordered duplicates are returned.

Every returned package totals exactly 700.

The original prices list remains unchanged.

Zero and negative prices are rejected.

No-solution input returns an empty list.
```

---

# 25. Testing Checklist

Test:

```python
build_packages([200, 300, 600, 700], 700)
build_packages([200], 600)
build_packages([400, 600], 500)
build_packages([], 700)
build_packages([200, 0], 700)
build_packages([200, -100], 700)
build_packages([200, 200], 700)
build_packages([200, 300], 0)
```

Also verify every result:

```python
for package in result:
    assert sum(package) == budget
```

Verify no duplicate combinations:

```python
normalized = {tuple(package) for package in result}
assert len(normalized) == len(result)
```

---

# 26. Git and Review Expectations

Suggested commits:

```text
feat: implement combination-sum package builder
test: cover valid and invalid budget scenarios
refactor: add validation and search limits
docs: document assumptions and client questions
```

During code review, check:

- Can recursion always make progress?
- Are duplicates prevented?
- Is caller input preserved?
- Are limits enforced?
- Are error messages clear?
- Are tests checking behavior rather than implementation details?

---

# 27. Your Day 26 Assignment

Complete three levels:

### Level 1 — Algorithm

Implement `build_packages()` using backtracking.

### Level 2 — Production quality

Add validation, type hints, tests and configurable limits.

### Level 3 — Requirement analysis

Write:

- Client clarification questions
- Acceptance criteria
- Production risks
- Explanation of how the design changes if reuse is forbidden

## Day 26 takeaway

```text
remaining == 0 → save
too large → prune
same index → reuse
start index → prevent reordered duplicates
append → explore → pop
```
