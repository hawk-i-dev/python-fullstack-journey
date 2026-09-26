# Day 23 DSA Quiz — Recursive Palindrome

### 1. What is a palindrome?

A. A string containing only unique characters  
B. A string that reads the same forward and backward  
C. A string arranged alphabetically  
D. A string containing an even number of characters  

### 2. Which technique is used in the recursive palindrome solution?

A. Prefix sum  
B. Sliding window  
C. Two pointers  
D. Binary search  

### 3. Where does the `left` pointer begin?

A. Index `0`  
B. Index `1`  
C. The middle index  
D. The final index  

### 4. Where does the `right` pointer begin?

A. `0`  
B. `len(text)`  
C. `len(text) - 1`  
D. `len(text) + 1`  

### 5. What is the successful base case?

A. `left == 0`  
B. `left >= right`  
C. `right == len(text)`  
D. `text[left] == text[right]`  

### 6. When should the function return `False`?

A. When `left >= right`  
B. When the string is empty  
C. When `text[left] != text[right]`  
D. When the string has an odd length  

### 7. What is the correct recursive call?

A. `check(left + 1, right - 1)`  
B. `check(left - 1, right + 1)`  
C. `check(left + 1, right + 1)`  
D. `check(left - 1, right - 1)`  

### 8. Why shouldn’t we return `True` after finding one matching pair?

A. Matching one pair does not prove every inner pair matches  
B. Matching characters must be deleted  
C. Recursion cannot return Boolean values  
D. The string must first be sorted  

### 9. What does this return?

```python
is_palindrome("level")
```

A. `False`  
B. `"level"`  
C. `True`  
D. `None`  

### 10. What does this return?

```python
is_palindrome("hello")
```

A. `True`  
B. `False`  
C. `"olleh"`  
D. An error  

### 11. Under the exact, case-sensitive implementation, what does this return?

```python
is_palindrome("Racecar")
```

A. `True`  
B. `False`  
C. `"racecar"`  
D. An error  

### 12. What does the base implementation return for an empty string?

A. `True`  
B. `False`  
C. `None`  
D. An index error  

### 13. What is the time complexity?

A. `O(1)`  
B. `O(log n)`  
C. `O(n²)`  
D. `O(n)`  

### 14. What is the recursive space complexity?

A. `O(1)`  
B. `O(n)`  
C. `O(n²)`  
D. `O(log n)`  

### 15. Why is an iterative two-pointer solution generally preferred for large production inputs?

A. It always runs in `O(1)` time  
B. It automatically ignores punctuation  
C. It uses `O(1)` extra space and avoids recursive call-stack growth  
D. It changes the original string  

Send your answers like this:

```text
1.B
2.C
3.A
...
15.C
```
