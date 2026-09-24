# Day 21 DSA — Find Maximum in an Array Using Recursion

## 1. Problem

Given an array of numbers, return its largest value using recursion.

```python
nums = [4, 7, 2, 9, 5]
```

Expected output:

```text
9
```

---

## 2. Feynman explanation

Imagine five children holding number cards.

The first child asks the remaining children:

> “What is the biggest number among you?”

The remaining children repeat the same question until only one child remains.

When only one number remains, that number is automatically the largest in that tiny group.

While returning, every number compares itself with the maximum found on its right.

```text
max(4, maximum of [7, 2, 9, 5])
max(7, maximum of [2, 9, 5])
max(2, maximum of [9, 5])
max(9, maximum of [5])
```

---

## 3. The important 20%

Every recursive solution needs:

1. A smaller version of the original problem.
2. A base case that stops recursion.
3. Work performed while returning.

For this problem:

```text
Base case:
If index reaches the final element,
return that element.

Recursive case:
Find the maximum in the remaining array.
Compare it with the current element.
```

Recurrence:

```text
maximum(nums, index)
    = max(nums[index], maximum(nums, index + 1))
```

---

## 4. How to think before coding

Ask these questions:

### What changes?

The `index` moves forward:

```text
0 → 1 → 2 → 3 → 4
```

### What stays unchanged?

The original array:

```python
nums
```

### When should recursion stop?

When `index` points to the final element:

```python
index == len(nums) - 1
```

### What should each call return?

The largest value from its current position to the end.

---

## 5. Variables required

```python
nums   # original array
index  # current position
```

No additional array is required.

---

## 6. Algorithm

```text
1. Reject an empty array.
2. Start at index 0.
3. If the index is at the final element:
      return that element.
4. Recursively find the maximum on the right.
5. Compare the current element with that result.
6. Return the larger value.
```

---

## 7. Dry run

Input:

```python
nums = [4, 7, 2, 9, 5]
```

Recursive calls:

```text
find_max(nums, 0)
find_max(nums, 1)
find_max(nums, 2)
find_max(nums, 3)
find_max(nums, 4)
```

At index `4`, the base case returns `5`.

Now recursion returns upward:

```text
index 4: return 5
index 3: max(9, 5) → 9
index 2: max(2, 9) → 9
index 1: max(7, 9) → 9
index 0: max(4, 9) → 9
```

Answer:

```text
9
```

---

## 8. Python implementation

```python
def find_max(nums, index=0):
    if not nums:
        raise ValueError("The array cannot be empty")

    # Base case: only the final element remains
    if index == len(nums) - 1:
        return nums[index]

    # Ask recursion for the maximum on the right
    right_maximum = find_max(nums, index + 1)

    # Compare the current number with the returned maximum
    return max(nums[index], right_maximum)


numbers = [4, 7, 2, 9, 5]
print(find_max(numbers))
```

Output:

```text
9
```

---

## 9. Build the code from the algorithm

First, write the function:

```python
def find_max(nums, index=0):
```

Protect against invalid input:

```python
if not nums:
    raise ValueError("The array cannot be empty")
```

Write the stopping condition:

```python
if index == len(nums) - 1:
    return nums[index]
```

Solve the smaller problem:

```python
right_maximum = find_max(nums, index + 1)
```

Combine the current value with the smaller answer:

```python
return max(nums[index], right_maximum)
```

---

## 10. Why it works

Each function call promises:

> “I will return the maximum value from my index to the end.”

The final call can keep this promise because only one element remains.

Every earlier call compares:

```text
current element vs maximum of remaining elements
```

Therefore, the first call returns the maximum of the entire array.

---

## 11. Complexity

For `n` numbers:

```text
Time:  O(n)
Space: O(n)
```

Why?

- Each element is visited once.
- Every recursive call occupies one call-stack frame.

An iterative solution also takes `O(n)` time but only `O(1)` extra space.

---

## 12. Production version

For a normal Python application, iteration is generally safer because a very large array can exceed Python’s recursion limit.

```python
def find_max_iterative(nums):
    if not nums:
        raise ValueError("The array cannot be empty")

    largest = nums[0]

    for number in nums[1:]:
        if number > largest:
            largest = number

    return largest
```

Use recursion here to learn recursive thinking. Prefer iteration for large production inputs.

---

## 13. Common mistakes

### Missing the base case

```python
return find_max(nums, index + 1)
```

This eventually attempts to access an invalid index.

### Using `0` as the base value

```python
if index == len(nums):
    return 0
```

This fails for an all-negative array:

```python
[-8, -3, -10]
```

It would incorrectly return `0`.

### Creating a new sliced array each time

```python
find_max(nums[1:])
```

This works, but repeatedly copying slices increases memory usage.

Passing an index is more efficient.

### Forgetting to return the recursive result

```python
find_max(nums, index + 1)
```

Without `return` or a comparison, the result is lost.

---

## 14. Practice problem

Implement this function without using Python’s built-in `max()`:

```python
def recursive_max(nums, index=0):
    pass
```

Requirements:

```python
recursive_max([3, 8, 2, 6])     # 8
recursive_max([-9, -2, -15])    # -2
recursive_max([7])              # 7
recursive_max([])               # raise ValueError
```

Replace this line:

```python
return max(nums[index], right_maximum)
```

with an `if` condition written by you.

## Day 21 takeaway

```text
Base case:
Return the final element.

Smaller problem:
Find the maximum from index + 1 onward.

Combine:
Compare the current value with the recursive result.
```
