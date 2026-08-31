## Day 7 DSA — Variable Sliding Window

Title:

```text
Variable Sliding Window + Longest Substring Without Repeating Characters
```

## Feynman Idea

Day 6 window size was fixed:

```text
Always size k
```

Day 7 window size changes.

Imagine a rubber band window over a string:

```text
abcabcbb
```

You expand the right side to include new characters.

If duplicate comes, shrink from the left until the window becomes valid again.

## 80/20 Core

Remember this:

```text
If window becomes invalid, move left until it becomes valid again.
```

For this problem:

```text
Valid window = no repeated characters
```

## Problem

Given a string, find the length of the longest substring without repeating characters.

Important:

```text
Substring = continuous part of string
```

Example:

```python
s = "abcabcbb"
```

Answer:

```python
3
```

Because longest valid substrings are:

```text
"abc"
"bca"
"cab"
```

Length:

```python
3
```

## Mental Model

Use:

```python
left = 0
seen = set()
max_length = 0
```

Move `right` one step at a time.

If `s[right]` is already inside `seen`, duplicate found.

Then shrink from left:

```python
seen.remove(s[left])
left += 1
```

Continue until duplicate is removed.

## Code

Create file:

```text
day_07_longest_unique_substring.py
```

Write:

```python
def length_of_longest_substring(s):
    seen = set()
    left = 0
    max_length = 0

    for right in range(len(s)):
        while s[right] in seen:
            seen.remove(s[left])
            left += 1

        seen.add(s[right])

        window_length = right - left + 1
        max_length = max(max_length, window_length)

    return max_length


print(length_of_longest_substring("abcabcbb"))  # 3
print(length_of_longest_substring("bbbbb"))     # 1
print(length_of_longest_substring("pwwkew"))    # 3
print(length_of_longest_substring(""))          # 0
print(length_of_longest_substring("abcdef"))    # 6
```

## Dry Run

For:

```python
s = "abcabcbb"
```

Steps:

```text
a        valid, max = 1
ab       valid, max = 2
abc      valid, max = 3
abca     duplicate a, shrink left
bca      valid, max = 3
bcab     duplicate b, shrink left
cab      valid, max = 3
```

Final answer:

```python
3
```

## Complexity

```text
Time Complexity: O(n)
```

Each character enters and leaves the window at most once.

```text
Space Complexity: O(k)
```

`k` = number of unique characters inside the window.

## Common Mistakes

Avoid these:

```text
1. Confusing substring with subsequence
2. Moving left only once instead of using while
3. Forgetting to remove s[left] from set
4. Updating max_length before window is valid
5. Using nested loops unnecessarily
```

## Practice Task

Solve:

```python
def length_of_longest_substring(s):
    pass
```

Test cases:

```python
print(length_of_longest_substring("abcabcbb"))  # 3
print(length_of_longest_substring("bbbbb"))     # 1
print(length_of_longest_substring("pwwkew"))    # 3
print(length_of_longest_substring("dvdf"))      # 3
print(length_of_longest_substring("abba"))      # 2
```

Key rule:

```text
Variable window = expand right, shrink left when invalid.
```
