## Day 15 DSA — Binary Search + Search Insert Position

Title:

```text
Binary Search + Search Insert Position
```

## Feynman Idea

Imagine a dictionary book.

If you want to find a word, you do not read every page from page 1.

You open the middle.

If your word should come after that page, ignore the left half.

If your word should come before that page, ignore the right half.

That is binary search.

## 80/20 Core

Remember this:

```text
Use binary search when the data is sorted and each check can remove half.
```

Clues:

```text
sorted array
find target
first/last position
insert position
minimum possible answer
maximum possible answer
```

## Problem: Search Insert Position

Given a sorted list and a target, return:

```text
index if target exists
correct insert index if target does not exist
```

Example 1:

```python
nums = [1, 3, 5, 6]
target = 5
```

Output:

```python
2
```

Because `5` is already at index `2`.

Example 2:

```python
nums = [1, 3, 5, 6]
target = 2
```

Output:

```python
1
```

Because `2` should be inserted before `3`.

## Why Binary Search Fits

The array is sorted:

```text
small values on left
large values on right
```

So when we check middle:

```text
middle too small → answer is on right side
middle too large → answer is on left side
```

Each step removes half of the search area.

## Mental Model

Use two pointers:

```python
left = 0
right = len(nums) - 1
```

Middle:

```python
mid = (left + right) // 2
```

Decision:

```text
nums[mid] == target → return mid
nums[mid] < target  → move left to mid + 1
nums[mid] > target  → move right to mid - 1
```

If target is not found:

```text
left becomes the correct insert position
```

## Code

Create file:

```text
day_15_search_insert_position.py
```

Write:

```python
def search_insert(nums, target):
    left = 0
    right = len(nums) - 1

    while left <= right:
        mid = (left + right) // 2

        if nums[mid] == target:
            return mid

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return left


print(search_insert([1, 3, 5, 6], 5))  # 2
print(search_insert([1, 3, 5, 6], 2))  # 1
print(search_insert([1, 3, 5, 6], 7))  # 4
print(search_insert([1, 3, 5, 6], 0))  # 0
print(search_insert([1], 1))           # 0
```

## Dry Run

For:

```python
nums = [1, 3, 5, 6]
target = 2
```

Start:

```text
left = 0
right = 3
```

Step 1:

```text
mid = (0 + 3) // 2 = 1
nums[mid] = 3
3 > 2
move right = mid - 1 = 0
```

Step 2:

```text
left = 0
right = 0
mid = 0
nums[mid] = 1
1 < 2
move left = mid + 1 = 1
```

Stop:

```text
left = 1
right = 0
left > right
```

Return:

```python
1
```

Because `2` should be inserted at index `1`.

## Complexity

```text
Time Complexity: O(log n)
```

Each step removes half of the remaining search space.

```text
Space Complexity: O(1)
```

Only variables are used.

## Common Mistakes

Avoid these:

```text
1. Using binary search on unsorted array
2. Using left < right incorrectly
3. Forgetting return left for insert position
4. Moving left/right wrongly
5. Infinite loop due to wrong pointer updates
```

## Practice Task

Solve:

```python
def search_insert(nums, target):
    pass
```

Test cases:

```python
print(search_insert([1, 3, 5, 6], 5))  # 2
print(search_insert([1, 3, 5, 6], 2))  # 1
print(search_insert([1, 3, 5, 6], 7))  # 4
print(search_insert([1, 3, 5, 6], 0))  # 0
print(search_insert([1, 3], 2))        # 1
```

Key rule:

```text
Sorted data + remove half each step = binary search.
```
