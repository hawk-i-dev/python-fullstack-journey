## Day 2 DSA — Hash Map Pattern + Two Sum

Today’s main concept: using a `dict` to remember previous values so we avoid nested loops.

Yesterday we used `set` when we only cared whether a value exists.

Today we use `dict` when we need to store extra information, like:

```python
value -> index
```

Example:

```python
seen = {
    2: 0,
    7: 1
}
```

Means:

```text
value 2 is at index 0
value 7 is at index 1
```

## Problem: Two Sum

Given a list of numbers and a target, return the indexes of two numbers whose sum equals the target.

Example:

```python
nums = [2, 7, 11, 15]
target = 9
```

Answer:

```python
[0, 1]
```

Because:

```text
nums[0] + nums[1] = 2 + 7 = 9
```

## Brute Force Thinking

Check every pair:

```python
for i in range(len(nums)):
    for j in range(i + 1, len(nums)):
        if nums[i] + nums[j] == target:
            return [i, j]
```

Time Complexity:

```text
O(n²)
```

Because for every number, we again loop through remaining numbers.

## Optimized Thinking

For every number, ask:

```text
What number do I need to reach target?
```

Formula:

```python
needed = target - current_number
```

Example:

```python
nums = [2, 7, 11, 15]
target = 9
```

At index `0`:

```text
current = 2
needed = 9 - 2 = 7
```

If `7` already exists in dictionary, answer found.

If not, store `2`.

At index `1`:

```text
current = 7
needed = 9 - 7 = 2
```

`2` exists in dictionary, so return indexes.

## Practice File

Create:

```text
day_02_two_sum.py
```

Code template:

```python
def two_sum(nums, target):
    seen = {}

    for index, num in enumerate(nums):
        needed = target - num

        # check if needed number already exists
        # if yes, return [seen[needed], index]

        # otherwise store current number with its index

    return []


print(two_sum([2, 7, 11, 15], 9))      # [0, 1]
print(two_sum([3, 2, 4], 6))           # [1, 2]
print(two_sum([3, 3], 6))              # [0, 1]
print(two_sum([1, 2, 3], 10))          # []
```

## Your Target

Solve using dictionary, not nested loops.

Expected final explanation:

```text
Approach:
For each number, calculate needed = target - num.
If needed is already in dictionary, return indexes.
Otherwise store current number and index.

Time Complexity: O(n)
Space Complexity: O(n)
```

Key concept to remember:

```text
set = fast existence check
dict = fast existence check + store useful data
```
