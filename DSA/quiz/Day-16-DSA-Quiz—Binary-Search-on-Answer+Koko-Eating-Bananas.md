Day 16 DSA Quiz — Binary Search on Answer + Koko Eating Bananas

1. In “Binary Search on Answer”, what are we searching?

A. Array indexes  
B. Possible answer values  
C. Dictionary keys  
D. Stack elements  

2. In Koko Eating Bananas, what is the answer we need to find?

A. Maximum pile size  
B. Minimum eating speed `k`  
C. Number of piles  
D. Total bananas  

3. For Koko’s problem, the lowest possible speed is usually:

A. `0`  
B. `1`  
C. `len(piles)`  
D. `h`  

4. The highest possible speed can be:

A. `sum(piles)`  
B. `len(piles)`  
C. `max(piles)`  
D. `h / 2`  

5. Why can binary search be used here?

A. The piles array is sorted  
B. The answer space has a monotonic pass/fail pattern  
C. We use a stack  
D. We need recursion  

6. If a guessed speed finishes within `h` hours, what should we do?

A. Try a larger speed only  
B. Save it and try a smaller speed  
C. Stop always  
D. Ignore it  

7. If a guessed speed takes more than `h` hours, what should we do?

A. Try a smaller speed  
B. Try a bigger speed  
C. Return `-1`  
D. Sort the piles  

8. For one pile, how do we calculate hours needed?

A. `pile // speed`  
B. `ceil(pile / speed)`  
C. `speed // pile`  
D. `pile * speed`  

9. For `piles = [3,6,7,11]`, `h = 8`, answer is:

A. `3`  
B. `4`  
C. `6`  
D. `11`  

10. If speed is too slow, the result is:

A. Pass  
B. Fail  
C. Sorted  
D. Duplicate  

11. In the pass/fail pattern `F F F T T T`, what are we finding?

A. First `True`  
B. Last `False`  
C. Middle pile  
D. Maximum index  

12. Time complexity of Koko Eating Bananas optimized solution is:

A. `O(n)`  
B. `O(log n)`  
C. `O(n log m)`  
D. `O(n²)`  

13. In `O(n log m)`, what does `m` represent?

A. Number of piles  
B. Largest pile size  
C. Number of hours  
D. Smallest pile size  

14. Space complexity is:

A. `O(1)`  
B. `O(n)`  
C. `O(log n)`  
D. `O(n²)`  

15. Which is a common mistake in this problem?

A. Searching speed values  
B. Using ceiling division  
C. Searching pile indexes instead of speed values  
D. Saving the current valid answer  

Send your answers like:

```text
1.B
2.B
3.B
...
15.C
```
