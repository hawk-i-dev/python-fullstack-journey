# Day 25 DSA Quiz — Permutations and Real-World Practice

### 1. What is a permutation?

A. A selection where order does not matter  
B. An ordering that uses every element exactly once  
C. A sorted version of an array  
D. A collection containing only one element  

### 2. What is the main difference between subsets and permutations?

A. Subsets always use recursion  
B. Permutations cannot contain numbers  
C. Subsets decide skip/take; permutations choose an unused element for each position  
D. There is no difference  

### 3. Which structure records whether an input position is already selected?

A. `result`  
B. `used`  
C. `path.copy()`  
D. `index + 1`  

### 4. What is the correct base case?

A. `len(path) == len(nums)`  
B. `len(result) == len(nums)`  
C. `index == 0`  
D. `path == []`  

### 5. Why should the completed path be copied?

```python
result.append(path.copy())
```

A. To sort the permutation  
B. To save a snapshot instead of the same changing list  
C. To remove duplicates  
D. To reduce factorial growth  

### 6. What should happen when `used[index]` is `True`?

A. Add the element again  
B. Return from the complete function  
C. Skip that index using `continue`  
D. Clear the path  

### 7. Which is the correct choose operation?

A.

```python
used[index] = False
path.pop()
```

B.

```python
result.append(nums)
```

C.

```python
used[index] = True
path.append(nums[index])
```

D.

```python
path = []
```

### 8. What must happen after the recursive exploration?

A. Sort `result`  
B. Undo both `path` and `used` changes  
C. Delete the original input  
D. Return from inside the loop  

### 9. How many permutations exist for four unique elements?

A. `8`  
B. `16`  
C. `20`  
D. `24`  

### 10. What is the time complexity, including copying each permutation?

A. `O(n × n!)`  
B. `O(n)`  
C. `O(2ⁿ)`  
D. `O(log n)`  

### 11. What does the presented algorithm return for an empty input?

```python
generate_permutations([])
```

A. `[]`  
B. `[[]]`  
C. `None`  
D. An error  

### 12. What problem occurs if `used[index] = False` is forgotten?

A. The original array is sorted  
B. Previously selected positions remain unavailable to later branches  
C. The algorithm becomes iterative  
D. Every result becomes empty  

### 13. At the institute level, which submission best demonstrates complete understanding?

A. Only a screenshot of the output  
B. Code copied without explanation  
C. Implementation, tests, README and ability to answer viva questions  
D. Only complexity notation  

### 14. Why might a corporate implementation restrict the input to eight teams?

A. Python lists cannot contain nine values  
B. Factorial growth can consume excessive CPU and memory  
C. Eight is required by backtracking  
D. `used` works only for eight elements  

### 15. A client asks for every possible team schedule. What should you do before implementation?

A. Immediately generate all schedules without asking questions  
B. Replace the requirement with sorting  
C. Clarify limits, constraints, duplicates, output format and whether all schedules are required  
D. Promise that any input size will complete instantly  

Send your answers like this:

```text
1.B
2.C
3.B
...
15.C
```
