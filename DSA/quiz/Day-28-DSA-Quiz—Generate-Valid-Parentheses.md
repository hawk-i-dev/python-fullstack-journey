# Day 28 DSA Quiz — Generate Valid Parentheses

### 1. When can an opening parenthesis be added?

A. `open_count < n`  
B. `open_count <= n`  
C. `close_count < n`  
D. `open_count == close_count`  

### 2. When can a closing parenthesis be added?

A. `close_count < n`  
B. `close_count < open_count`  
C. `close_count <= open_count`  
D. `open_count < close_count`  

### 3. Why can’t `")"` be the first character?

A. The path must start alphabetically  
B. No opening parenthesis exists to close  
C. Python rejects strings beginning with `")"`  
D. It would exceed `n` opening parentheses  

### 4. When is a sequence complete?

A. `open_count == 0`  
B. `close_count == 0`  
C. `len(path) == n`  
D. `len(path) == 2 * n`  

### 5. Which invariant must remain true?

A.

```text
0 ≤ open_count ≤ close_count ≤ n
```

B.

```text
0 ≤ close_count ≤ open_count ≤ n
```

C.

```text
open_count > n
```

D.

```text
close_count == n + 1
```

### 6. What does this condition prevent?

```python
if close_count < open_count:
```

A. Adding too many opening parentheses  
B. Creating a prefix with more closings than openings  
C. Completing a valid sequence  
D. Joining the path  

### 7. How many valid results exist for `n = 3`?

A. `3`  
B. `4`  
C. `5`  
D. `8`  

### 8. What should `generate_parentheses(0)` return?

A. `[]`  
B. `[""]`  
C. `["()"]`  
D. An error  

### 9. Why is `path.pop()` required?

A. To undo a choice before exploring another branch  
B. To delete a completed result  
C. To decrease `n`  
D. To validate the input type  

### 10. What is wrong with this condition?

```python
if open_count <= n:
```

A. It prevents all opening parentheses  
B. It may allow `n + 1` opening parentheses  
C. It allows too many closing parentheses  
D. Nothing is wrong  

### 11. Why is constrained backtracking better than generating every string and filtering afterward?

A. It sorts the results automatically  
B. It prevents invalid branches from being explored  
C. It uses no recursion  
D. It always produces one result  

### 12. What is the time complexity in terms of the Catalan number `Cₙ`?

A. `O(n)`  
B. `O(2n)`  
C. `O(Cₙ × n)`  
D. `O(log n)`  

### 13. Which corporate safeguard is appropriate?

A. Accept an unlimited `n` value  
B. Validate the input and enforce pair/result limits  
C. Generate every result even when only the count is required  
D. Accept negative values silently  

### 14. If the client needs only the number of valid patterns, what is the better approach?

A. Generate and store every pattern  
B. Return an arbitrary estimate  
C. Calculate the Catalan count without building every string  
D. Generate invalid strings too  

### 15. If the client introduces multiple bracket types such as `()`, `[]` and `{}`, what may be required?

A. No state changes  
B. Only sorting  
C. Tracking bracket types, potentially using a stack  
D. Removing the base case  

Send your answers like this:

```text
1.A
2.B
3.B
...
15.C
```
