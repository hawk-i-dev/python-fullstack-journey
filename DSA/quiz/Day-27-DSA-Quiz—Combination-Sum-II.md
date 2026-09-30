# Day 27 DSA Quiz — Combination Sum II

### 1. What is the main rule in Combination Sum II?

A. Every candidate can be reused forever  
B. Each input position may be used only once  
C. Every candidate must be selected  
D. Order creates a new combination  

### 2. Why is the recursive call made with `index + 1`?

```python
backtrack(index + 1, remaining - number)
```

A. To allow the same position to be reused  
B. To restart from the beginning  
C. To prevent the selected position from being reused  
D. To sort the remaining candidates  

### 3. Why are candidates sorted first?

A. To place duplicates together and enable safe pruning  
B. To remove every duplicate value  
C. To make recursion unnecessary  
D. To change combinations into permutations  

### 4. Which condition skips a duplicate choice at the same recursion level?

A.

```python
if number == numbers[index - 1]:
    return
```

B.

```python
if index > start and number == numbers[index - 1]:
    continue
```

C.

```python
if index == start:
    continue
```

D.

```python
if number in path:
    break
```

### 5. What does `index > start` indicate in the duplicate rule?

A. The current value is a later sibling choice at the same level  
B. The target has been reached  
C. The candidate is larger than the target  
D. The current path is empty  

### 6. Why is `[1, 1, 6]` allowed when the input contains two `1` values?

A. The same input position is reused  
B. The two `1` values come from separate input positions  
C. Duplicate skipping is disabled everywhere  
D. The algorithm automatically creates another `1`  

### 7. Why should duplicates not be skipped globally?

A. Sorting would stop working  
B. Valid combinations using separate equal-valued positions could be lost  
C. It would modify the original list  
D. It would make `remaining` negative  

### 8. When should the current path be saved?

A. `remaining == 0`  
B. `index == 0`  
C. `len(path) == len(numbers)`  
D. `number == start`  

### 9. Which condition safely stops the loop after sorting?

A.

```python
if number < remaining:
    break
```

B.

```python
if number == remaining:
    continue
```

C.

```python
if number > remaining:
    break
```

D.

```python
if start > index:
    return
```

### 10. What should this return?

```python
combination_sum_once([1, 1, 2], 2)
```

A.

```python
[[1, 1], [2]]
```

B.

```python
[[1, 1]]
```

C.

```python
[[2]]
```

D.

```python
[[1, 1], [1, 1], [2]]
```

### 11. Why use `sorted(candidates)` instead of `candidates.sort()`?

A. `sorted()` returns only unique values  
B. `sorted()` avoids modifying the caller’s input list  
C. `.sort()` cannot sort integers  
D. `.sort()` has exponential complexity  

### 12. What is the worst-case search complexity, including path copying?

A. `O(n)`  
B. `O(n log n)`  
C. `O(n × 2ⁿ)`  
D. `O(log n)`  

### 13. Why is generating duplicates and removing them later with a set less desirable?

A. Sets cannot contain tuples  
B. It wastes time exploring duplicate branches that could be prevented  
C. Sets always modify the original candidates  
D. It prevents recursion from ending  

### 14. In the coupon assignment, what does “use each coupon once” mean?

A. A coupon value can never appear twice  
B. Each physical coupon position can be selected at most once  
C. Only one coupon may be used in total  
D. Duplicate-value coupons must be rejected  

### 15. Which client requirement could significantly change the algorithm?

A. Whether coupons can be reused, must total exactly, or have incompatibility rules  
B. The developer’s preferred text-editor theme  
C. The variable-name font  
D. Whether the README has a blue heading  

Send your answers like this:

```text
1.B
2.C
3.A
...
15.A
```
