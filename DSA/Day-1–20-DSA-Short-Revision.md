Day 1–20 DSA Short Revision

## Day 1 — Contains Duplicate

Pattern: Set

Core idea:

```text
If a number is already seen, duplicate exists.
```

Practice:

```python
contains_duplicate([1, 2, 3, 1]) # True
```

## Day 2 — Two Sum

Pattern: HashMap

Core idea:

```text
For every num, check if target - num was seen before.
```

Practice:

```python
two_sum([2, 7, 11, 15], 9) # [0, 1]
```

## Day 3 — Valid Anagram

Pattern: Frequency Map

Core idea:

```text
Two strings are anagrams if character counts match.
```

Practice:

```python
is_anagram("listen", "silent") # True
```

## Day 4 — Valid Palindrome

Pattern: Two Pointers

Core idea:

```text
Move left and right inward, compare valid characters.
```

Practice:

```python
is_palindrome("A man, a plan, a canal: Panama") # True
```

## Day 5 — Two Sum II

Pattern: Two Pointers on Sorted Array

Core idea:

```text
If sum too small, move left.
If sum too big, move right.
```

Practice:

```python
two_sum_sorted([2, 7, 11, 15], 9) # [1, 2]
```

## Day 6 — Max Sum Subarray of Size K

Pattern: Fixed Sliding Window

Core idea:

```text
Add new element, remove old element, update max.
```

Practice:

```python
max_sum_subarray([2, 1, 5, 1, 3, 2], 3) # 9
```

## Day 7 — Longest Unique Substring

Pattern: Variable Sliding Window

Core idea:

```text
Expand right, shrink left when duplicate appears.
```

Practice:

```python
longest_unique_substring("abcabcbb") # 3
```

## Day 8 — Range Sum Query

Pattern: Prefix Sum

Core idea:

```text
prefix[i] stores sum before index i.
range sum = prefix[right + 1] - prefix[left]
```

Practice:

```python
range_sum([1, 2, 3, 4, 5], 1, 3) # 9
```

## Day 9 — Subarray Sum Equals K

Pattern: Prefix Sum + HashMap

Core idea:

```text
If current_sum - k existed before, subarray exists.
```

Practice:

```python
subarray_sum([1, 1, 1], 2) # 2
```

## Day 10 — Valid Parentheses

Pattern: Stack

Core idea:

```text
Push opening brackets.
For closing bracket, top must match.
```

Practice:

```python
is_valid_parentheses("()[]{}") # True
```

## Day 11 — Min Stack

Pattern: Stack + Min Tracking

Core idea:

```text
Store value and current minimum together.
```

Practice:

```python
stack.get_min() # O(1)
```

## Day 12 — Daily Temperatures

Pattern: Monotonic Stack

Core idea:

```text
Keep indexes of unresolved colder days.
When warmer day comes, resolve previous days.
```

Practice:

```python
daily_temperatures([73,74,75,71,69,72,76,73])
# [1,1,4,2,1,1,0,0]
```

## Day 13 — RecentCounter

Pattern: Queue

Core idea:

```text
Keep only timestamps from last 3000 ms.
Remove older ones from front.
```

Practice:

```python
counter.ping(3002) # 3
```

## Day 14 — Reverse Linked List

Pattern: Linked List Pointers

Core idea:

```text
For each node:
save next
point current.next to previous
move previous and current forward
```

Practice:

```text
1 -> 2 -> 3 -> None
becomes
3 -> 2 -> 1 -> None
```

## Day 15 — Search Insert Position

Pattern: Binary Search

Core idea:

```text
Find target or position where target should be inserted.
```

Practice:

```python
search_insert([1, 3, 5, 6], 2) # 1
```

## Day 16 — Koko Eating Bananas

Pattern: Binary Search on Answer

Core idea:

```text
Search possible speeds.
If speed works, try smaller.
If speed fails, try bigger.
```

Practice:

```python
min_eating_speed([3, 6, 7, 11], 8) # 4
```

## Day 17 — Ship Packages Within D Days

Pattern: Binary Search on Answer

Core idea:

```text
Search possible ship capacity.
If capacity works, try smaller.
If capacity fails, try bigger.
```

Practice:

```python
ship_within_days([1,2,3,4,5,6,7,8,9,10], 5) # 15
```

## Day 18 — Factorial

Pattern: Basic Recursion

Core idea:

```text
factorial(n) = n * factorial(n - 1)
Base case: factorial(0) = 1
```

Practice:

```python
factorial(5) # 120
```

## Day 19 — Fibonacci

Pattern: Recursion + Memoization

Core idea:

```text
Naive recursion repeats work.
Memoization stores already solved answers.
```

Practice:

```python
fib(6) # 8
```

## Day 20 — Sum of Array

Pattern: Recursion with Index

Core idea:

```text
Current number + sum of remaining numbers.
Stop when index == len(nums).
```

Practice:

```python
sum_array([2, 4, 6, 8]) # 20
```

Final revision rule:

```text
Day 1–13: Data structure patterns
Day 14: Pointer manipulation
Day 15–17: Binary search patterns
Day 18–20: Recursion basics
```
