## Day 4 DSA — Two Pointers + Valid Palindrome

Title:

```text
Two Pointers + Valid Palindrome
```

Today’s pattern:

```text
Use two indexes: one from start, one from end.
Move them toward the middle.
```

## Concept: Two Pointers

Two pointers means we track two positions at the same time.

Example:

```python
s = "madam"

left = 0              # starts at m
right = len(s) - 1    # starts at m
```

Then compare:

```text
left char == right char?
```

If yes, move inward:

```python
left += 1
right -= 1
```

## Problem: Valid Palindrome

A palindrome reads the same forward and backward.

Examples:

```python
"madam" → True
"racecar" → True
"hello" → False
```

But in the real problem, we ignore:

```text
spaces
commas
colons
symbols
uppercase/lowercase difference
```

Example:

```python
"A man, a plan, a canal: Panama"
```

After ignoring symbols and case:

```text
amanaplanacanalpanama
```

Output:

```python
True
```

## Brute Force Approach

Clean the string first:

```python
cleaned = ""

for char in s:
    if char.isalnum():
        cleaned += char.lower()

return cleaned == cleaned[::-1]
```

This works, but it creates a new string.

Complexity:

```text
Time: O(n)
Space: O(n)
```

## Optimized Two Pointer Approach

We do not create a new string.

We directly compare characters from both sides.

## Code

Create file:

```text
day_04_valid_palindrome.py
```

Write:

```python
def is_palindrome(s):
    left = 0
    right = len(s) - 1

    while left < right:
        while left < right and not s[left].isalnum():
            left += 1

        while left < right and not s[right].isalnum():
            right -= 1

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True


print(is_palindrome("A man, a plan, a canal: Panama"))  # True
print(is_palindrome("race a car"))                      # False
print(is_palindrome("madam"))                           # True
print(is_palindrome(" "))                               # True
print(is_palindrome("0P"))                              # False
```

## Step-by-Step Logic

For:

```python
s = "race a car"
```

Compare:

```text
r vs r → same
a vs a → same
c vs c → same
e vs a → not same
```

So return:

```python
False
```

## Complexity

```text
Time Complexity: O(n)
```

Each character is visited at most once.

```text
Space Complexity: O(1)
```

We only use two variables:

```python
left
right
```

## Practice Task

Solve this without using:

```python
[::-1]
re
extra cleaned string
```

Function:

```python
def is_palindrome(s):
    pass
```

Test cases:

```python
print(is_palindrome("A man, a plan, a canal: Panama"))  # True
print(is_palindrome("race a car"))                      # False
print(is_palindrome("No lemon, no melon"))              # True
print(is_palindrome(""))                                # True
print(is_palindrome("ab"))                              # False
```

Expected explanation from you:

```text
Approach:
Time Complexity:
Space Complexity:
Code:
```

Key rule:

```text
Two pointers = compare from both ends and move inward.
```
