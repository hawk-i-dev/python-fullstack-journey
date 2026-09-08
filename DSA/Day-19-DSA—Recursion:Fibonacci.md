Day 19 DSA — Recursion: Fibonacci

Today we continue recursion with the Fibonacci sequence. This topic is important because it shows both the power and danger of recursion.

## 1. What is Fibonacci?

Fibonacci is a sequence where each number is the sum of the previous two numbers.

```text
0, 1, 1, 2, 3, 5, 8, 13, 21...
```

Formula:

```text
fib(n) = fib(n - 1) + fib(n - 2)
```

Base cases:

```text
fib(0) = 0
fib(1) = 1
```

Example:

```text
fib(5) = 5
```

Because:

```text
fib(5)
= fib(4) + fib(3)
= 3 + 2
= 5
```

## 2. Feynman explanation

Think of Fibonacci like asking two smaller questions:

```text
To find fib(5), ask:
What is fib(4)?
What is fib(3)?
Then add both answers.
```

So recursion branches into two calls.

That is the key difference from factorial.

Factorial:

```text
factorial(n) calls factorial(n - 1)
```

Fibonacci:

```text
fib(n) calls fib(n - 1) and fib(n - 2)
```

## 3. Simple recursive solution

```python
def fib(n):
    if n == 0:
        return 0

    if n == 1:
        return 1

    return fib(n - 1) + fib(n - 2)
```

Test:

```python
print(fib(0))  # 0
print(fib(1))  # 1
print(fib(5))  # 5
print(fib(6))  # 8
```

## 4. Why simple recursion is slow

For `fib(5)`:

```text
fib(5)
├── fib(4)
│   ├── fib(3)
│   └── fib(2)
└── fib(3)
    ├── fib(2)
    └── fib(1)
```

Notice `fib(3)` and `fib(2)` are calculated again and again.

This is called repeated work.

## 5. Complexity of simple recursion

Time:

```text
O(2^n)
```

Because every call can branch into two more calls.

Space:

```text
O(n)
```

Because the deepest call stack is roughly `n`.

## 6. Better solution: memoization

Memoization means:

```text
Remember already calculated answers.
```

If we already calculated `fib(4)`, don’t calculate it again.

Use dictionary:

```python
def fib(n, memo={}):
    if n in memo:
        return memo[n]

    if n == 0:
        return 0

    if n == 1:
        return 1

    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]
```

Better safer version:

```python
def fib(n, memo=None):
    if memo is None:
        memo = {}

    if n in memo:
        return memo[n]

    if n == 0:
        return 0

    if n == 1:
        return 1

    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]
```

## 7. Complexity with memoization

Time:

```text
O(n)
```

Each Fibonacci number is calculated once.

Space:

```text
O(n)
```

Dictionary + recursion stack.

## 8. Iterative solution

```python
def fib_iterative(n):
    if n == 0:
        return 0

    previous = 0
    current = 1

    for _ in range(2, n + 1):
        next_value = previous + current
        previous = current
        current = next_value

    return current
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

## 9. Common mistakes

1. Missing base cases

Fibonacci needs two base cases:

```python
fib(0) = 0
fib(1) = 1
```

2. Thinking recursion is always efficient

Simple Fibonacci recursion is very slow.

3. Using mutable default argument

Avoid this:

```python
def fib(n, memo={}):
```

Prefer:

```python
def fib(n, memo=None):
```

4. Confusing return value

`fib(5)` is `5`, not `8`.

Sequence by index:

```text
index: 0 1 2 3 4 5 6
value: 0 1 1 2 3 5 8
```

## 10. Practice problems

### Problem 1

```python
def fib(n):
    pass
```

Return nth Fibonacci number.

### Problem 2

```python
def fib_memo(n):
    pass
```

Use memoization.

### Problem 3

```python
def fib_iterative(n):
    pass
```

Use loop with `O(1)` space.

Test:

```python
print(fib(5))          # 5
print(fib_memo(10))    # 55
print(fib_iterative(6)) # 8
```

## 11. Interview sentence

Say this clearly:

“Naive recursive Fibonacci has overlapping subproblems, so it becomes exponential. Memoization stores already computed results and reduces the time complexity to O(n).”
