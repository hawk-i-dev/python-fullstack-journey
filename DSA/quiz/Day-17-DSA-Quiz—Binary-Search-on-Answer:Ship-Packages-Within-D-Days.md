Day 17 DSA Quiz — Binary Search on Answer: Ship Packages Within D Days

1. In this problem, what are we binary searching?

A. Package indexes  
B. Package weights order  
C. Possible ship capacity values  
D. Number of packages  

2. What is the minimum possible ship capacity?

A. `1`  
B. `max(weights)`  
C. `sum(weights)`  
D. `len(weights)`  

3. What is the maximum possible ship capacity?

A. `max(weights)`  
B. `days`  
C. `sum(weights)`  
D. `len(weights)`  

4. Why can binary search be used here?

A. The weights array is sorted  
B. Capacity values form a monotonic fail/pass pattern  
C. The answer is always the middle value  
D. We use recursion  

5. If a guessed capacity can ship within the given days, what should we do?

A. Save it and try smaller capacity  
B. Try larger capacity  
C. Return `-1`  
D. Sort the weights  

6. If a guessed capacity cannot ship within the given days, what should we do?

A. Try smaller capacity  
B. Try bigger capacity  
C. Stop immediately  
D. Remove heavy packages  

7. Why should `left = max(weights)`?

A. Because packages must be sorted  
B. Because ship must carry the heaviest package  
C. Because it gives fastest runtime only  
D. Because days starts from 1  

8. Why can `right = sum(weights)`?

A. Because this capacity can ship all packages in one day  
B. Because it is always the final answer  
C. Because it sorts the packages  
D. Because it avoids loops  

9. During checking one capacity, when do we start a new day?

A. When current load becomes zero  
B. When `current_load + weight > capacity`  
C. When weight is even  
D. When days become zero  

10. Should we sort `weights` before solving?

A. Yes, sorting is required  
B. No, order must be preserved  
C. Yes, descending order is best  
D. Only if days is 1  

11. For `weights = [1,2,3,4,5,6,7,8,9,10]`, `days = 5`, answer is:

A. `10`  
B. `14`  
C. `15`  
D. `55`  

12. If capacity `14` fails, what does it mean?

A. All smaller capacities will also fail  
B. All bigger capacities will fail  
C. We should return 14  
D. We should sort the array  

13. If capacity `20` works, what does it mean?

A. All smaller capacities definitely work  
B. A smaller capacity may still work, so search left  
C. We should search right  
D. The array is sorted  

14. Time complexity of the optimized solution is:

A. `O(n²)`  
B. `O(log n)`  
C. `O(n log S)`  
D. `O(S)`  

15. Space complexity is:

A. `O(1)`  
B. `O(n)`  
C. `O(log S)`  
D. `O(n²)`  

Send answers like:

```text
1.C
2.B
3.C
...
15.A
```
