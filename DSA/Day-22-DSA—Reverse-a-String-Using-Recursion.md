# Day 22 DSA — Reverse a String Using Recursion

## 1. Problem

Given a string, reverse it using recursion.

```python
text = "HELLO"
```

Expected output:

```text
OLLEH
```

---

## 2. Feynman explanation

Imagine letters standing in a line:

```text
H  E  L  L  O
```

The first and last letters exchange places:

```text
O  E  L  L  H
```

Then the second and second-last letters exchange places:

```text
O  L  L  E  H
```

When the two pointers meet in the middle, the work is complete.

Recursion handles one pair and asks a smaller version of the same problem to handle the inner pair.

---

## 3. The important 20%

A recursive two-pointer solution needs only three ideas:

```text
1. Swap the outside characters.
2. Move both pointers inward.
3. Stop when the pointers meet or cross.
```

Pointer movement:

```text
left  →              ←  right
```

Recursive pattern:

```text
swap(left, right)
swap(left + 1, right - 1)
```

---

## 4. How to think before coding

Ask four questions.

### What changes?

```text
left increases by 1
right decreases by 1
```

### What stays the same?

The same character list is used by every call.

### When should recursion stop?

When the pointers meet or cross:

```python
left >= right
```

### What work does each call perform?

It swaps one outside pair.

---

## 5. Why use a character list?

Python strings are immutable.

This means we cannot do this:

```python
text[0] = "O"
```

It produces an error.

Therefore, convert the string to a mutable list:

```python
characters = list(text)
```

After reversing it, join the characters:

```python
return "".join(characters)
```

---

## 6. Variables required

```python
characters  # mutable list of characters
left        # pointer from the beginning
right       # pointer from the end
```

---

## 7. Algorithm

```text
1. Convert the string into a character list.
2. Place left at index 0.
3. Place right at the final index.
4. If left >= right, stop.
5. Swap characters[left] and characters[right].
6. Recursively process left + 1 and right - 1.
7. Join the character list and return the result.
```

---

## 8. Dry run

Input:

```text
HELLO
```

Initial state:

```text
H  E  L  L  O
0  1  2  3  4

left = 0
right = 4
```

### Call 1

Swap indexes `0` and `4`:

```text
O  E  L  L  H
```

Continue with:

```text
left = 1
right = 3
```

### Call 2

Swap indexes `1` and `3`:

```text
O  L  L  E  H
```

Continue with:

```text
left = 2
right = 2
```

### Call 3

```text
left >= right
2 >= 2
```

Base case reached. Stop.

Final answer:

```text
OLLEH
```

---

## 9. Python implementation

```python
def reverse_string(text):
    characters = list(text)

    def reverse(left, right):
        # Base case: pointers met or crossed
        if left >= right:
            return

        # Swap the outside characters
        characters[left], characters[right] = (
            characters[right],
            characters[left],
        )

        # Reverse the smaller inner section
        reverse(left + 1, right - 1)

    reverse(0, len(characters) - 1)

    return "".join(characters)


print(reverse_string("HELLO"))
```

Output:

```text
OLLEH
```

---

## 10. Building the code step by step

Create a mutable character list:

```python
characters = list(text)
```

Create the recursive helper:

```python
def reverse(left, right):
```

Write the stopping condition:

```python
if left >= right:
    return
```

Swap the outside characters:

```python
characters[left], characters[right] = (
    characters[right],
    characters[left],
)
```

Move inward:

```python
reverse(left + 1, right - 1)
```

Start with the outside positions:

```python
reverse(0, len(characters) - 1)
```

Create the final string:

```python
return "".join(characters)
```

---

## 11. Why the algorithm works

Every recursive call correctly reverses one outside pair:

```text
first  ↔ last
second ↔ second-last
```

After swapping, the unresolved part becomes smaller:

```text
HELLO
 └─ELL─┘
   └L┘
```

Eventually, no pair remains to swap.

Therefore, the complete string is reversed.

---

## 12. Edge cases

### Empty string

```python
reverse_string("")
```

Result:

```text
""
```

Initial pointers are:

```text
left = 0
right = -1
```

The base case immediately stops recursion.

### One character

```python
reverse_string("A")
```

Result:

```text
"A"
```

Both pointers start at index `0`, so no swap is needed.

### Even number of characters

```python
reverse_string("CODE")
```

Result:

```text
"EDOC"
```

The pointers eventually cross.

---

## 13. Complexity

For a string containing `n` characters:

```text
Time:  O(n)
Space: O(n)
```

Explanation:

- Approximately `n / 2` swaps are performed.
- The character list requires `O(n)` space.
- Recursive calls use `O(n)` stack space in Big-O terms.
- Joining the list takes `O(n)` time.

---

## 14. Common mistakes

### Wrong base case

```python
if left == right:
    return
```

This works for odd-length strings but fails to stop correctly after pointers cross in even-length strings.

Use:

```python
if left >= right:
    return
```

### Moving both pointers in the same direction

Incorrect:

```python
reverse(left + 1, right + 1)
```

Correct:

```python
reverse(left + 1, right - 1)
```

### Trying to modify a Python string

Incorrect:

```python
text[left], text[right] = text[right], text[left]
```

Strings are immutable. Convert the string to a list first.

### Forgetting to start recursion

Defining the helper does not execute it.

You must call:

```python
reverse(0, len(characters) - 1)
```

---

## 15. Production reality

For ordinary Python application code, use slicing:

```python
reversed_text = text[::-1]
```

It is short, readable and implemented efficiently.

Use the recursive solution to learn:

- Base cases
- Two-pointer movement
- Smaller subproblems
- Recursive call-stack behavior

---

## 16. Practice problem

Complete this without using:

- `[::-1]`
- `reversed()`
- A loop

```python
def reverse_string(text):
    characters = list(text)

    def reverse(left, right):
        # Write the base case

        # Swap the characters

        # Make the recursive call
        pass

    reverse(0, len(characters) - 1)
    return "".join(characters)
```

Expected results:

```python
reverse_string("PYTHON")   # "NOHTYP"
reverse_string("A")        # "A"
reverse_string("")         # ""
reverse_string("12345")    # "54321"
```

## Day 22 takeaway

```text
Swap the outside pair.
Move both pointers inward.
Stop when they meet or cross.
```

Core recursive call:

```python
reverse(left + 1, right - 1)
```
