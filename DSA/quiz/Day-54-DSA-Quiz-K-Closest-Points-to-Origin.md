# Day 54 DSA Quiz — K Closest Points to the Origin

15 MCQs • One correct answer per question.

Use our lesson’s rules: rank by **smaller squared distance**, then **smaller `x`**, then **smaller `y`**. Duplicate points count separately. The output is returned in rank order.

**1. What is the squared distance of `(-3, 4)` from the origin?**

A. `1`  
B. `5`  
C. `7`  
D. `25`

**2. Why can we skip the square root when ranking points?**

A. Taking the square root preserves the order of nonnegative numbers  
B. Squared distance and actual distance are always equal  
C. Squared distances eliminate ties  
D. Negative coordinates make square roots invalid  

**3. When the size-`k` heap is full, which point should be at its root?**

A. The closest point seen so far  
B. The most recently inserted point  
C. The worst-ranked point among those retained  
D. The point with the smallest `x`, regardless of distance  

**4. Which key makes Python’s min-heap expose the point we want to evict? Here, `d = x*x + y*y`.**

A. `(d, x, y)`  
B. `(-d, -x, -y)`  
C. `(-d, x, y)`  
D. `(d, -x, -y)`  

**5. For `k = 1`, which point wins between `(1, 2)` and `(1, -2)`?**

A. `(1, 2)`, because its `y` is larger  
B. Whichever appears first  
C. Neither; the function must return both  
D. `(1, -2)`, because distance and `x` tie, so the smaller `y` wins  

**6. What should happen here?**

```python
# The heap is full. Its root is the key for (3, 0).
heap[0] = (-9, -3, 0)  # Illustrates the current root
candidate = (-4, -2, 0)  # Key for (2, 0)
```

A. Ignore the candidate because `-4` is negative  
B. Replace the root because the candidate key is larger  
C. Ignore the candidate because the heap is already full  
D. Push the candidate and let the heap grow beyond `k`  

**7. What does this return?**

```python
k_closest([(3, 3), (1, 2), (-2, 1), (0, 1)], 2)
```

A. `[(0, 1), (-2, 1)]`  
B. `[(0, 1), (1, 2)]`  
C. `[(-2, 1), (0, 1)]`  
D. `[(1, 2), (3, 3)]`  

**8. If we use `(-d, x, y)` instead, what goes wrong?**

A. The farthest point no longer takes priority for eviction  
B. Points with negative coordinates cannot be inserted  
C. Distance ranking still works, but equal-distance points can be kept or evicted in the wrong order  
D. The heap starts removing duplicates automatically  

**9. What does this return?**

```python
k_closest([(2, 0), (0, 0), (0, 0)], 2)
```

A. `[(0, 0)]`  
B. `[(0, 0), (2, 0)]`  
C. `[(2, 0), (0, 0)]`  
D. `[(0, 0), (0, 0)]`  

**10. What does the heap contain after processing `m` points?**

A. The best `min(k, m)` points seen so far  
B. Exactly `k` points, even when `m < k`  
C. The first `k` points, regardless of distance  
D. Every point seen so far, in sorted order  

**11. Why do we sort the retained keys with `reverse=True` before decoding them?**

A. A min-heap stores all entries in ascending order  
B. Larger encoded keys represent better points, and the heap itself is not fully sorted  
C. Sorting converts squared distance to actual distance  
D. Sorting is needed to remove duplicate coordinates  

**12. Under our contract, what happens for these two calls?**

```python
k_closest([], 0)
k_closest([(2, 3)], 2)
```

A. Both return `[]`  
B. The first raises `ValueError`; the second returns `[(2, 3)]`  
C. The first returns `[]`; the second raises `ValueError`  
D. The first returns `None`; the second returns `[(2, 3)]`  

**13. For `n` input points and `k >= 1`, what are the worst-case time and auxiliary space bounds, including the final sort?**

A. Time `O(k log n)`, space `O(n)`  
B. Time `O(n²)`, space `O(1)`  
C. Time `O(log(k + 1))`, space `O(k)`  
D. Time `O(n log(k + 1))`, space `O(k)`  

**14. If the reference point changes to `(cx, cy)`, how should you calculate the ranking distance for `(x, y)`?**

A. `(x + cx)**2 + (y + cy)**2`  
B. `(x - cx)**2 + (y - cy)**2`  
C. `x*x + y*y`  
D. `(x - y)**2 + (cx - cy)**2`  

**15. A client says “nearest store” should mean shortest driving time. What should you do?**

A. Confirm how driving time is measured, then rank by that metric instead of squared straight-line distance  
B. Keep squared straight-line distance because it always gives the same ranking  
C. Rank stores by the smallest `x` coordinate  
D. Switch to a max-heap without changing the distance metric  

Send your choices as `1.<letter>, 2.<letter>, …, 15.<letter>`.

**Optional explain-back:** In Q5, write both heap keys. Which key is larger, and why does that point deserve to stay?
