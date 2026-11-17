# Day 51 DSA Quiz — Min-Heap & Kth Largest

15 MCQs • One correct answer per question.  
Think about **which values to retain and which value to remove**.

**1. Which two properties define a binary min-heap?**

A. Sorted values and equal subtree sizes  
B. BST ordering and balanced height  
C. Complete-tree shape and every parent ≤ its children  
D. Every node has either zero or two children  

**2. Why do we use a min-heap to retain the largest `k` values?**

A. It sorts the entire input automatically  
B. It exposes the smallest retained value—the candidate to replace  
C. It removes duplicate values  
D. Its root is the largest value overall  

**3. What does this return?**

```python
kth_largest([5, 5, 4], 2)
```

A. `4`  
B. `2`  
C. `ValueError`  
D. `5`  

**4. Which statement about a min-heap’s underlying list is guaranteed?**

A. `heap[0]` is a minimum value  
B. The entire list is sorted  
C. `heap[1]` is the second-smallest value  
D. The last element is the maximum value  

**5. The heap contains fewer than `k` values. What should our algorithm do with the next value?**

A. Ignore it if it is negative  
B. Replace the root immediately  
C. Push it into the heap  
D. Return the current root  

**6. A full size-3 heap contains the values `[4, 7, 9]`. The next value is `3`. What should happen?**

A. Replace `4` with `3`  
B. Ignore `3`  
C. Replace `9` with `3`  
D. Increase the heap size to four permanently  

**7. A full size-3 heap contains `[4, 7, 9]`. The next value is `8`. Which values should be retained?**

A. `[4, 7, 8]`  
B. `[4, 8, 9]`  
C. `[4, 7, 9, 8]`  
D. `[7, 8, 9]`  

**8. Why is the condition important here?**

```python
elif value > heap[0]:
    heapq.heapreplace(heap, value)
```

A. `heapreplace` always replaces the root, even if the incoming value is worse  
B. `heapreplace` only accepts positive integers  
C. The condition makes the heap a BST  
D. The condition prevents all duplicates from entering the heap  

**9. What is the correct invariant for our algorithm?**

A. The heap contains the smallest values seen so far  
B. The heap always contains exactly `k` values, even before enough input is processed  
C. The heap contains the largest `min(k, processed_count)` values seen so far  
D. The heap’s underlying list is always sorted descending  

**10. What is the result?**

```python
kth_largest([-5, -1, -3], 2)
```

A. `-5`  
B. `-3`  
C. `-1`  
D. `3`  

**11. After processing `[3, 2, 1, 5, 6, 4]` with `k = 3`, what does the root represent?**

A. The smallest value in the entire input: `1`  
B. The largest value in the entire input: `6`  
C. The number of retained values: `3`  
D. The smallest of the three largest values: `4`  

**12. Which complexity statement correctly includes the `k = 1` case?**

A. Time `O(n log(k + 1))`, auxiliary space `O(k)`  
B. Time `O(log n)`, auxiliary space `O(1)`  
C. Time `O(n²)`, auxiliary space `O(n)`  
D. Time `O(k)`, auxiliary space `O(n)`  

**13. Under our lesson’s contract, what should happen here?**

```python
kth_largest([10, 20], 3)
```

A. Return `10`  
B. Return `None`  
C. Raise `ValueError`  
D. Return `20`  

**14. To retain the `k` smallest values using a bounded heap, which retained value should be easiest to remove?**

A. The smallest, using a min-heap  
B. The largest, using a max-heap  
C. The earliest inserted value, using a queue  
D. An arbitrary value, using a set  

**15. A client changes “top three transactions of all time” to “top three transactions in the last hour.” Why is our simple bounded heap insufficient by itself?**

A. Heaps cannot store transaction amounts  
B. Time windows require sorting every transaction every second  
C. Duplicate amounts become invalid  
D. Old retained transactions can expire, and previously discarded candidates may become relevant  

Send your answers like:

```text
1.C
2.B
...
15.D
```

**Optional explanation:** For Q8 and Q15, explain the failure you would get if you ignored that issue.
