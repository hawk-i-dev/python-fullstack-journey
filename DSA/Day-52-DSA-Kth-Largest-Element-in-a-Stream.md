# Day 52 DSA: Kth Largest Element in a Stream

Yesterday, you found the kth largest value in a complete list.

Today, values arrive **one at a time**, and you must update the answer after every arrival.

The new skill is **preserving useful state between calls**.

## 1. Understand it like a live leaderboard

Imagine a competition where scores keep arriving.

You want the **third-highest score so far** after every new score.

Initial scores:

```text
[4, 5, 8, 2]
```

The top three are:

```text
[4, 5, 8]
```

So the third largest is `4`.

Now score `10` arrives:

```text
Top three become [5, 8, 10]
Third largest becomes 5
```

We do not need to sort the entire history again.

We only need to update the three retained winners.

> **Keep the strongest `k` values seen so far. Return the weakest retained winner.**

## 2. What changes from Day 51?

| Day 51: Batch | Day 52: Stream |
|---|---|
| Receive a whole list | Receive values over time |
| Return one answer | Return an answer after each arrival |
| State exists during one function call | State survives between method calls |
| Function | Object with an `add()` method |

The heap idea stays the same. Its **lifetime** changes.

This is an important application-development concept: some operations depend on previously processed events.

## 3. Define the contract before coding

Build this class:

```python
tracker = KthLargest(k, initial_values)
tracker.add(value)
```

Our rules:

- `k` must be a positive integer.
- Scores are integers.
- Duplicate scores count separately.
- `add(value)` returns the kth largest score seen so far.
- If fewer than `k` scores have arrived, return `None`.
- Initial values may be empty.
- Do not modify the caller’s initial list.
- This is an **append-only, all-time** stream: no expiry, deletion, or correction.

Example:

```python
tracker = KthLargest(3)

tracker.add(10)  # None: only one score
tracker.add(7)   # None: only two scores
tracker.add(8)   # 7: now three scores
```

**Why `None`, not `0`?** Zero could be a legitimate score.

## 4. How to think through the design

Ask four questions.

### What must survive between arrivals?

```python
self.k
self.heap
```

- `k` tells us how many winners to retain.
- `heap` stores those winners.

### What can we safely forget?

Values smaller than the top `k`, once the heap is full.

Why?

There are already `k` values at least as large. Future arrivals cannot make that discarded value enter the all-time top `k`.

This reasoning depends on **old scores never disappearing or changing**.

### What should the root represent?

When the heap has `k` values:

```python
self.heap[0]
```

is the kth largest value.

Before then, it is merely the smallest score retained so far—not a valid kth-largest answer.

### What happens on every arrival?

```text
Space available?
    Push the score.

Heap full and score exceeds the root?
    Replace the root.

Otherwise?
    Ignore the score.

Fewer than k retained?
    Return None.

Otherwise?
    Return the root.
```

## 5. The invariant

> After each arrival, the heap contains the largest `min(k, scores_seen)` values, counting duplicates.

Two consequences:

1. The heap never contains more than `k` values.
2. Once full, its root is the answer.

You do not need to store every previous score.

## 6. Complete runnable Python

```python
import heapq


class KthLargest:
    def __init__(self, k, initial_values=()):
        if isinstance(k, bool) or not isinstance(k, int):
            raise TypeError("k must be an integer")

        if k < 1:
            raise ValueError("k must be positive")

        self.k = k
        self.heap = []

        for value in initial_values:
            self.add(value)

    def add(self, value):
        if isinstance(value, bool) or not isinstance(value, int):
            raise TypeError("score must be an integer")

        if len(self.heap) < self.k:
            heapq.heappush(self.heap, value)

        elif value > self.heap[0]:
            heapq.heapreplace(self.heap, value)

        if len(self.heap) < self.k:
            return None

        return self.heap[0]


tracker = KthLargest(3, [4, 5, 8, 2])

for score in [3, 5, 10, 9, 4]:
    print(tracker.add(score))
```

Output:

```text
4
5
5
8
8
```

The explicit checks reject booleans because Python otherwise treats `True` and `False` as integers.

### Why use `self.heap`?

A local variable created inside `add()` would disappear after the call.

```python
# Wrong for a streaming tracker:
def add(self, value):
    heap = []  # Starts over on every arrival!
```

The object must remember previous winners.

### Why use `initial_values=()`?

It provides an immutable empty default.

Each instance still creates its own separate mutable heap:

```python
self.heap = []
```

## 7. Dry run

Start with:

```text
k = 3
Initial scores = [4, 5, 8, 2]
Retained winners = [4, 5, 8]
```

The table shows retained values **sorted for explanation**. The actual heap array need not be fully sorted.

| New score | Decision | Retained winners | Return |
|---|---|---|---|
| `3` | Ignore: below `4` | `[4, 5, 8]` | `4` |
| `5` | Replace `4` | `[5, 5, 8]` | `5` |
| `10` | Replace one `5` | `[5, 8, 10]` | `5` |
| `9` | Replace `5` | `[8, 9, 10]` | `8` |
| `4` | Ignore: below `8` | `[8, 9, 10]` | `8` |

Notice that the two scores of `5` count separately.

## 8. Why ignoring a score is safe

Suppose the retained winners are:

```text
[8, 9, 10]
```

A score of `4` arrives.

There are already three scores greater than `4`. Adding more scores later cannot remove those three.

Therefore, `4` can never become the third-largest score in this append-only history.

But if score `10` could later be deleted, the argument would change.

**General lesson:**

