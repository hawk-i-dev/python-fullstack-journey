# Day 23 DSA — Check a Palindrome Using Recursion

## 1. Problem

Determine whether a string reads the same forward and backward using recursion.

```python
text = "racecar"
```

Expected output:

```text
True
```

Another example:

```python
text = "python"
```

Expected output:

```text
False
```

---

## 2. Feynman explanation

Imagine two children checking a word:

```text
R A C E C A R
↑           ↑
left      right
```

They compare the outside letters:

```text
R == R
```

Because they match, they move inward:

```text
R A C E C A R
  ↑       ↑
```

They continue until:

- A pair does not match → `False`
- The pointers meet or cross → `True`

---

## 3. The important 20%

A recursive palindrome solution needs only three rules:

```text
1. If the outside characters differ, return False.
2. If the pointers meet or cross, return True.
3. Otherwise, check the smaller inner string.
```

Recursive call:

```python
check(left + 1, right - 1)
```

---

## 4. How to think before coding

### What changes?

```text
left moves right
right moves left
```

### What stays unchanged?

```text
The original string
```

### When does it return `False`?

```python
text[left] != text[right]
```

### When does it return `True`?

```python
left >= right
```

### What is the smaller problem?

Check whether the string between the current pointers is a palindrome.

---

## 5. Variables required

```python
text   # original string
left   # pointer starting from the beginning
right  # pointer starting from the end
```

We do not need to modify the string.

---

## 6. Algorithm

```text
1. Set left to 0.
2. Set right to the final index.
3. If left >= right, return True.
4. If text[left] != text[right], return False.
5. Recursively check left + 1 and right - 1.
```

Order matters:

```text
Base case
Mismatch check
Recursive call
```

---

## 7. Dry run — palindrome

Input:

```text
racecar
```

### Call 1

```text
r a c e c a r
↑           ↑

r == r
```

Continue:

```python
check(1, 5)
```

### Call 2

```text
r a c e c a r
  ↑       ↑

a == a
```

Continue:

```python
check(2, 4)
```

### Call 3

```text
r a c e c a r
    ↑   ↑

c == c
```

Continue:

```python
check(3, 3)
```

### Base case

```text
left == right
```

Only the middle character remains, so return:

```text
True
```

Final result:

```text
True
```

---

## 8. Dry run — not a palindrome

Input:

```text
hello
```

First comparison:

```text
h e l l o
↑       ↑

h != o
```

Return immediately:

```text
False
```

No remaining characters need to be checked.

---

## 9. Python implementation

```python
def is_palindrome(text):
    def check(left, right):
        # Base case: pointers met or crossed
        if left >= right:
            return True

        # Outside characters do not match
        if text[left] != text[right]:
            return False

        # Check the smaller inner section
        return check(left + 1, right - 1)

    return check(0, len(text) - 1)


print(is_palindrome("racecar"))
print(is_palindrome("hello"))
```

Output:

```text
True
False
```

---

## 10. Build the code from the algorithm

Create the function:

```python
def is_palindrome(text):
```

Create a recursive helper:

```python
def check(left, right):
```

Stop successfully when the pointers meet or cross:

```python
if left >= right:
    return True
```

Stop unsuccessfully when a pair differs:

```python
if text[left] != text[right]:
    return False
```

Check the smaller inner section:

```python
return check(left + 1, right - 1)
```

Start with the outside characters:

```python
return check(0, len(text) - 1)
```

---

## 11. Why it works

Each recursive call verifies one matching pair:

```text
first  == last
second == second-last
```

If any pair differs, the complete string cannot be a palindrome.

If every pair matches until the pointers meet or cross, the entire string is a palindrome.

---

## 12. Edge cases

### Empty string

```python
is_palindrome("")
```

The initial pointers are:

```text
left = 0
right = -1
```

Because `left >= right`, the result is:

```text
True
```

An empty string is considered a palindrome.

### One character

```python
is_palindrome("A")
```

Result:

```text
True
```

### Two equal characters

```python
is_palindrome("aa")
```

Result:

```text
True
```

### Two different characters

```python
is_palindrome("ab")
```

Result:

```text
False
```

### Case sensitivity

```python
is_palindrome("Racecar")
```

Result:

```text
False
```

Because:

```text
"R" != "r"
```

---

## 13. Ignore case and special characters

If the requirement is to treat this as a palindrome:

```text
A man, a plan, a canal: Panama
```

Normalize the text first:

```python
def is_clean_palindrome(text):
    cleaned = "".join(
        character.lower()
        for character in text
        if character.isalnum()
    )

    def check(left, right):
        if left >= right:
            return True

        if cleaned[left] != cleaned[right]:
            return False

        return check(left + 1, right - 1)

    return check(0, len(cleaned) - 1)


print(is_clean_palindrome("A man, a plan, a canal: Panama"))
```

Output:

```text
True
```

Always confirm the requirement before deciding whether to normalize the input.

---

## 14. Complexity

For a string of length `n`:

```text
Time:  O(n)
Space: O(n)
```

Why?

- At most half the characters are compared.
- Big-O removes the constant `1/2`, resulting in `O(n)`.
- Recursive calls consume call-stack space.

The normalized version also requires `O(n)` space for `cleaned`.

---

## 15. Common mistakes

### Returning `True` after one matching pair

Incorrect:

```python
if text[left] == text[right]:
    return True
```

One matching pair does not prove the entire string is a palindrome.

Correct:

```python
return check(left + 1, right - 1)
```

### Moving pointers incorrectly

Incorrect:

```python
return check(left + 1, right + 1)
```

Correct:

```python
return check(left + 1, right - 1)
```

### Forgetting to return the recursive result

Incorrect:

```python
check(left + 1, right - 1)
```

Correct:

```python
return check(left + 1, right - 1)
```

### Using the wrong base case

Incorrect:

```python
if left == right:
    return True
```

This does not handle even-length strings after the pointers cross.

Correct:

```python
if left >= right:
    return True
```

---

## 16. Production reality

Recursion is useful for learning, but an iterative two-pointer solution avoids call-stack growth:

```python
def is_palindrome_iterative(text):
    left = 0
    right = len(text) - 1

    while left < right:
        if text[left] != text[right]:
            return False

        left += 1
        right -= 1

    return True
```

Complexity:

```text
Time:  O(n)
Space: O(1)
```

Iteration is usually preferable for large production inputs.

---

## 17. Practice problem

Complete the recursive function:

```python
def is_palindrome(text):
    def check(left, right):
        # Write the successful base case

        # Check for a mismatch

        # Check the smaller inner string
        pass

    return check(0, len(text) - 1)
```

Expected results:

```python
is_palindrome("level")    # True
is_palindrome("python")   # False
is_palindrome("aa")       # True
is_palindrome("ab")       # False
is_palindrome("")         # True
is_palindrome("A")        # True
```

## Day 23 takeaway

```text
Pointers meet or cross → True
Outside characters differ → False
Otherwise → check the inner substring
```

Core recursive step:

```python
return check(left + 1, right - 1)
```
