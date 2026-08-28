## Day 3 DSA — String Frequency Map + Valid Anagram

Title:

```text
String Frequency Map + Valid Anagram
```

Today’s main pattern:

```text
Use a dictionary to count how many times each character appears.
```

## Problem: Valid Anagram

Two strings are anagrams if they contain:

- Same characters
- Same frequency/count of each character
- Order can be different

Example:

```python
s = "anagram"
t = "nagaram"
```

Output:

```python
True
```

Because both have:

```text
a → 3
n → 1
g → 1
r → 1
m → 1
```

Another example:

```python
s = "rat"
t = "car"
```

Output:

```python
False
```

## First Thought: Sorting

If we sort both strings:

```python
sorted("anagram") == sorted("nagaram")
```

This works.

But complexity:

```text
Time: O(n log n)
Space: O(n)
```

Because sorting is not linear.

## Better Approach: Frequency Map

Use dictionary:

```python
count = {}
```

Store:

```text
character → frequency
```

Example:

```python
s = "anagram"
```

Frequency map becomes:

```python
{
    "a": 3,
    "n": 1,
    "g": 1,
    "r": 1,
    "m": 1
}
```

## Optimized Algorithm

Step-by-step:

```text
1. If lengths are different, return False
2. Count every character in s
3. Loop every character in t
4. If character is missing, return False
5. Subtract 1 from count
6. If count goes below 0, return False
7. End means both strings match, return True
```

## Code

Create file:

```text
day_03_valid_anagram.py
```

Write:

```python
def is_anagram(s, t):
    if len(s) != len(t):
        return False

    count = {}

    for char in s:
        count[char] = count.get(char, 0) + 1

    for char in t:
        if char not in count:
            return False

        count[char] -= 1

        if count[char] < 0:
            return False

    return True


print(is_anagram("anagram", "nagaram"))  # True
print(is_anagram("rat", "car"))          # False
print(is_anagram("listen", "silent"))    # True
print(is_anagram("a", "ab"))             # False
print(is_anagram("", ""))                # True
```

## Complexity

Let `n` be the length of the string.

```text
Time Complexity: O(n)
```

Because we loop through both strings once.

```text
Space Complexity: O(1)
```

If only lowercase English letters `a-z`, maximum 26 characters.

More generally:

```text
Space Complexity: O(k)
```

where `k` is the number of unique characters.

## Practice Problem

Solve without using:

```python
sorted()
collections.Counter
```

Function:

```python
def is_anagram(s, t):
    pass
```

Test cases:

```python
print(is_anagram("anagram", "nagaram"))  # True
print(is_anagram("rat", "car"))          # False
print(is_anagram("aacc", "ccac"))        # False
print(is_anagram("abc", "cba"))          # True
```

Expected explanation from you:

```text
Approach:
Time Complexity:
Space Complexity:
Code:
```
