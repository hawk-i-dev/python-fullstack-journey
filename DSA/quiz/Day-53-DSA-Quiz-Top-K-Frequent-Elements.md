# Day 53 DSA Quiz — Top K Frequent Elements

15 MCQs • One correct answer per question.

**Our ranking rule:** Higher frequency first; when frequencies tie, the smaller value wins.

**1. What does “Top K Frequent Elements” rank primarily?**

A. Numeric size  
B. Frequency of occurrence  
C. Position in the input  
D. Number of digits  

**2. What should this return?**

```python
top_k_frequent([4, 4, 4, 7, 7, 9], 2)
```

A. `[9, 7]`  
B. `[4, 4]`  
C. `[7, 4]`  
D. `[4, 7]`  

**3. Why do we build a frequency map before processing heap candidates?**

A. Each distinct candidate needs its final occurrence count  
B. Heaps cannot contain negative numbers  
C. A frequency map sorts values automatically  
D. Counting removes all repeated events without recording them  

**4. What should this return under our tie rule?**

```python
top_k_frequent([8, 8, 3, 3, 5], 1)
```

A. `[8]`  
B. `[5]`  
C. `[3]`  
D. `[8, 3]`  

**5. Which key do we use in the bounded min-heap?**

A. `(value, frequency)`  
B. `(frequency, -value)`  
C. `(-frequency, value)`  
D. `(frequency, value)`  

**6. Why is `(2, -8)` considered weaker than `(2, -3)`?**

A. Frequencies tie, and value `8` loses to the smaller value `3`  
B. Value `8` has fewer occurrences  
C. Negative keys cannot stay in a heap  
D. Larger original values always win ties  

**7. Once the heap is full, what should its root represent?**

A. The strongest retained candidate  
B. The smallest input value regardless of frequency  
C. The most recently processed candidate  
D. The weakest retained candidate under our ranking rule  

**8. A full size-2 heap contains keys `(2, -8)` and `(2, -3)`. Candidate `(3, -5)` arrives. What happens?**

A. Ignore it because `5 > 3`  
B. Remove `(2, -3)`  
C. Replace `(2, -8)`  
D. Keep all three candidates permanently  

**9. Why is `(frequency, value)` the wrong key for our particular tie rule?**

A. Python cannot compare tuples containing positive integers  
B. It makes the smaller tied value easier to remove  
C. It ignores frequency completely  
D. It makes every heap operation linear  

**10. What does this return?**

```python
top_k_frequent([-1, -1, -2, -2, 5], 2)
```

A. `[5, -1]`  
B. `[-1, -2]`  
C. `[-2, 5]`  
D. `[-2, -1]`  

**11. Which limit must a valid `k` satisfy under our contract?**

A. `0 <= k <= number_of_distinct_values`  
B. `1 <= k <= len(nums)`  
C. `k` must equal the highest frequency  
D. `k` must be smaller than every input value  

**12. Why do we sort the retained heap entries before returning the result?**

A. Sorting is required before every heap insertion  
B. Sorting creates the frequency map  
C. A heap is not fully sorted, but our output has a defined ranking order  
D. Sorting removes all frequency ties  

**13. Let `u` be the number of distinct values. Why is total auxiliary space `O(u)`, rather than just `O(k)`?**

A. Every input occurrence is stored in the heap  
B. The frequency map stores a count for each distinct value  
C. Each heap operation creates a full copy of the input  
D. Sorting always requires quadratic space  

**14. What is the expected output?**

```python
nums = [6, 6, 2, 2, 2, 9, 9, 4, 4]
top_k_frequent(nums, 3)
```

A. `[2, 9, 6]`  
B. `[4, 6, 9]`  
C. `[2, 2, 2]`  
D. `[2, 4, 6]`  

**15. A client asks for live frequency updates instead of a batch report. Why can’t we simply change an existing heap entry’s frequency in place?**

A. Its changed priority may break heap ordering, so updates require additional handling  
B. Frequencies cannot change after their first calculation  
C. Heaps support only unique frequencies  
D. Live updates make tie-breaking unnecessary  

Send your answers like:

```text
1.B
2.D
...
15.A
```

**Optional thinking challenge:** Explain why value `9` is excluded in Q14 even though it appears just as often as `4` and `6`.
