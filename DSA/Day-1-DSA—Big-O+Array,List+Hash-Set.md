Day 1 DSA — Big-O + Arrays/List + Hash Set

## Today’s Concept: How to Think Like DSA

DSA is not only writing code. It is mainly:

1. Understand the input.
2. Find a simple solution first.
3. Improve time/space complexity.
4. Explain why your solution is better.

## Big-O Basics

Big-O tells how your code grows when input size grows.

Example: if list has `n` items:

| Complexity | Meaning | Example |
|---|---|---|
| O(1) | Constant time | Access one index |
| O(n) | Linear time | Loop through list once |
| O(n²) | Nested loops | Compare every item with every other item |
| O(log n) | Divide and reduce | Binary search |

For interviews, always say:

```text
Time Complexity: ...
Space Complexity: ...
```

## Data Structure Today: List + Set

Python list:

```python
nums = [1, 2, 3, 4]
```

Good for ordered data, but checking if something exists can take O(n).

Python set:

```python
seen = set()
```

Good for fast lookup. Checking if value exists is usually O(1).

Example:

```python
seen = set()

seen.add(10)

if 10 in seen:
    print("Found")
```

## Practice Problem: Contains Duplicate

Given a list of numbers, return `True` if any number appears more than once. Otherwise return `False`.

Examples:

```python
Input: [1, 2, 3, 1]
Output: True
```

```python
Input: [1, 2, 3, 4]
Output: False
```

```python
Input: []
Output: False
```

## Your Task

Create file:

```bash
mkdir dsa-practice
cd dsa-practice
code .
```

Create:

```text
day_01_contains_duplicate.py
```

Write this function:

```python
def contains_duplicate(nums):
    # your code here
    pass


print(contains_duplicate([1, 2, 3, 1]))  # True
print(contains_duplicate([1, 2, 3, 4]))  # False
print(contains_duplicate([]))            # False
print(contains_duplicate([5]))           # False
print(contains_duplicate([10, 20, 10]))  # True
```

Rules:

- Do not use `nums.count()`
- First think brute force
- Then solve using `set`

## Expected Explanation From You

After coding, send me:

```text
Approach:
Time Complexity:
Space Complexity:
Code:
```

Small hint: keep one `seen` set. While looping, if current number already exists in `seen`, duplicate found.
