## Day 12 DSA — Monotonic Stack + Daily Temperatures

Title:

```text
Monotonic Stack + Daily Temperatures
```

## Feynman Idea

Imagine you write each day’s temperature on a card.

For each day, you want to know:

```text
How many days should I wait for a warmer temperature?
```

If you check future days one by one for every day, it becomes slow.

Instead, keep a stack of “waiting days”.

When a warmer day arrives, it answers previous colder days.

## 80/20 Core

Remember this:

```text
Use monotonic stack when previous items are waiting for a future greater/smaller value.
```

For Daily Temperatures:

```text
Stack stores colder days waiting for warmer future day.
```

## Problem: Daily Temperatures

Given daily temperatures, return how many days to wait until a warmer temperature.

If no warmer day exists, return `0`.

Example:

```python
temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
```

Output:

```python
[1, 1, 4, 2, 1, 1, 0, 0]
```

Explanation:

```text
Day 0: 73 waits 1 day for 74
Day 1: 74 waits 1 day for 75
Day 2: 75 waits 4 days for 76
Day 3: 71 waits 2 days for 72
Day 4: 69 waits 1 day for 72
Day 5: 72 waits 1 day for 76
Day 6: 76 has no warmer future day
Day 7: 73 has no warmer future day
```

## Why Brute Force Is Slow

Brute force:

```text
For each day:
    scan future days until warmer found
```

Worst case:

```text
Time: O(n²)
```

Example:

```python
[90, 80, 70, 60]
```

Each day searches but never finds warmer.

## Mental Model

Use stack to store indexes, not temperatures.

Why indexes?

Because answer needs distance:

```text
current_index - previous_index
```

Stack contains days waiting for warmer temperature.

When current temperature is warmer than stack top:

```text
pop previous day
answer[previous_day] = current_day - previous_day
```

## Code

Create file:

```text
day_12_daily_temperatures.py
```

Write:

```python
def daily_temperatures(temperatures):
    answer = [0] * len(temperatures)
    stack = []

    for current_day, current_temp in enumerate(temperatures):
        while stack and current_temp > temperatures[stack[-1]]:
            previous_day = stack.pop()
            answer[previous_day] = current_day - previous_day

        stack.append(current_day)

    return answer


print(daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]))
# [1, 1, 4, 2, 1, 1, 0, 0]

print(daily_temperatures([30, 40, 50, 60]))
# [1, 1, 1, 0]

print(daily_temperatures([30, 60, 90]))
# [1, 1, 0]

print(daily_temperatures([90, 80, 70, 60]))
# [0, 0, 0, 0]
```

## Dry Run

For:

```python
temperatures = [73, 74, 75, 71]
```

Start:

```text
answer = [0, 0, 0, 0]
stack = []
```

Day 0, temp 73:

```text
stack empty
push day 0
stack = [0]
```

Day 1, temp 74:

```text
74 > temp at day 0 = 73
pop day 0
answer[0] = 1 - 0 = 1
push day 1
stack = [1]
```

Day 2, temp 75:

```text
75 > temp at day 1 = 74
pop day 1
answer[1] = 2 - 1 = 1
push day 2
stack = [2]
```

Day 3, temp 71:

```text
71 is not warmer than 75
push day 3
stack = [2, 3]
```

Final partial answer:

```python
[1, 1, 0, 0]
```

Day 2 and day 3 still have no warmer day in this small example.

## Complexity

```text
Time Complexity: O(n)
```

Each index is pushed once and popped once.

```text
Space Complexity: O(n)
```

Stack and answer array use memory.

## Common Mistakes

Avoid these:

```text
1. Storing temperatures instead of indexes
2. Using if instead of while
3. Forgetting answer defaults to 0
4. Comparing with wrong stack item
5. Thinking nested while makes it O(n²)
```

Important:

```text
Even with while loop, total work is O(n)
because each index is popped only once.
```

## Practice Task

Solve:

```python
def daily_temperatures(temperatures):
    pass
```

Test cases:

```python
print(daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]))
# [1, 1, 4, 2, 1, 1, 0, 0]

print(daily_temperatures([30, 40, 50, 60]))
# [1, 1, 1, 0]

print(daily_temperatures([30, 60, 90]))
# [1, 1, 0]

print(daily_temperatures([90, 80, 70, 60]))
# [0, 0, 0, 0]
```

Key rule:

```text
Future warmer/greater answer = monotonic stack.
```
