## Day 12 DSA Quiz — Monotonic Stack + Daily Temperatures

Reply like:

```text
1.A
2.B
3.C
...
15.A
```

### 1. Daily Temperatures problem asks:

A. Sort temperatures  
B. Find days until a warmer future temperature  
C. Count duplicate temperatures  
D. Reverse temperatures

### 2. 80/20 core for this topic is:

A. Use monotonic stack when old items wait for a future greater value  
B. Always use nested loops  
C. Use prefix sum for all arrays  
D. Use binary search without sorting

### 3. Brute force approach scans:

A. Previous days  
B. Future days for each day  
C. Only first day  
D. Only last day

### 4. Brute force worst-case time is:

A. O(1)  
B. O(n)  
C. O(n²)  
D. O(log n)

### 5. Optimized solution uses:

A. Stack  
B. Queue  
C. Tree  
D. Database

### 6. Stack stores:

A. Temperatures only  
B. Indexes of waiting days  
C. Sorted values  
D. Answers only

### 7. Why store indexes instead of temperatures?

A. Need distance: `current_day - previous_day`  
B. Need alphabetical order  
C. Need database ID  
D. Need duplicate removal only

### 8. When current temperature is warmer than temperature at stack top:

A. Pop previous day and calculate wait  
B. Sort stack  
C. Clear entire answer  
D. Return immediately always

### 9. Answer formula after popping previous day is:

A. `previous_day - current_day`  
B. `current_day - previous_day`  
C. `current_temp - previous_temp`  
D. `len(stack)`

### 10. Why do we use `while` instead of single `if`?

A. One warm day may answer multiple previous colder days  
B. While is always faster  
C. Stack cannot use if  
D. Temperatures are strings

### 11. Default answer values should be:

A. 0  
B. 1  
C. -1  
D. None

### 12. For `[30, 40, 50, 60]`, output is:

A. `[1, 1, 1, 0]`  
B. `[0, 0, 0, 0]`  
C. `[3, 2, 1, 0]`  
D. `[1, 2, 3, 4]`

### 13. For `[90, 80, 70, 60]`, output is:

A. `[1, 1, 1, 0]`  
B. `[0, 0, 0, 0]`  
C. `[3, 2, 1, 0]`  
D. `[90, 80, 70, 60]`

### 14. Optimized time complexity is:

A. O(1)  
B. O(n)  
C. O(n²)  
D. O(log n)

### 15. Why is nested `while` still O(n)?

A. Each index is pushed once and popped once  
B. Because Python skips loops  
C. Because stack is sorted  
D. Because answer starts with zero
