# Day 24 DSA Quiz — Backtracking and Subsets

### 1. What does backtracking generally mean?

A. Sort, search and delete  
B. Choose, explore and undo  
C. Push, pop and peek  
D. Divide, merge and sort  

### 2. For each number in the subsets problem, what choices are available?

A. Skip it or take it  
B. Sort it or reverse it  
C. Add it or multiply it  
D. Move it left or right  

### 3. Which variable stores the subset currently being built?

A. `result`  
B. `index`  
C. `path`  
D. `nums`  

### 4. What is the correct base case?

A. `index == 0`  
B. `index == len(nums)`  
C. `index == len(nums) - 1`  
D. `path == nums`  

### 5. What should happen at the base case?

A. Clear `result`  
B. Sort `path`  
C. Save a copy of `path` and return  
D. Remove the final number from `nums`  

### 6. Why should we use `path.copy()`?

A. To sort the current subset  
B. To save a snapshot instead of the same changing list  
C. To remove duplicate numbers  
D. To reduce the recursion depth  

### 7. What does this line represent?

```python
backtrack(index + 1)
```

when called before adding the current number?

A. Taking the current number  
B. Removing the current number  
C. Skipping the current number  
D. Restarting the algorithm  

### 8. What does `path.pop()` do?

A. Removes the most recently selected number  
B. Removes every number  
C. Copies the current subset  
D. Deletes a subset from `result`  

### 9. Why is `path.pop()` necessary?

A. To sort `path`  
B. To undo a choice before exploring another branch  
C. To stop recursion permanently  
D. To calculate the subset count  

### 10. How many subsets exist for three unique elements?

A. `3`  
B. `6`  
C. `8`  
D. `9`  

### 11. What does `generate_subsets([])` return?

A. `[]`  
B. `[[]]`  
C. `None`  
D. An error  

### 12. What happens if we store `path` without copying it?

```python
result.append(path)
```

A. Each subset is automatically sorted  
B. Python immediately raises an error  
C. Saved entries refer to the same list and change with it  
D. Recursion becomes `O(1)`  

### 13. What is the time complexity when subset copying is included?

A. `O(n × 2ⁿ)`  
B. `O(n)`  
C. `O(n²)`  
D. `O(log n)`  

### 14. What is the recursion depth for `n` elements?

A. `O(1)`  
B. `O(log n)`  
C. `O(2ⁿ)`  
D. `O(n)`  

### 15. What should be considered before generating every subset in production?

A. Whether the array is already sorted  
B. Whether every subset is truly required because output grows exponentially  
C. Whether a stack can replace Python  
D. Whether every value is positive  

Send your answers like this:

```text
1.B
2.A
3.C
...
15.B
```
