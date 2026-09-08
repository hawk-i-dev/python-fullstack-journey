Day 18 DSA — Recursion Basics + Factorial

Today’s concept is recursion. This is a foundation topic for trees, backtracking, DFS, dynamic programming, and divide-and-conquer.

## 1. What is recursion?

Recursion means a function calls itself to solve a smaller version of the same problem.

Simple idea:

```text
Big problem
→ smaller problem
→ smaller problem
→ smallest problem
→ return answers back
```

Example:

```text
factorial(5)
= 5 * factorial(4)
= 5 * 4 * factorial(3)
= 5 * 4 * 3 * factorial(2)
= 5 * 4 * 3 * 2 * factorial(1)
= 120
```

## 2. The two required parts

Every recursive solution needs:

1. Base case  
2. Recursive case

### Base case

The condition where recursion stops.

```python
if n == 0:
    return 1
```

Without a base case, recursion keeps calling itself forever until stack overflow / recursion error.

### Recursive case

The function calls itself with a smaller input.

```python
return n * factorial(n - 1)
```

## 3. Factorial problem

Factorial means:

```text
n! = n × (n-1) × (n-2) × ... × 1
```

Examples:

```text
5! = 5 × 4 × 3 × 2 × 1 = 120
4! = 4 × 3 × 2 × 1 = 24
1! = 1
0! = 1
```

## 4. Python solution

```python
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)
```

Test:

```python
print(factorial(5))  # 120
print(factorial(4))  # 24
print(factorial(0))  # 1
```

## 5. How the call stack works

For:

```python
factorial(5)
```

Calls go down:

```text
factorial(5)
factorial(4)
factorial(3)
factorial(2)
factorial(1)
factorial(0)
```

Then results come back up:

```text
factorial(0) = 1
factorial(1) = 1 * 1 = 1
factorial(2) = 2 * 1 = 2
factorial(3) = 3 * 2 = 6
factorial(4) = 4 * 6 = 24
factorial(5) = 5 * 24 = 120
```

## 6. Recursion mental model

Think of recursion like:

```text
I will solve the current problem
if someone gives me the answer for the smaller problem.
```

For factorial:

```text
factorial(5) says:
"I know the answer is 5 * factorial(4).
I trust factorial(4) to solve the smaller problem."
```

## 7. Iterative version

Same problem without recursion:

```python
def factorial_iterative(n):
    result = 1

    for i in range(2, n + 1):
        result *= i

    return result
```

## 8. Complexity

For `factorial(n)`:

Time:

```text
O(n)
```

Because we make `n` recursive calls.

Space:

```text
O(n)
```

Because each call waits on the call stack.

Iterative version:

Time:

```text
O(n)
```

Space:

```text
O(1)
```

## 9. Common mistakes

1. No base case

```python
def factorial(n):
    return n * factorial(n - 1)
```

This never stops.

2. Wrong recursive direction

```python
factorial(n + 1)
```

This moves away from the base case.

3. Forgetting `return`

Wrong:

```python
def factorial(n):
    if n == 0:
        return 1
    n * factorial(n - 1)
```

Correct:

```python
return n * factorial(n - 1)
```

4. Thinking recursion is always better

Not true. For factorial, iteration is usually more memory-efficient.

## 10. Practice problem

Write recursive functions for:

### Problem 1

```python
def factorial(n):
    pass
```

### Problem 2

```python
def sum_numbers(n):
    pass
```

Expected:

```python
sum_numbers(5)
# 5 + 4 + 3 + 2 + 1 = 15
```

### Problem 3

```python
def countdown(n):
    pass
```

Expected:

```text
5
4
3
2
1
Done
```

## 11. Interview sentence

Say this clearly:

“Recursion solves a problem by reducing it into smaller versions of the same problem. Every recursive function needs a base case to stop and a recursive case that moves toward the base case.”
