Day 20 DSA — Recursion: Sum of Array

Today we use recursion for a very practical problem: finding the sum of all numbers in an array.

From today, we’ll follow the implementation-ready format.

## 1. Problem in simple words

Given an array of numbers, return the total sum.

Example:

```python
nums = [2, 4, 6, 8]
```

Output:

```python
20
```

Because:

```text
2 + 4 + 6 + 8 = 20
```

## 2. 5-year-old explanation

Imagine you have 4 chocolate boxes:

```text
[2, 4, 6, 8]
```

You pick the first box: `2`.

Then you ask your friend:

```text
What is the total of the remaining boxes [4, 6, 8]?
```

Your friend says:

```text
18
```

So total is:

```text
2 + 18 = 20
```

That is recursion.

You solve one small part and ask recursion to solve the rest.

## 3. How to think before coding

Ask these questions:

```text
What is given?
An array nums

What should I return?
Sum of all numbers

What is the smallest case?
Empty array has sum 0

What changes each step?
Array becomes smaller, or index moves forward

What should I do at each step?
Take current number + sum of remaining numbers

When should I stop?
When index reaches end of array

What should I return?
0 at the end
```

## 4. Practical thinking

For:

```python
nums = [2, 4, 6, 8]
```

Think:

```text
sum_array(0)
= nums[0] + sum_array(1)

sum_array(1)
= nums[1] + sum_array(2)

sum_array(2)
= nums[2] + sum_array(3)

sum_array(3)
= nums[3] + sum_array(4)

sum_array(4)
= 0
```

Then return back:

```text
sum_array(3) = 8 + 0 = 8
sum_array(2) = 6 + 8 = 14
sum_array(1) = 4 + 14 = 18
sum_array(0) = 2 + 18 = 20
```

## 5. Variables we need

We need:

```python
nums
index
```

Why `index`?

Because instead of creating a new smaller array every time, we move the index forward.

Better:

```text
Use index
```

Avoid:

```text
nums[1:]
```

Because slicing creates new arrays and wastes memory.

## 6. Step-by-step algorithm

```text
1. Create helper(index)
2. If index == len(nums), return 0
3. Take current number nums[index]
4. Add it to helper(index + 1)
5. Return final answer
```

## 7. Python code

```python
def sum_array(nums):
    def helper(index):
        if index == len(nums):
            return 0

        return nums[index] + helper(index + 1)

    return helper(0)
```

Test:

```python
print(sum_array([2, 4, 6, 8]))  # 20
print(sum_array([1, 2, 3]))     # 6
print(sum_array([]))            # 0
```

## 8. Why this code works

For each index:

```python
return nums[index] + helper(index + 1)
```

Means:

```text
Current value + sum of remaining values
```

Base case:

```python
if index == len(nums):
    return 0
```

Means:

```text
No numbers left, so sum is 0
```

## 9. Dry run

For:

```python
nums = [2, 4, 6, 8]
```

Call stack:

```text
helper(0) returns 2 + helper(1)
helper(1) returns 4 + helper(2)
helper(2) returns 6 + helper(3)
helper(3) returns 8 + helper(4)
helper(4) returns 0
```

Returning:

```text
helper(3) = 8
helper(2) = 14
helper(1) = 18
helper(0) = 20
```

## 10. Complexity

Time:

```text
O(n)
```

Because each element is visited once.

Space:

```text
O(n)
```

Because recursive calls stay on the call stack.

## 11. Iterative version

In real production code, loop version is usually better for this problem:

```python
def sum_array_iterative(nums):
    total = 0

    for num in nums:
        total += num

    return total
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

## 12. Common mistakes

1. Forgetting base case

```python
def helper(index):
    return nums[index] + helper(index + 1)
```

This will go beyond array length.

2. Wrong base case

Wrong:

```python
if index == 0:
    return 0
```

Correct:

```python
if index == len(nums):
    return 0
```

3. Forgetting to move index

Wrong:

```python
helper(index)
```

Correct:

```python
helper(index + 1)
```

4. Using slicing repeatedly

Avoid:

```python
sum_array(nums[1:])
```

This creates new arrays.

5. Thinking recursion is best here

For array sum, loop is simpler and more memory-efficient.

## 13. Practice problem

Implement:

```python
def sum_array(nums):
    pass
```

Then test:

```python
print(sum_array([5, 10, 15]))  # 30
print(sum_array([1]))          # 1
print(sum_array([]))           # 0
```

Extra practice:

```python
def count_items(nums):
    pass
```

Expected:

```python
count_items([10, 20, 30])
# 3
```

## 14. Interview sentence

Say this clearly:

“To recursively sum an array, I process the current index and ask recursion to return the sum of the remaining indexes. The base case is when the index reaches the end of the array.”
