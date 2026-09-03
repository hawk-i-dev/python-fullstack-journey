## Day 11 DSA Quiz — Stack + Min Stack

Reply like:

```text
1.A
2.B
3.C
...
15.A
```

### 1. Min Stack must support which operations?

A. `push`, `pop`, `top`, `get_min`  
B. `sort`, `reverse`, `merge`  
C. `search`, `delete`, `join`  
D. `commit`, `push`, `pull`

### 2. Target time complexity for each Min Stack operation is:

A. O(n)  
B. O(n²)  
C. O(1)  
D. O(log n)

### 3. Why is normal stack not enough?

A. `top()` is impossible  
B. `pop()` is impossible  
C. `get_min()` needs scanning if no extra memory is used  
D. Stack cannot store numbers

### 4. Main stack stores:

A. Only minimum values  
B. All values  
C. Only maximum values  
D. Sorted values

### 5. Min stack stores:

A. Current minimum history  
B. All values in sorted order  
C. Only popped values  
D. Only indexes

### 6. `get_min()` returns:

A. Top of `main_stack`  
B. Top of `min_stack`  
C. First value inserted  
D. Last popped value

### 7. On `push(value)`, we always push value into:

A. `main_stack`  
B. `min_stack` only  
C. Neither stack  
D. Database

### 8. When do we push value into `min_stack`?

A. Always only if value is positive  
B. If `min_stack` is empty or value `<=` current min  
C. Only if value is greater than current min  
D. Never

### 9. Why use `<=` instead of only `<` when pushing to `min_stack`?

A. To handle duplicate minimum values correctly  
B. To sort the stack  
C. To make values positive  
D. To avoid pop operation

### 10. On `pop()`, when should we also pop from `min_stack`?

A. Always  
B. Never  
C. If popped value equals current min  
D. If popped value is positive

### 11. If we call `min(stack)` inside `get_min()`, time becomes:

A. O(1)  
B. O(n)  
C. O(log n)  
D. O(n²)

### 12. Space complexity of Min Stack is:

A. O(1)  
B. O(n)  
C. O(n²)  
D. O(log n)

### 13. Dry run: after `push(-2), push(0), push(-3)`, `get_min()` returns:

A. -2  
B. 0  
C. -3  
D. None

### 14. After `push(-2), push(0), push(-3), pop()`, `get_min()` returns:

A. -3  
B. 0  
C. -2  
D. None

### 15. Which is a common mistake?

A. Handling empty stack  
B. Using helper min stack  
C. Forgetting duplicate minimums  
D. Returning top of min stack for `get_min()`
