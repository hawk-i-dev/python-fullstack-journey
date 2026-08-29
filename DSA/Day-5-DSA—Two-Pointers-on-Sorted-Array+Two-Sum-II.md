## Day 5 DSA — Two Pointers on Sorted Array + Two Sum II

Title:

```text
Sorted Array Two Pointers
```

## Feynman Explanation

Imagine numbers are arranged in increasing order:

```python
[2, 7, 11, 15]
```

You need two numbers whose sum is target.

Instead of checking every pair, stand at both ends:

```text
left  → smallest number
right → biggest number
```

Now check the sum.

```text
If sum is too small → move left forward
If sum is too big   → move right backward
If sum matches      → answer found
```

Why this works?

Because the array is sorted.

## Problem: Two Sum II

Given a sorted list of numbers, return indexes of two numbers whose sum equals target.

Example:

```python
numbers = [2, 7, 11, 15]
target = 9
```

Output:

```python
[0, 1]
```

Because:

```python
numbers[0] + numbers[1] = 2 + 7 = 9
```

## Important Difference From Day 2

Day 2 Two Sum:

```text
Unsorted array → use dictionary
Time: O(n)
Space: O(n)
```

Day 5 Two Sum II:

```text
Sorted array → use two pointers
Time: O(n)
Space: O(1)
```

## Mental Model

Start:

```python
left = 0
right = len(numbers) - 1
```

Calculate:

```python
current_sum = numbers[left] + numbers[right]
```

Decision:

```text
current_sum == target → return answer
current_sum < target  → need bigger sum → left += 1
current_sum > target  → need smaller sum → right -= 1
```

## Code

Create file:

```text
day_05_two_sum_sorted.py
```

Write:

```python
def two_sum_sorted(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left < right:
        current_sum = numbers[left] + numbers[right]

        if current_sum == target:
            return [left, right]

        if current_sum < target:
            left += 1
        else:
            right -= 1

    return []


print(two_sum_sorted([2, 7, 11, 15], 9))      # [0, 1]
print(two_sum_sorted([1, 2, 3, 4, 6], 6))     # [1, 3]
print(two_sum_sorted([1, 3, 4, 5, 7], 12))    # [3, 4]
print(two_sum_sorted([1, 2, 3], 10))          # []
```

## Dry Run

For:

```python
numbers = [1, 2, 3, 4, 6]
target = 6
```

Steps:

```text
left = 0 → 1
right = 4 → 6
sum = 7
7 is too big, move right leftward

left = 0 → 1
right = 3 → 4
sum = 5
5 is too small, move left forward

left = 1 → 2
right = 3 → 4
sum = 6
answer found → [1, 3]
```

## Complexity

```text
Time Complexity: O(n)
```

Because each pointer moves only forward/backward once.

```text
Space Complexity: O(1)
```

Because we only use variables:

```python
left
right
current_sum
```

## Common Mistakes

Avoid these:

```text
1. Using this method on unsorted array
2. Moving right when sum is too small
3. Moving left when sum is too big
4. Using left <= right instead of left < right
5. Returning numbers instead of indexes
```

## Practice Task

Solve without using dictionary and without nested loops:

```python
def two_sum_sorted(numbers, target):
    pass
```

Test cases:

```python
print(two_sum_sorted([2, 7, 11, 15], 9))      # [0, 1]
print(two_sum_sorted([1, 2, 3, 4, 6], 6))     # [1, 3]
print(two_sum_sorted([1, 3, 4, 5, 7], 12))    # [3, 4]
print(two_sum_sorted([1, 2, 3], 10))          # []
```

Key rule:

```text
Sorted array + pair sum = two pointers.
```
