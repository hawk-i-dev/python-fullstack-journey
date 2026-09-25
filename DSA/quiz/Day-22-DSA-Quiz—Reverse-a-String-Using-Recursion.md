# Day 22 DSA Quiz — Reverse a String Using Recursion

### 1. What is the goal of the Day 22 problem?

A. Sort a string  
B. Reverse a string using recursion  
C. Count the characters  
D. Remove repeated characters  

### 2. Why do we convert the Python string into a list?

A. Lists automatically reverse themselves  
B. Lists use no memory  
C. Strings are immutable, while lists can be modified  
D. Recursion works only with lists  

### 3. Where does the `left` pointer begin?

A. Index `0`  
B. Index `1`  
C. The middle index  
D. The final index  

### 4. Where does the `right` pointer begin?

A. Index `0`  
B. Index `1`  
C. `len(characters)`  
D. `len(characters) - 1`  

### 5. What is the correct base case?

A. `left == 0`  
B. `right == 0`  
C. `left >= right`  
D. `left <= right`  

### 6. Why do we use `left >= right`?

A. It handles pointers meeting and crossing  
B. It sorts the characters  
C. It detects repeated characters  
D. It creates a new string  

### 7. What work does each recursive call perform?

A. Deletes one character  
B. Swaps one outside pair  
C. Sorts the remaining characters  
D. Counts the remaining characters  

### 8. What is the correct recursive call?

A. `reverse(left - 1, right + 1)`  
B. `reverse(left + 1, right + 1)`  
C. `reverse(left - 1, right - 1)`  
D. `reverse(left + 1, right - 1)`  

### 9. What is the result of this call?

```python
reverse_string("CODE")
```

A. `"EDOC"`  
B. `"CODE"`  
C. `"DOCE"`  
D. `"OCDE"`  

### 10. What happens when the input is an empty string?

A. Python raises an index error  
B. Recursion never stops  
C. The base case stops immediately and returns `""`  
D. The function returns `None`  

### 11. What is the result of reversing a one-character string?

```python
reverse_string("A")
```

A. `""`  
B. `"A"`  
C. `None`  
D. An error  

### 12. What does this statement do?

```python
characters[left], characters[right] = (
    characters[right],
    characters[left],
)
```

A. Deletes both characters  
B. Joins both characters  
C. Compares both characters  
D. Swaps both characters  

### 13. What is the time complexity?

A. `O(n)`  
B. `O(1)`  
C. `O(log n)`  
D. `O(n²)`  

### 14. What is the recursive space complexity?

A. `O(1)`  
B. `O(n)`  
C. `O(log n)`  
D. `O(n²)`  

### 15. Which solution is normally preferred for simply reversing a string in production Python code?

A. Recursive binary search  
B. A hashmap  
C. `text[::-1]`  
D. A queue  

Send your answers like this:

```text
1.B
2.C
3.A
...
15.C
```
