# Day 26 DSA Quiz — Combination Sum and Real-World Practice

### 1. What is the goal of Combination Sum?

A. Sort the candidates  
B. Find combinations whose sum equals the target  
C. Find only the largest candidate  
D. Generate every permutation  

### 2. What does `remaining` represent?

A. The number of candidates  
B. The current recursion depth  
C. The amount still required to reach the target  
D. The number of completed answers  

### 3. When should the current path be saved?

A. `remaining == 0`  
B. `remaining > target`  
C. `start == 0`  
D. `len(path) == len(candidates)`  

### 4. Why does the recursive call use the same index?

```python
backtrack(index, remaining - number)
```

A. To prevent the number from being reused  
B. To allow the selected number to be reused  
C. To restart the search  
D. To sort the candidates  

### 5. If each candidate could be used only once, which call would be appropriate?

A. `backtrack(0, remaining)`  
B. `backtrack(index, remaining)`  
C. `backtrack(index + 1, remaining - number)`  
D. `backtrack(index - 1, remaining - number)`  

### 6. Why is a `start` index used?

A. To generate reordered duplicates  
B. To prevent reordered versions of the same combination  
C. To count the candidates  
D. To change the original input  

### 7. Why are the candidates sorted?

A. To guarantee that every candidate is selected  
B. To make `path.copy()` unnecessary  
C. To safely stop when the current and later values are too large  
D. To convert combinations into permutations  

### 8. Which condition safely prunes the loop after sorting?

A.

```python
if number > remaining:
    break
```

B.

```python
if number < remaining:
    return
```

C.

```python
if number == remaining:
    continue
```

D.

```python
if start > 0:
    break
```

### 9. Why is `path.pop()` required?

A. To sort the current combination  
B. To undo the selected candidate before exploring another branch  
C. To remove an answer from `result`  
D. To decrease the target permanently  

### 10. What is the result for this input?

```python
combination_sum([2], 6)
```

A. `[]`  
B. `[[2, 2]]`  
C. `[[2, 2, 2]]`  
D. `[[6]]`  

### 11. Why is a reusable candidate of `0` unsafe?

A. Zero automatically sorts the array  
B. Selecting zero does not reduce `remaining`, potentially causing infinite recursion  
C. Zero cannot exist in a Python list  
D. It always makes the target negative  

### 12. How should the time complexity generally be described?

A. Always `O(n)`  
B. Always `O(n²)`  
C. Exponential and dependent on the target, candidates and output  
D. Always `O(log n)`  

### 13. For the budget-package assignment, why should money normally be represented as integer paise or cents?

A. Integers automatically generate combinations  
B. Floating-point arithmetic may not represent decimal currency exactly  
C. Python cannot add floating-point values  
D. Integers prevent every duplicate combination  

### 14. Which corporate safeguard is most appropriate?

A. Accept unlimited targets and unlimited results  
B. Remove all input validation  
C. Validate positive values and enforce budget/result limits  
D. Modify the caller’s price list in place  

### 15. A client asks for package combinations. Which requirement must be clarified?

A. Whether reuse is allowed and whether the total must be exact or under budget  
B. Which code editor the developer prefers  
C. Whether variable names should contain vowels  
D. Whether recursion should always run forever  

Send your answers like this:

```text
1.B
2.C
3.A
...
15.A
```
