## Day 8 DSA — Prefix Sum + Range Sum Query

Title:

```text
Prefix Sum + Range Sum Query
```

## Feynman Idea

Imagine you are tracking daily expenses.

```python
nums = [2, 4, 6, 8]
```

If someone asks:

```text
What is the total from index 1 to index 3?
```

Normal way:

```text
4 + 6 + 8 = 18
```

But if many people ask many range totals, recalculating again and again is waste.

So we prepare running totals once.

## 80/20 Core

Remember this:

```text
For repeated range-sum questions, build prefix sum once.
Then each range answer becomes O(1).
```

## Problem

Given an array, answer sum of numbers between `left` and `right`.

Example:

```python
nums = [2, 4, 6, 8, 10]
left = 1
right = 3
```

Answer:

```python
4 + 6 + 8 = 18
```

## Brute Force

```python
total = 0

for i in range(left, right + 1):
    total += nums[i]
```

Complexity per query:

```text
Time: O(n)
Space: O(1)
```

If there are many queries, this becomes slow.

## Prefix Sum Mental Model

Create a new array where each position stores total so far.

For:

```python
nums = [2, 4, 6, 8, 10]
```

Build:

```python
prefix = [0, 2, 6, 12, 20, 30]
```

Meaning:

```text
prefix[0] = sum before array starts = 0
prefix[1] = sum of nums[0]
prefix[2] = sum of nums[0..1]
prefix[3] = sum of nums[0..2]
```

Formula:

```python
prefix[i + 1] = prefix[i] + nums[i]
```

## Range Sum Formula

To get sum from `left` to `right`:

```python
prefix[right + 1] - prefix[left]
```

Example:

```python
nums = [2, 4, 6, 8, 10]
prefix = [0, 2, 6, 12, 20, 30]

left = 1
right = 3
```

Formula:

```python
prefix[4] - prefix[1]
20 - 2 = 18
```

Answer:

```python
18
```

## Code

Create file:

```text
day_08_prefix_sum.py
```

Write:

```python
def build_prefix(nums):
    prefix = [0]

    for num in nums:
        prefix.append(prefix[-1] + num)

    return prefix


def range_sum(prefix, left, right):
    return prefix[right + 1] - prefix[left]


nums = [2, 4, 6, 8, 10]
prefix = build_prefix(nums)

print(prefix)                    # [0, 2, 6, 12, 20, 30]
print(range_sum(prefix, 1, 3))    # 18
print(range_sum(prefix, 0, 2))    # 12
print(range_sum(prefix, 2, 4))    # 24
print(range_sum(prefix, 0, 4))    # 30
```

## Dry Run

For:

```python
nums = [2, 4, 6, 8, 10]
```

Build prefix:

```text
start: [0]

add 2  → [0, 2]
add 4  → [0, 2, 6]
add 6  → [0, 2, 6, 12]
add 8  → [0, 2, 6, 12, 20]
add 10 → [0, 2, 6, 12, 20, 30]
```

Query:

```text
sum index 1 to 3
= prefix[4] - prefix[1]
= 20 - 2
= 18
```

## Complexity

Building prefix:

```text
Time: O(n)
Space: O(n)
```

Each range query:

```text
Time: O(1)
Space: O(1)
```

## Common Mistakes

Avoid these:

```text
1. Forgetting prefix starts with 0
2. Using prefix[right] instead of prefix[right + 1]
3. Confusing inclusive right index
4. Recalculating sum for every query
5. Thinking prefix sum works only for positive numbers
```

Important:

```text
Prefix sum works with positive, zero, and negative numbers.
```

## Practice Task

Solve:

```python
def build_prefix(nums):
    pass


def range_sum(prefix, left, right):
    pass
```

Test cases:

```python
nums = [-2, 0, 3, -5, 2, -1]
prefix = build_prefix(nums)

print(range_sum(prefix, 0, 2))  # 1
print(range_sum(prefix, 2, 5))  # -1
print(range_sum(prefix, 0, 5))  # -3
```

Key rule:

```text
Range sum = prefix[right + 1] - prefix[left]
```
