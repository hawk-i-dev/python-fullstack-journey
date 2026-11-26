# Day 55 DSA Quiz — Last Stone Weight

15 MCQs • One correct answer per question.  
For the trace questions, write down what goes **back into the pile** before choosing the next pair.

**1. Which stones must you pick in each round?**

A. The two lightest  
B. The two heaviest currently in the pile  
C. The first two in the original input  
D. Any two as long as their weights differ  

**2. What happens when you pick two stones that both weigh `6`?**

A. Put one stone of weight `6` back  
B. Put one stone of weight `12` back  
C. Put one stone of weight `0` back  
D. Remove both and put nothing back  

**3. Why store `-8` and `-7` instead of `8` and `7` in Python’s min-heap?**

A. `-8` pops before `-7`, so the original weight `8` comes out first  
B. Python’s min-heap cannot store positive integers  
C. Negation makes the entire heap list sorted  
D. `-7` pops first, which gives the heavier stone  

**4. You pop the stored keys `-10` and `-6`. What key should you push back?**

A. `4`  
B. `-16`  
C. `-4`  
D. `0`  

**5. For `[9, 7, 1]`, what happens after the first round?**

A. Put back `2`; pick `2` and `1` next  
B. Put back `2`; stop because only one original stone remains  
C. Put back `16`; pick `16` and `1` next  
D. Put nothing back; return `1` immediately  

**6. What does the heap represent at the start of every round?**

A. Only the original stones that have never been picked  
B. Exactly the remaining stones, each stored as its negative weight  
C. Every original stone plus all the stones created so far  
D. The remaining positive weights in fully sorted order  

**7. What does this return?**

```python
last_stone_weight([1, 2, 3])
```

A. `3`  
B. `2`  
C. `1`  
D. `0`  

**8. Which loop condition lets you safely pop two stones in each round?**

A. `while heap:`  
B. `while len(heap) == 2:`  
C. `while len(heap) >= 2:`  
D. `while len(heap) > 2:`  

**9. Why isn’t it enough to sort the original list once and then walk through it without reinserting anything?**

A. Sorting makes duplicate weights disappear  
B. A newly created stone can be among the heaviest in the next round  
C. Sorting cannot compare stones of different weights  
D. The lightest original stone always needs to be picked first  

**10. Why does the loop always terminate?**

A. Every round weighs the stones again and reduces each weight by `1`  
B. Each round removes exactly one stone, even on ties  
C. The heap automatically stops after a fixed number of pops  
D. Each round removes two stones and adds at most one, so the pile shrinks  

**11. How long does `heapq.heapify(heap)` take for `n` entries?**

A. `O(n)`  
B. `O(log n)`  
C. `O(n log n)`  
D. `O(n²)`  

**12. What are the worst-case time and auxiliary space bounds for the full solution?**

A. Time `O(n)`, space `O(1)`  
B. Time `O(log n)`, space `O(n)`  
C. Time `O(n log n)`, space `O(n)`  
D. Time `O(n log n)`, space `O(1)`  

**13. If the loop ends with `heap == [-4]`, what should the function return?**

A. `-4`  
B. `4`  
C. `1`  
D. `0`  

**14. What is the result for `[10, 4, 2, 10, 3]`?**

A. `0`  
B. `2`  
C. `3`  
D. `1`  

**15. In a work queue, a partially completed job has been removed for processing. It now needs to go back with a new priority. What should you do?**

A. Reinsert the job with a key based on its new priority  
B. Reinsert it with the old priority and leave it there  
C. Edit an unrelated heap entry to reflect the new priority  
D. Use a FIFO queue and assume the most urgent job still comes out first  

Send your choices as `1.<letter>, 2.<letter>, …, 15.<letter>`.

**Optional explain-back:** A full run on `[8, 7, 3]` should leave a stone of weight `2`. Trace the heap keys and explain what goes wrong if you push `+1` instead of `-1` after the first round.
