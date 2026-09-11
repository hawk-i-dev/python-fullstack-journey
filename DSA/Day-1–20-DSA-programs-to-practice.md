Here are Day 1–20 DSA programs to practice.

## Day 1 — Contains Duplicate

Question: Return `True` if any number appears more than once.

```python
def contains_duplicate(nums):
    pass
```

Expected:

```python
contains_duplicate([1, 2, 3, 1]) # True
contains_duplicate([1, 2, 3, 4]) # False
```

## Day 2 — Two Sum

Question: Return indexes of two numbers whose sum equals target.

```python
def two_sum(nums, target):
    pass
```

Expected:

```python
two_sum([2, 7, 11, 15], 9) # [0, 1]
two_sum([3, 2, 4], 6)      # [1, 2]
```

## Day 3 — Valid Anagram

Question: Return `True` if two strings have the same characters with same frequency.

```python
def is_anagram(s, t):
    pass
```

Expected:

```python
is_anagram("listen", "silent") # True
is_anagram("hello", "world")   # False
```

## Day 4 — Valid Palindrome

Question: Return `True` if string reads same forward/backward after ignoring case, spaces, and punctuation.

```python
def is_palindrome(s):
    pass
```

Expected:

```python
is_palindrome("A man, a plan, a canal: Panama") # True
is_palindrome("race a car")                     # False
```

## Day 5 — Two Sum II

Question: Given sorted array, return 1-based positions of two numbers adding to target.

```python
def two_sum_sorted(numbers, target):
    pass
```

Expected:

```python
two_sum_sorted([2, 7, 11, 15], 9) # [1, 2]
two_sum_sorted([2, 3, 4], 6)      # [1, 3]
```

## Day 6 — Max Sum Subarray of Size K

Question: Return maximum sum of any continuous subarray of size `k`.

```python
def max_sum_subarray(nums, k):
    pass
```

Expected:

```python
max_sum_subarray([2, 1, 5, 1, 3, 2], 3) # 9
max_sum_subarray([1, 9, -1, -2, 7, 3], 3) # 10
```

## Day 7 — Longest Substring Without Repeating Characters

Question: Return length of longest substring without duplicate characters.

```python
def longest_unique_substring(s):
    pass
```

Expected:

```python
longest_unique_substring("abcabcbb") # 3
longest_unique_substring("bbbbb")    # 1
```

## Day 8 — Range Sum Query

Question: Return sum from index `left` to `right`.

```python
def range_sum(nums, left, right):
    pass
```

Expected:

```python
range_sum([1, 2, 3, 4, 5], 1, 3) # 9
range_sum([10, 20, 30], 0, 2)    # 60
```

## Day 9 — Subarray Sum Equals K

Question: Count continuous subarrays whose sum equals `k`.

```python
def subarray_sum(nums, k):
    pass
```

Expected:

```python
subarray_sum([1, 1, 1], 2) # 2
subarray_sum([1, 2, 3], 3) # 2
```

## Day 10 — Valid Parentheses

Question: Return `True` if brackets are valid and balanced.

```python
def is_valid_parentheses(s):
    pass
```

Expected:

```python
is_valid_parentheses("()[]{}") # True
is_valid_parentheses("(]")     # False
```

## Day 11 — Min Stack

Question: Build stack with `push`, `pop`, `top`, `get_min` in `O(1)`.

```python
class MinStack:
    def __init__(self):
        pass

    def push(self, val):
        pass

    def pop(self):
        pass

    def top(self):
        pass

    def get_min(self):
        pass
```

Expected:

```python
stack = MinStack()
stack.push(3)
stack.push(1)
stack.push(2)

stack.get_min() # 1
stack.pop()
stack.top()     # 1
stack.get_min() # 1
```

## Day 12 — Daily Temperatures

Question: Return how many days to wait for a warmer temperature.

```python
def daily_temperatures(temperatures):
    pass
```

Expected:

```python
daily_temperatures([73,74,75,71,69,72,76,73])
# [1,1,4,2,1,1,0,0]
```

## Day 13 — Number of Recent Calls

Question: Count requests in the last 3000 milliseconds.

```python
class RecentCounter:
    def __init__(self):
        pass

    def ping(self, t):
        pass
```

Expected:

```python
counter = RecentCounter()
counter.ping(1)    # 1
counter.ping(100)  # 2
counter.ping(3001) # 3
counter.ping(3002) # 3
```

## Day 14 — Reverse Linked List

Question: Reverse linked list and return new head.

```python
def reverse_list(head):
    pass
```

Expected:

```text
Input:  1 -> 2 -> 3 -> 4 -> None
Output: 4 -> 3 -> 2 -> 1 -> None
```

## Day 15 — Search Insert Position

Question: Return target index if found. Otherwise return insertion index.

```python
def search_insert(nums, target):
    pass
```

Expected:

```python
search_insert([1, 3, 5, 6], 5) # 2
search_insert([1, 3, 5, 6], 2) # 1
```

## Day 16 — Koko Eating Bananas

Question: Return minimum eating speed `k` to finish all piles within `h` hours.

```python
def min_eating_speed(piles, h):
    pass
```

Expected:

```python
min_eating_speed([3, 6, 7, 11], 8) # 4
min_eating_speed([30, 11, 23, 4, 20], 5) # 30
```

## Day 17 — Ship Packages Within D Days

Question: Return minimum ship capacity needed to ship all packages within given days.

```python
def ship_within_days(weights, days):
    pass
```

Expected:

```python
ship_within_days([1,2,3,4,5,6,7,8,9,10], 5) # 15
ship_within_days([3,2,2,4,1,4], 3) # 6
```

## Day 18 — Factorial

Question: Return factorial of `n` using recursion.

```python
def factorial(n):
    pass
```

Expected:

```python
factorial(5) # 120
factorial(0) # 1
```

## Day 19 — Fibonacci

Question: Return nth Fibonacci number.

```python
def fib(n):
    pass
```

Expected:

```python
fib(0) # 0
fib(1) # 1
fib(5) # 5
fib(6) # 8
```

## Day 20 — Sum of Array

Question: Return sum of all numbers using recursion.

```python
def sum_array(nums):
    pass
```

Expected:

```python
sum_array([2, 4, 6, 8]) # 20
sum_array([5, 10, 15])  # 30
sum_array([])           # 0
```

Practice rule:

```text
For each problem, write:
1. Input
2. Output
3. Variables needed
4. Stop/base condition
5. Step-by-step logic
6. Code
7. Test output
```
