# Day 52 DSA Quiz — Kth Largest Element in a Stream

15 MCQs • One correct answer per question.  
Focus on **persistent state, heap decisions, and real-world requirements**.

**1. What is the main difference between our Day 51 and Day 52 solutions?**

A. Streaming requires sorting after every arrival  
B. Streaming does not allow duplicates  
C. Streaming preserves state and updates the answer across calls  
D. Streaming uses a BST instead of a heap  

**2. Which information must each `KthLargest` object preserve?**

A. Only the most recent score  
B. `self.k` and `self.heap`  
C. A sorted copy of every score ever received  
D. Only the current maximum  

**3. Under our contract, what happens here?**

```python
tracker = KthLargest(3)
result = tracker.add(10)
```

A. `result` is `10`  
B. `result` is `0`  
C. `add()` raises `ValueError`  
D. `result` is `None`  

**4. Why is `None` better than `0` for representing “not enough scores yet”?**

A. Zero could be a legitimate kth-largest score  
B. Python heaps cannot contain zero  
C. Returning zero changes the heap  
D. `None` automatically counts missing scores  

**5. What does this print?**

```python
tracker = KthLargest(2)

print(tracker.add(5))
print(tracker.add(5))
print(tracker.add(4))
```

A. `None, None, 4`  
B. `5, 5, 4`  
C. `None, 5, 5`  
D. `None, 5, 4`  

**6. What is wrong with creating `heap = []` inside `add()` on every call?**

A. Empty lists cannot become heaps  
B. It discards the state from previous arrivals  
C. It prevents negative scores  
D. It automatically shares state between objects  

**7. A full size-3 heap retains `[4, 5, 8]`. What happens when `10` arrives?**

A. Replace `8`; return `4`  
B. Retain all four values; return `10`  
C. Ignore `10`; return `4`  
D. Replace `4`; return `5`  

**8. Why can we safely discard a low score in this append-only, all-time problem?**

A. There are already `k` scores at least as large, and future arrivals do not remove them  
B. Low scores can never appear twice  
C. The heap stores discarded values elsewhere  
D. Every future score must be larger  

**9. Once at least `k` scores exist, how can the kth-largest answer change as new scores arrive?**

A. It must increase after every arrival  
B. It must decrease after every arrival  
C. It can stay the same or increase, but cannot decrease  
D. It can change arbitrarily  

**10. What does this print?**

```python
tracker = KthLargest(3, [4, 5, 8, 2])

print(tracker.add(3))
print(tracker.add(5))
print(tracker.add(10))
```

A. `3, 5, 10`  
B. `4, 5, 5`  
C. `4, 4, 5`  
D. `5, 5, 8`  

**11. What are the worst-case update time and stored-state space for our tracker?**

A. Time `O(n log n)`, space `O(n)`  
B. Time `O(1)` for every update, space `O(1)`  
C. Time `O(k²)`, space `O(n)`  
D. Time `O(log(k + 1))`, space `O(k)`  

**12. What should a read-only `peek()` method do?**

A. Return the current answer or `None`, without changing the heap  
B. Remove and return the root  
C. Add a duplicate of the root  
D. Sort and clear the heap  

**13. Two separate transactions both have amount `500`. One of those transactions is accidentally delivered twice. What should a system that requires event deduplication do?**

A. Count every delivery because equal amounts always represent separate transactions  
B. Use a set of amounts, keeping only one `500`  
C. Count the two distinct transactions, but ignore the repeated delivery using its event ID  
D. Ignore every transaction with amount `500`  

**14. What happens if a service using only this in-memory tracker restarts?**

A. Python automatically restores the heap  
B. The state is lost unless restored from persistence or rebuilt from history  
C. The heap becomes a max-heap  
D. Only the smallest retained value survives  

**15. What should these calls return?**

```python
tracker = KthLargest(3, [6, 2])

results = [
    tracker.add(value)
    for value in [4, 8, 1, 10, 8]
]
```

A. `[4, 6, 1, 8, 8]`  
B. `[None, 2, 2, 4, 6]`  
C. `[2, 4, 1, 6, 8]`  
D. `[2, 4, 4, 6, 8]`  

Send your answers like:

```text
1.C
2.B
...
15.D
```

**Optional explanation:** Why might a previously discarded score become important if the requirement changes to a rolling one-hour leaderboard?
