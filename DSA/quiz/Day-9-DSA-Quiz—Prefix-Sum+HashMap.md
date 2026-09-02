## Day 9 DSA Quiz — Prefix Sum + Hash Map

Reply like:

```text
1.A
2.B
3.C
...
15.A
```

### 1. Day 9 problem is mainly about:

A. Prefix Sum + Hash Map  
B. Two Pointers  
C. Stack  
D. Binary Search

### 2. Problem asks us to count:

A. Sorted elements  
B. Continuous subarrays whose sum equals k  
C. Duplicate characters  
D. Tree nodes

### 3. Subarray means:

A. Continuous part of array  
B. Any selected elements in any order  
C. Only first element  
D. Sorted elements only

### 4. Key equation is:

A. `current_sum + old_sum = k`  
B. `current_sum - old_sum = k`  
C. `old_sum - current_sum = k`  
D. `current_sum * old_sum = k`

### 5. From the key equation, `needed` is:

A. `current_sum + k`  
B. `current_sum - k`  
C. `k - current_sum`  
D. `current_sum * k`

### 6. `prefix_count` stores:

A. index → value  
B. prefix_sum → how many times seen  
C. value → sorted index  
D. character → count

### 7. Why do we store counts, not only True/False?

A. Same prefix sum can appear multiple times  
B. To sort prefix sums  
C. To reverse the array  
D. To remove negative numbers

### 8. Why start with `{0: 1}`?

A. To count subarrays that start at index 0  
B. To sort the map  
C. To skip first number  
D. To avoid loops

### 9. Correct operation order is:

A. Check needed → update current_sum → save current_sum  
B. Update current_sum → check needed → save current_sum  
C. Save current_sum → update current_sum → check needed  
D. Sort → check → save

### 10. For `nums = [1, 1, 1]`, `k = 2`, output is:

A. 1  
B. 2  
C. 3  
D. 0

### 11. For `nums = [1, 2, 3]`, `k = 3`, output is:

A. 1  
B. 2  
C. 3  
D. 0

### 12. Why normal sliding window is not safe for this problem?

A. Because negative numbers can exist  
B. Because arrays cannot be looped  
C. Because hash map is always sorted  
D. Because strings are used

### 13. Time complexity is:

A. O(1)  
B. O(n)  
C. O(n²)  
D. O(log n)

### 14. Space complexity is:

A. O(1)  
B. O(n)  
C. O(n²)  
D. O(log n)

### 15. Which is a common mistake?

A. Forgetting `{0: 1}`  
B. Using prefix count map  
C. Checking needed after current_sum update  
D. Counting continuous subarrays
