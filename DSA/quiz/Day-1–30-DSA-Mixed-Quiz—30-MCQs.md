# Day 1–30 DSA Mixed Quiz — 30 MCQs

One question represents each learning day. Do not check your notes while answering.

### 1. Which data structure provides fast duplicate detection?

A. Queue  
B. Set  
C. Linked list  
D. Stack  

### 2. In the optimized Two Sum solution, what is normally stored in the hash map?

A. Target and array length  
B. Only the complement  
C. Number and its index  
D. Sorted numbers  

### 3. What is required for two strings to be anagrams?

A. The same characters with the same frequencies  
B. Only the same length  
C. The same first character  
D. Characters in the same order  

### 4. Which pattern efficiently checks whether a string is a palindrome?

A. Prefix sum  
B. Two pointers  
C. Binary search  
D. Queue  

### 5. Why is Two Sum II efficiently solved using two pointers?

A. The array contains no duplicates  
B. The array has an even length  
C. Every number is positive  
D. The array is sorted  

### 6. When should a fixed sliding window be considered?

A. The input is a linked list  
B. Every value must be sorted  
C. The required contiguous window has a fixed size  
D. Recursion is mandatory  

### 7. What does a variable sliding window normally do?

A. Expands and shrinks according to a condition  
B. Always contains exactly `k` elements  
C. Moves only the left pointer  
D. Uses recursive calls  

### 8. After prefix-sum preprocessing, a range-sum query can usually be answered in:

A. `O(n)`  
B. `O(1)`  
C. `O(n²)`  
D. `O(log n)` only  

### 9. Which pattern is commonly used for Subarray Sum Equals K?

A. Stack and queue  
B. Two pointers only  
C. Binary search  
D. Prefix sum and hash map  

### 10. Which data structure is used for Valid Parentheses?

A. Stack  
B. Set  
C. Queue  
D. Heap  

### 11. What complexity should `get_min()` have in a correctly designed Min Stack?

A. `O(n)`  
B. `O(log n)`  
C. `O(1)`  
D. `O(n²)`  

### 12. Daily Temperatures is efficiently solved with:

A. Normal queue  
B. Monotonic stack  
C. Prefix sum  
D. Binary tree  

### 13. Which Python structure is best for removing expired requests from the front?

A. `list` with `pop()`  
B. Set  
C. Tuple  
D. `collections.deque`  

### 14. Which variables are central to iterative linked-list reversal?

A. `previous`, `current`, `next_node`  
B. `left`, `right`, `middle`  
C. `prefix`, `target`, `count`  
D. `minimum`, `maximum`, `average`  

### 15. What is the time complexity of binary search?

A. `O(n²)`  
B. `O(n)`  
C. `O(log n)`  
D. `O(1)` for every search  

### 16. What does Koko Eating Bananas binary-search over?

A. Banana indexes  
B. Possible eating speeds  
C. Number of monkeys  
D. Sorted pile positions  

### 17. What does Ship Packages Within D Days binary-search over?

A. Package indexes  
B. Number of packages  
C. Number of workers  
D. Possible ship capacities  

### 18. What must every correct recursive solution have?

A. A base case that eventually stops recursion  
B. A queue  
C. A sorted input  
D. Two hash maps  

### 19. Why is naive recursive Fibonacci slow?

A. It modifies the input  
B. It sorts every result  
C. It repeatedly solves the same subproblems  
D. It uses no recursive calls  

### 20. What is the correct base case for recursive array sum using an index?

A. `index == 0`  
B. `index == len(nums)`  
C. `nums[index] == 0`  
D. `index == nums[index]`  

### 21. What should recursive maximum return when the index reaches the final element?

A. `0`  
B. The array length  
C. The first element  
D. `nums[index]`  

### 22. What is the correct stopping condition for recursive string reversal with two pointers?

A. `left >= right`  
B. `left == 0`  
C. `right == len(text)`  
D. `left <= right`  

### 23. When should recursive palindrome checking return `False`?

A. When the pointers meet  
B. When the string is empty  
C. When `text[left] != text[right]`  
D. When the string length is odd  

### 24. Why does subset generation save `path.copy()`?

A. To sort each subset  
B. To save a snapshot rather than the same changing list  
C. To calculate `2ⁿ`  
D. To remove every empty subset  

### 25. What prevents an input position from being selected twice while generating permutations?

A. Prefix sums  
B. A minimum stack  
C. Binary search  
D. A `used` list  

### 26. In Combination Sum, what recursive index allows the current candidate to be reused?

A. The same `index`  
B. `index + 1`  
C. `index - 1`  
D. Always `0`  

### 27. Which condition correctly skips same-level duplicates in Combination Sum II?

A.

```python
if number in path:
    continue
```

B.

```python
if index == start:
    continue
```

C.

```python
if index > start and number == numbers[index - 1]:
    continue
```

D.

```python
if number == numbers[index - 1]:
    return
```

### 28. When is a closing parenthesis allowed during Generate Parentheses?

A. `close_count < n`  
B. `close_count < open_count`  
C. `close_count <= open_count`  
D. `open_count < close_count`  

### 29. After exploring from a temporarily marked grid cell in Word Search, what must happen?

A. Delete the cell  
B. Leave it permanently marked  
C. Sort its row  
D. Restore its original value  

### 30. What is the recursive formula for maximum binary-tree depth?

A.

```python
1 + max(max_depth(root.left), max_depth(root.right))
```

B.

```python
max_depth(root.left) + max_depth(root.right)
```

C.

```python
1 + min(max_depth(root.left), max_depth(root.right))
```

D.

```python
max_depth(root.left)
```

Send your answers in this format:

```text
1.B
2.C
3.A
...
30.A
```
