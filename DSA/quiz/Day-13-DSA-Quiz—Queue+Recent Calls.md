## Day 13 DSA Quiz — Queue + Recent Calls

Reply like:

```text
1.A
2.B
3.C
...
15.A
```

### 1. Queue follows which rule?

A. LIFO  
B. FIFO  
C. Random order  
D. Sorted order

### 2. FIFO means:

A. First In, First Out  
B. Fast Input, Fast Output  
C. Final Index First Only  
D. First Item Forced Outside

### 3. Queue is like:

A. Stack of plates  
B. Ticket line  
C. Binary tree  
D. Hash map only

### 4. RecentCounter counts requests in:

A. Last 3000 milliseconds  
B. Last 3 hours  
C. Last 3000 seconds  
D. All requests forever

### 5. For `ping(t)`, valid range is:

A. `[t - 3000, t]`  
B. `[0, t]`  
C. `[t, t + 3000]`  
D. `[t - 30, t]`

### 6. Is the range `[t - 3000, t]` inclusive?

A. Yes  
B. No  
C. Only for `t = 0`  
D. Only for negative values

### 7. Which timestamp is expired?

A. `timestamp < t - 3000`  
B. `timestamp == t - 3000`  
C. `timestamp > t`  
D. `timestamp == t`

### 8. Why is `timestamp == t - 3000` still valid?

A. Because the range is inclusive  
B. Because queue is sorted  
C. Because Python ignores it  
D. Because deque removes it

### 9. Why does queue fit this problem?

A. Oldest timestamp is at the front  
B. Newest timestamp must leave first  
C. Stack is required  
D. Sorting is required every time

### 10. Which Python structure is best here?

A. `list` with `pop(0)`  
B. `deque`  
C. `tuple`  
D. `set`

### 11. Why avoid `list.pop(0)` repeatedly?

A. It shifts elements and becomes slow  
B. It cannot remove values  
C. It sorts the list  
D. It removes from back

### 12. At `t = 3002`, valid range is:

A. `[2, 3002]`  
B. `[1, 3002]`  
C. `[0, 3002]`  
D. `[3002, 6002]`

### 13. For calls `ping(1), ping(100), ping(3001), ping(3002)`, outputs are:

A. `1, 2, 3, 3`  
B. `1, 1, 1, 1`  
C. `0, 1, 2, 3`  
D. `4, 3, 2, 1`

### 14. Average time for each ping is:

A. O(1)  
B. O(n²)  
C. O(log n)  
D. O(n log n)

### 15. Common mistake is:

A. Removing timestamps where `timestamp == t - 3000`  
B. Using deque  
C. Removing expired old pings  
D. Returning queue length
