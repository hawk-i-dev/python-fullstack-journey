## Day 11 DSA — Stack + Min Stack

Title:

```text
Stack + Min Stack
```

## Feynman Idea

Imagine a stack of plates.

Normal stack can tell you only the top plate.

But now someone asks:

```text
What is the smallest plate number currently in the stack?
```

If you search the full stack every time, it becomes slow.

So we keep a second helper stack that remembers the minimum so far.

## 80/20 Core

Remember this:

```text
Use an extra stack to remember useful history.
```

For Min Stack:

```text
main_stack stores all values
min_stack stores current minimum values
```

Then `get_min()` becomes:

```text
O(1)
```

## Problem: Min Stack

Design a stack that supports:

```text
push(x)
pop()
top()
get_min()
```

All operations should work in:

```text
O(1)
```

Example:

```python
stack = MinStack()

stack.push(-2)
stack.push(0)
stack.push(-3)

stack.get_min()  # -3
stack.pop()
stack.top()      # 0
stack.get_min()  # -2
```

## Why Normal Stack Is Not Enough

Normal stack:

```python
stack = [-2, 0, -3]
```

To find minimum:

```python
min(stack)
```

This takes:

```text
O(n)
```

But requirement says:

```text
get_min() must be O(1)
```

So we need memory of minimums.

## Mental Model

Use two stacks:

```python
main_stack = []
min_stack = []
```

When pushing a value:

```text
Always push into main_stack
Push into min_stack only if value <= current minimum
```

Why `<=`?

Because duplicates matter.

Example:

```text
push 2
push 2
pop 2
```

Minimum should still be `2`.

## Code

Create file:

```text
day_11_min_stack.py
```

Write:

```python
class MinStack:
    def __init__(self):
        self.main_stack = []
        self.min_stack = []

    def push(self, value):
        self.main_stack.append(value)

        if not self.min_stack or value <= self.min_stack[-1]:
            self.min_stack.append(value)

    def pop(self):
        if not self.main_stack:
            return None

        value = self.main_stack.pop()

        if value == self.min_stack[-1]:
            self.min_stack.pop()

        return value

    def top(self):
        if not self.main_stack:
            return None

        return self.main_stack[-1]

    def get_min(self):
        if not self.min_stack:
            return None

        return self.min_stack[-1]


stack = MinStack()

stack.push(-2)
stack.push(0)
stack.push(-3)

print(stack.get_min())  # -3
print(stack.pop())      # -3
print(stack.top())      # 0
print(stack.get_min())  # -2
```

## Dry Run

Operations:

```text
push(-2)
```

```text
main_stack = [-2]
min_stack  = [-2]
```

```text
push(0)
```

```text
main_stack = [-2, 0]
min_stack  = [-2]
```

```text
push(-3)
```

```text
main_stack = [-2, 0, -3]
min_stack  = [-2, -3]
```

```text
get_min() = -3
```

```text
pop()
```

Popped value is `-3`, same as current min.

```text
main_stack = [-2, 0]
min_stack  = [-2]
```

```text
get_min() = -2
```

## Complexity

Each operation:

```text
push: O(1)
pop: O(1)
top: O(1)
get_min: O(1)
```

Space:

```text
O(n)
```

because we use extra stack.

## Common Mistakes

Avoid these:

```text
1. Calling min(stack) inside get_min()
2. Forgetting duplicate minimums
3. Using < instead of <= while pushing to min_stack
4. Forgetting to pop from min_stack
5. Not handling empty stack
```

## Practice Task

Solve:

```python
class MinStack:
    pass
```

Required methods:

```python
push(value)
pop()
top()
get_min()
```

Test:

```python
stack = MinStack()
stack.push(2)
stack.push(0)
stack.push(3)
stack.push(0)

print(stack.get_min())  # 0
stack.pop()
print(stack.get_min())  # 0
stack.pop()
print(stack.get_min())  # 0
stack.pop()
print(stack.get_min())  # 2
```

Key rule:

```text
Min Stack = main stack + helper min stack.
```
