## Day 9 DSA — Prefix Sum + Hash Map

Title:

```text
Prefix Sum + Hash Map — Subarray Sum Equals K
```

## Feynman Idea

Imagine you are walking and tracking total distance so far.

At one point:

```text
current_sum = 7
```

You want to know:

```text
Did I previously stand at total 5?
```

Because:

```text
7 - 5 = 2
```

That means the section between old point and current point has sum `2`.

Same idea in arrays:

```text
current_sum - old_prefix_sum = k
```

So:

```text
old_prefix_sum = current_sum - k
```

## 80/20 Core

Remember this one line:

```text
For subarray sum equals k, store old prefix sums and check:
needed = current_sum - k
```

If `needed` exists in map, then a subarray ending at current index has sum `k`.

## Problem

Given an integer array `nums` and integer `k`, count how many continuous subarrays have sum equal to `k`.

Example:

```python
nums = [1, 1, 1]
k = 2
```

Valid subarrays:

```text
[1, 1] at index 0..1
[1, 1] at index 1..2
```

Output:

```python
2
```

## Why Normal Sliding Window Is Not Enough

Sliding window works well when numbers are positive.

But this problem may contain:

```text
positive numbers
zero
negative numbers
```

Example:

```python
nums = [1, -1, 1]
k = 1
```

Because negatives can reduce the sum, simple “move left when sum is too big” logic fails.

So we use:

```text
prefix sum + hash map
```

## Key Equation

If:

```text
current_sum - old_sum = k
```

Then:

```text
old_sum = current_sum - k
```

So at every index:

```python
needed = current_sum - k
```

If `needed` appeared before, each occurrence creates one valid subarray.

## Why Store Counts?

Sometimes same prefix sum appears multiple times.

Example:

```text
prefix sum 0 may appear more than once
```

So map should store:

```text
prefix_sum → how many times seen
```

Not only:

```text
prefix_sum → True
```

## Code

Create file:

```text
day_09_subarray_sum_equals_k.py
```

Write:

```python
def subarray_sum(nums, k):
    prefix_count = {0: 1}
    current_sum = 0
    result = 0

    for num in nums:
        current_sum += num

        needed = current_sum - k

        if needed in prefix_count:
            result += prefix_count[needed]

        prefix_count[current_sum] = prefix_count.get(current_sum, 0) + 1

    return result


print(subarray_sum([1, 1, 1], 2))        # 2
print(subarray_sum([1, 2, 3], 3))        # 2
print(subarray_sum([1, -1, 1], 1))       # 3
print(subarray_sum([3, 4, 7, 2, -3, 1, 4, 2], 7))  # 4
```

## Dry Run

For:

```python
nums = [1, 1, 1]
k = 2
```

Start:

```python
prefix_count = {0: 1}
current_sum = 0
result = 0
```

Index 0:

```text
num = 1
current_sum = 1
needed = 1 - 2 = -1
-1 not found
save 1
prefix_count = {0: 1, 1: 1}
```

Index 1:

```text
num = 1
current_sum = 2
needed = 2 - 2 = 0
0 found once
result = 1
save 2
```

Index 2:

```text
num = 1
current_sum = 3
needed = 3 - 2 = 1
1 found once
result = 2
save 3
```

Final:

```python
2
```

## Complexity

```text
Time Complexity: O(n)
```

One pass through array.

```text
Space Complexity: O(n)
```

Hash map may store many prefix sums.

## Common Mistakes

Avoid these:

```text
1. Forgetting prefix_count = {0: 1}
2. Using set instead of count map
3. Saving current_sum before checking needed
4. Using normal sliding window when negatives exist
5. Confusing subarray with subsequence
```

Important order:

```text
1. update current_sum
2. check needed
3. update prefix_count
```

## Practice Task

Solve:

```python
def subarray_sum(nums, k):
    pass
```

Test cases:

```python
print(subarray_sum([1, 1, 1], 2))        # 2
print(subarray_sum([1, 2, 3], 3))        # 2
print(subarray_sum([1, -1, 1], 1))       # 3
print(subarray_sum([0, 0, 0], 0))        # 6
print(subarray_sum([3, 4, 7, 2, -3, 1, 4, 2], 7))  # 4
```

Key rule:

```text
Subarray sum equals k = prefix sum + count map.
```
