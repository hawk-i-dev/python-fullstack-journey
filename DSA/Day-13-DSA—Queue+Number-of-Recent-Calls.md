## Day 13 DSA — Queue + Number of Recent Calls

Title:

```text
Queue + Number of Recent Calls
```

## Feynman Idea

Imagine people standing in a line at a ticket counter.

The first person who enters the line is the first person who leaves.

That is a queue.

```text
FIFO = First In, First Out
```

For this problem, imagine API requests coming with timestamps.

At every new request time `t`, we only care about requests from:

```text
t - 3000 to t
```

Older requests should leave the queue.

## 80/20 Core

Remember this:

```text
Use queue when the oldest item must leave first.
```

For time-window problems:

```text
Keep valid recent items.
Remove expired old items from the front.
```

## Problem: Number of Recent Calls

Design a class that counts recent requests.

A request is recent if it happened in the last `3000` milliseconds.

For every `ping(t)`, return number of pings in:

```text
[t - 3000, t]
```

Example:

```python
RecentCounter()
ping(1)      → 1
ping(100)    → 2
ping(3001)   → 3
ping(3002)   → 3
```

Why last answer is `3`?

At `t = 3002`, valid range is:

```text
[2, 3002]
```

So `1` is too old and removed.

Remaining:

```text
100, 3001, 3002
```

Answer:

```text
3
```

## Why Queue Fits

We receive pings in increasing time order.

So the oldest timestamp is always at the front.

That means we can remove expired timestamps from the front.

Python list can work, but removing from front using:

```python
pop(0)
```

is slow because it shifts all elements.

So we use:

```python
collections.deque
```

Deque supports fast removal from both ends.

## Mental Model

At each ping:

```text
1. Add current timestamp to back
2. Remove timestamps older than t - 3000 from front
3. Queue size is the answer
```

Valid condition:

```text
timestamp >= t - 3000
```

Expired condition:

```text
timestamp < t - 3000
```

## Code

Create file:

```text
day_13_recent_counter.py
```

Write:

```python
from collections import deque


class RecentCounter:
    def __init__(self):
        self.requests = deque()

    def ping(self, t):
        self.requests.append(t)

        while self.requests[0] < t - 3000:
            self.requests.popleft()

        return len(self.requests)


counter = RecentCounter()

print(counter.ping(1))     # 1
print(counter.ping(100))   # 2
print(counter.ping(3001))  # 3
print(counter.ping(3002))  # 3
```

## Dry Run

Start:

```text
queue = []
```

Ping `1`:

```text
add 1
valid range = [-2999, 1]
queue = [1]
answer = 1
```

Ping `100`:

```text
add 100
valid range = [-2900, 100]
queue = [1, 100]
answer = 2
```

Ping `3001`:

```text
add 3001
valid range = [1, 3001]
queue = [1, 100, 3001]
answer = 3
```

Ping `3002`:

```text
add 3002
valid range = [2, 3002]
1 is expired because 1 < 2
remove 1
queue = [100, 3001, 3002]
answer = 3
```

## Complexity

For each `ping`:

```text
Average Time Complexity: O(1)
```

Why?

Each timestamp is added once and removed once.

Space:

```text
O(n)
```

where `n` is number of valid pings in the queue.

## Common Mistakes

Avoid these:

```text
1. Using list.pop(0) repeatedly
2. Removing timestamps <= t - 3000
3. Forgetting the range is inclusive
4. Returning total pings ever received
5. Not removing expired old pings
```

Important:

```text
[t - 3000, t] is inclusive
```

So if timestamp is exactly:

```text
t - 3000
```

it is still valid.

That is why we remove only:

```text
timestamp < t - 3000
```

Not:

```text
timestamp <= t - 3000
```

## Practice Task

Solve:

```python
class RecentCounter:
    pass
```

Expected methods:

```python
__init__()
ping(t)
```

Test:

```python
counter = RecentCounter()

print(counter.ping(1))     # 1
print(counter.ping(100))   # 2
print(counter.ping(3001))  # 3
print(counter.ping(3002))  # 3
print(counter.ping(7000))  # 1
```

Key rule:

```text
Queue = keep recent items, remove expired old items from front.
```