> Discard information only when the problem’s rules prove you will never need it again.

## 9. A useful property: the answer cannot decrease

Once at least `k` scores exist, the kth-largest answer is **nondecreasing** in an append-only stream.

It may stay the same:

```text
Top three [8, 9, 10]
New score 4 → answer remains 8
```

Or increase:

```text
Top three [8, 9, 10]
New score 11 → top three [9, 10, 11]
Answer becomes 9
```

It cannot decrease because no existing score is removed.

This gives us an extra property to test—not just a few expected outputs.

## 10. Complexity

Let `k` be the retained heap size limit.

| Operation | Cost |
|---|---|
| Read the answer when full | `O(1)` |
| Ignore a score when full | `O(1)` |
| Push or replace | `O(log(k + 1))` |
| Stored state | `O(k)` |

For `m` initial scores, this constructor takes:

```text
O(m log(k + 1)) worst-case time
```

For `n` subsequent arrivals:

```text
O(n log(k + 1)) worst-case total time
```

The `log(k + 1)` form also handles `k = 1`, where each update is constant time.

## 11. Common mistakes

**Returning the root before the heap is full**

```text
k = 3
Scores seen = [10]
```

There is no third-largest value yet.

**Resetting the heap on every call**

This loses the stream’s history.

**Using a set**

This removes duplicates and changes the ranking problem.

**Replacing the root unconditionally**

A weaker incoming score could replace a stronger retained winner.

**Sharing a heap between objects**

Each tracker should own its own heap, not use a mutable class-level list.

**Assuming the tracker handles expired scores**

This implementation supports additions only.

## 12. Tests to run

Add these below the class:

```python
# Standard example
tracker = KthLargest(3, [4, 5, 8, 2])

assert [
    tracker.add(value)
    for value in [3, 5, 10, 9, 4]
] == [4, 5, 5, 8, 8]


# Not enough scores yet
tracker = KthLargest(3)
assert tracker.add(10) is None
assert tracker.add(7) is None
assert tracker.add(8) == 7


# Duplicates count
tracker = KthLargest(2)
assert tracker.add(5) is None
assert tracker.add(5) == 5
assert tracker.add(4) == 5


# Negative scores
tracker = KthLargest(2, [-5])
assert tracker.add(-1) == -5
assert tracker.add(-3) == -3


# k = 1 tracks the maximum
tracker = KthLargest(1)
assert tracker.add(-5) == -5
assert tracker.add(-10) == -5
assert tracker.add(2) == 2


# Separate instances keep separate state
first = KthLargest(1, [100])
second = KthLargest(1, [2])

assert second.add(3) == 3
assert first.add(50) == 100


# Input is unchanged
initial = [4, 5, 8, 2]
before = initial.copy()

tracker = KthLargest(3, initial)

assert initial == before

print("All tests passed!")
```

Also verify that `k=0` raises `ValueError` and a non-integer score raises `TypeError`.

## 13. Assignment

### Part A — Implement without copying

Build `KthLargest` with the same contract.

Before coding, write:

```text
Persistent state:
Invariant:
When can I discard a score?
When is the answer unavailable?
Time per update:
Space:
```

Use:

```python
k = 3
initial_values = [6, 2]
arrivals = [4, 8, 1, 10, 8]
```

Expected returns:

```python
[2, 4, 4, 6, 8]
```

Trace the retained winners after every arrival.

### Part B — Add a read-only method

Implement:

```python
tracker.peek()
```

Requirements:

- Return the current kth-largest value.
- Return `None` if fewer than `k` scores exist.
- Do not add or remove anything.
- Run in `O(1)` time.

Example:

```python
tracker = KthLargest(2, [10])

tracker.peek()    # None
tracker.add(7)    # 7
tracker.peek()    # 7
tracker.peek()    # 7: repeated reads change nothing
```

### Part C — Verify against a simple reference

For testing only, maintain the full history and compare:

```python
expected = (
    sorted(history, reverse=True)[k - 1]
    if len(history) >= k
    else None
)
```

The heap tracker’s result should match after every arrival.

**This is a valuable engineering technique:** use a simple, slower solution to check an optimized implementation.

## 14. Institute, corporate, and client-level thinking

| Setting | What to focus on |
|---|---|
| Institute/interview | Explain persistent state, the top-k invariant, and why a min-heap is used |
| Corporate | Define validation, instance isolation, concurrency, restart recovery, and duplicate-event handling |
| Client | Clarify all-time versus rolling-window rankings, ties, corrections, and whether results must survive restarts |

### Real-time issue: the same event arrives twice

Two different transactions with the same amount should count separately.

But the **same transaction delivered twice** may need to count only once.

Those are different situations.

The heap compares values; it does not detect repeated event IDs. If duplicate delivery is possible, deduplication belongs in the surrounding system.

### Real-time issue: the service restarts

This class stores its heap in memory. A restart loses that state.

An application must restore it from persisted state or rebuild it from an event history.

### Real-time issue: multiple requests update at once

This teaching class is not a concurrency-safe service by itself. Concurrent updates require appropriate synchronization or a single ordered processing path.

**The algorithm is one component of a reliable application—not the whole application.**

## 15. Remember this

> **Batch processing answers once. Streaming processing preserves state and updates the answer repeatedly.**

GitHub filename:

```text
Day-52-DSA-Kth-Largest-Element-in-a-Stream.md
```

**Explain-back checkpoint:** Why can we safely forget a low score in an all-time append-only stream, but not necessarily in a rolling one-hour leaderboard?
