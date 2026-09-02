## Day 10 DSA — Stack + Valid Parentheses

Title:

```text
Stack + Valid Parentheses
```

## Feynman Idea

Imagine a stack of plates.

You can only:

```text
put a plate on top
remove the top plate
```

That is a stack.

In DSA terms:

```text
Last In, First Out
```

Short form:

```text
LIFO
```

If `(` opens first, it must be closed last.

Example:

```text
({[]})
```

Open brackets go into stack:

```text
(  {  [
```

Then closing brackets must match from the top:

```text
] matches [
} matches {
) matches (
```

## 80/20 Core

Remember this:

```text
When the latest opened thing must close first, use stack.
```

Common stack clues:

```text
parentheses
undo
back button
nested structure
matching pairs
```

## Problem: Valid Parentheses

Given a string containing:

```text
( ) { } [ ]
```

Return `True` if brackets are valid.

Valid means:

```text
1. Every opening bracket has a closing bracket
2. Closing bracket type must match
3. Brackets must close in correct order
```

Examples:

```python
"()"       → True
"()[]{}"   → True
"(]"       → False
"([)]"     → False
"{[]}"     → True
```

## Mental Model

Use:

```python
stack = []
```

When opening bracket comes:

```text
push into stack
```

When closing bracket comes:

```text
top of stack must be matching opening bracket
```

If not:

```text
False
```

At the end:

```text
stack must be empty
```

## Code

Create file:

```text
day_10_valid_parentheses.py
```

Write:

```python
def is_valid_parentheses(s):
    stack = []

    pairs = {
        ")": "(",
        "}": "{",
        "]": "["
    }

    for char in s:
        if char in "({[":
            stack.append(char)
        else:
            if not stack:
                return False

            top = stack.pop()

            if top != pairs[char]:
                return False

    return len(stack) == 0


print(is_valid_parentheses("()"))      # True
print(is_valid_parentheses("()[]{}"))  # True
print(is_valid_parentheses("(]"))      # False
print(is_valid_parentheses("([)]"))    # False
print(is_valid_parentheses("{[]}"))    # True
print(is_valid_parentheses("("))       # False
print(is_valid_parentheses("]"))       # False
```

## Dry Run

For:

```python
s = "{[]}"
```

Steps:

```text
char = {
stack = ["{"]

char = [
stack = ["{", "["]

char = ]
top = [
] matches [
stack = ["{"]

char = }
top = {
} matches {
stack = []

end
stack empty → True
```

For:

```python
s = "([)]"
```

Steps:

```text
char = (
stack = ["("]

char = [
stack = ["(", "["]

char = )
top = [
) does not match [
return False
```

## Complexity

```text
Time Complexity: O(n)
```

We scan each character once.

```text
Space Complexity: O(n)
```

Worst case all characters are opening brackets:

```python
"((((("
```

## Common Mistakes

Avoid these:

```text
1. Matching closing bracket with first opening bracket
2. Forgetting stack empty check before pop
3. Returning True without checking stack is empty
4. Treating ([)] as valid
5. Using queue instead of stack
```

## Practice Task

Solve:

```python
def is_valid_parentheses(s):
    pass
```

Test cases:

```python
print(is_valid_parentheses("()"))       # True
print(is_valid_parentheses("()[]{}"))   # True
print(is_valid_parentheses("(]"))       # False
print(is_valid_parentheses("([)]"))     # False
print(is_valid_parentheses("{[]}"))     # True
print(is_valid_parentheses("((()))"))   # True
print(is_valid_parentheses("(()"))      # False
```

Key rule:

```text
Latest opened bracket must close first → stack.
```
