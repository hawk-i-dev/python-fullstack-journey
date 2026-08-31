## Day 6 DSA — Sliding Window + Maximum Sum Subarray

Title:

```text
Sliding Window + Max Sum Subarray
```

## Feynman Idea

Imagine a window covering `k` numbers.

Example:

```python
nums = [2, 1, 5, 1, 3, 2]
k = 3
```

First window:

```text
[2, 1, 5] 1 3 2
sum = 8
```

Next window should be:

```text
2 [1, 5, 1] 3 2
sum = 7
```

Instead of calculating again from zero:

```text
old sum = 8
remove left number = 2
add new right number = 1

new sum = 8 - 2 + 1 = 7
```

That is sliding window.

## 80/20 Core

Remember this:

```text
For fixed-size continuous subarray problems, use sliding window.
```

If question says:

```text
subarray
consecutive
continuous
size k
maximum/minimum sum
```

Think:

```text
Sliding Window
```

## Problem

Find the maximum sum of any `k` continuous numbers.

Example:

```python
nums = [2, 1, 5, 1, 3, 2]
k = 3
```

Possible windows:

```text
2 + 1 + 5 = 8
1 + 5 + 1 = 7
5 + 1 + 3 = 9
1 + 3 + 2 = 6
```

Answer:

```python
9
```

## Brute Force Approach

Check every window and calculate sum again.

```text
For every starting index:
    calculate next k numbers
```

Complexity:

```text
Time: O(n * k)
Space: O(1)
```

Problem: repeated work.

## Optimized Sliding Window

Steps:

```text
1. Find sum of first k numbers
2. Store it as max_sum
3. Move window one step
4. Remove outgoing left number
5. Add incoming right number
6. Update max_sum
```

## Code

Create file:

```text
day_06_max_sum_subarray.py
```

Write:

```python
def max_sum_subarray(nums, k):
    if k <= 0 or k > len(nums):
        return None

    window_sum = sum(nums[:k])
    max_sum = window_sum

    for right in range(k, len(nums)):
        outgoing = nums[right - k]
        incoming = nums[right]

        window_sum = window_sum - outgoing + incoming
        max_sum = max(max_sum, window_sum)

    return max_sum


print(max_sum_subarray([2, 1, 5, 1, 3, 2], 3))  # 9
print(max_sum_subarray([1, 2, 3, 4, 5], 2))     # 9
print(max_sum_subarray([5, 1, 2], 1))           # 5
print(max_sum_subarray([4, 2], 3))              # None
print(max_sum_subarray([-2, -1, -5], 2))        # -3
```

## Dry Run

For:

```python
nums = [2, 1, 5, 1, 3, 2]
k = 3
```

Start:

```text
window = [2, 1, 5]
window_sum = 8
max_sum = 8
```

Slide 1:

```text
remove 2
add 1
window_sum = 8 - 2 + 1 = 7
max_sum = 8
```

Slide 2:

```text
remove 1
add 3
window_sum = 7 - 1 + 3 = 9
max_sum = 9
```

Slide 3:

```text
remove 5
add 2
window_sum = 9 - 5 + 2 = 6
max_sum = 9
```

Final answer:

```python
9
```

## Complexity

```text
Time Complexity: O(n)
```

Each number enters and leaves the window once.

```text
Space Complexity: O(1)
```

We only use variables:

```python
window_sum
max_sum
right
outgoing
incoming
```

## Common Mistakes

Avoid these:

```text
1. Recalculating every window from zero
2. Using nested loops
3. Forgetting to subtract outgoing value
4. Forgetting to add incoming value
5. Setting max_sum = 0 when array has negative numbers
6. Not handling k > len(nums)
```

Important mistake:

```python
max_sum = 0
```

This is wrong if numbers are negative.

Correct:

```python
window_sum = sum(nums[:k])
max_sum = window_sum
```

## Practice Task

Solve:

```python
def max_sum_subarray(nums, k):
    pass
```

Test with:

```python
print(max_sum_subarray([2, 1, 5, 1, 3, 2], 3))  # 9
print(max_sum_subarray([1, 9, -1, -2, 7, 3], 3)) # 8
print(max_sum_subarray([4, 2, 1, 7, 8, 1, 2], 3)) # 16
```

Key rule:

```text
Fixed-size window = subtract left, add right.
```
