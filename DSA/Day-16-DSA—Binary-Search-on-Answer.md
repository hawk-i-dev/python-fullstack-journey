## Day 16 DSA — Binary Search on Answer

Title:

```text
Binary Search on Answer + Koko Eating Bananas
```

## Feynman Idea

Normal binary search searches inside an array.

Binary search on answer searches inside possible answers.

Example:

```text
What is the minimum speed needed?
```

You do not know the speed, but you know the answer is between:

```text
1 and max(piles)
```

So you try the middle speed.

If middle speed works, try smaller speed.

If middle speed fails, try bigger speed.

## 80/20 Core

Remember this:

```text
Use binary search on answer when you need minimum/maximum value and can test if a guess works.
```

Common clues:

```text
minimum possible
maximum possible
can finish within H hours
smallest speed
largest capacity
minimum days
```

## Problem: Koko Eating Bananas

Koko has banana piles.

```python
piles = [3, 6, 7, 11]
h = 8
```

Koko chooses an eating speed `k`.

If `k = 4`, each pile takes:

```text
ceil(3/4) = 1 hour
ceil(6/4) = 2 hours
ceil(7/4) = 2 hours
ceil(11/4) = 3 hours
```

Total:

```text
1 + 2 + 2 + 3 = 8 hours
```

So speed `4` works.

Question:

```text
What is the minimum speed k so Koko can finish within h hours?
```

Output:

```python
4
```

## Why Binary Search Fits

Possible speeds are sorted by behavior.

If speed `4` works:

```text
5, 6, 7, ... also work
```

If speed `3` fails:

```text
1, 2 also fail
```

So the answer space looks like:

```text
False False False True True True
```

We need the first `True`.

That is binary search.

## Helper Function: Can Finish?

For a guessed speed:

```python
speed = 4
```

Calculate total hours.

Formula for each pile:

```text
ceil(pile / speed)
```

In Python without importing `math`:

```python
(pile + speed - 1) // speed
```

Example:

```text
ceil(11 / 4) = (11 + 4 - 1) // 4 = 14 // 4 = 3
```

## Mental Model

Search range:

```python
left = 1
right = max(piles)
```

At each step:

```text
mid = guessed speed
```

Decision:

```text
If mid works:
    save answer
    try smaller speed
If mid fails:
    try bigger speed
```

## Code

Create file:

```text
day_16_koko_bananas.py
```

Write:

```python
def min_eating_speed(piles, h):
    left = 1
    right = max(piles)
    answer = right

    while left <= right:
        speed = (left + right) // 2

        hours = 0

        for pile in piles:
            hours += (pile + speed - 1) // speed

        if hours <= h:
            answer = speed
            right = speed - 1
        else:
            left = speed + 1

    return answer


print(min_eating_speed([3, 6, 7, 11], 8))      # 4
print(min_eating_speed([30, 11, 23, 4, 20], 5)) # 30
print(min_eating_speed([30, 11, 23, 4, 20], 6)) # 23
```

## Dry Run

For:

```python
piles = [3, 6, 7, 11]
h = 8
```

Search range:

```text
left = 1
right = 11
```

Try speed `6`:

```text
ceil(3/6)=1
ceil(6/6)=1
ceil(7/6)=2
ceil(11/6)=2
total = 6
```

6 hours <= 8, so speed works.

Try smaller:

```text
answer = 6
right = 5
```

Try speed `3`:

```text
ceil(3/3)=1
ceil(6/3)=2
ceil(7/3)=3
ceil(11/3)=4
total = 10
```

10 hours > 8, so speed fails.

Try bigger:

```text
left = 4
```

Try speed `4`:

```text
ceil(3/4)=1
ceil(6/4)=2
ceil(7/4)=2
ceil(11/4)=3
total = 8
```

Works.

```text
answer = 4
right = 3
```

Stop.

Return:

```python
4
```

## Complexity

Let:

```text
n = number of piles
m = max pile size
```

Time:

```text
O(n log m)
```

Because for every guessed speed, we scan all piles.

Space:

```text
O(1)
```

Only variables are used.

## Common Mistakes

Avoid these:

```text
1. Searching array indexes instead of answer values
2. Forgetting answer = speed when it works
3. Moving wrong side after success/failure
4. Using normal division instead of ceiling division
5. Thinking binary search needs an actual sorted array
```

## Practice Task

Solve:

```python
def min_eating_speed(piles, h):
    pass
```

Test cases:

```python
print(min_eating_speed([3, 6, 7, 11], 8))       # 4
print(min_eating_speed([30, 11, 23, 4, 20], 5)) # 30
print(min_eating_speed([30, 11, 23, 4, 20], 6)) # 23
print(min_eating_speed([1, 1, 1, 1], 4))        # 1
```

Key rule:

```text
Binary search on answer = guess value + check if guess works.
```
