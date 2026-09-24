# Day 21 DSA Quiz — Recursive Maximum

Choose one answer for each question.

### 1. What is the purpose of `find_max()`?

A. Sort the array  
B. Find the largest element  
C. Find the smallest element  
D. Reverse the array  

### 2. What does each recursive call solve?

A. Maximum from the current index to the end  
B. Maximum before the current index  
C. Sum of the complete array  
D. Length of the array  

### 3. Which value changes in every recursive call?

A. `nums`  
B. `index`  
C. Array length  
D. Maximum array size  

### 4. What is the correct base case?

A. `index == 0`  
B. `index == len(nums)`  
C. `index == len(nums) - 1`  
D. `nums[index] == 0`  

### 5. What should the base case return?

A. `0`  
B. `nums[0]`  
C. `nums[index]`  
D. `len(nums)`  

### 6. What is the correct recursive call?

A. `find_max(nums, index - 1)`  
B. `find_max(nums, index + 1)`  
C. `find_max(index + 1)`  
D. `find_max(nums + 1, index)`  

### 7. What does `right_max` contain?

A. The maximum value from `index + 1` to the end  
B. The array’s final index  
C. The current array element  
D. The sum of the remaining elements  

### 8. How do we combine the current element with the recursive answer?

A. Add them  
B. Compare them and return the larger one  
C. Subtract them  
D. Store them in a queue  

### 9. What is returned for this input?

```python
find_max([4, 7, 2, 9, 5])
```

A. `4`  
B. `5`  
C. `7`  
D. `9`  

### 10. What is returned for this input?

```python
find_max([-8, -3, -10])
```

A. `0`  
B. `-10`  
C. `-3`  
D. `-8`  

### 11. Why is returning `0` as the base value dangerous?

A. It produces an error for positive arrays  
B. It can produce an incorrect answer for all-negative arrays  
C. It changes the original array  
D. It sorts the array  

### 12. What is the time complexity?

A. `O(1)`  
B. `O(log n)`  
C. `O(n)`  
D. `O(n²)`  

### 13. What is the recursive space complexity?

A. `O(1)`  
B. `O(log n)`  
C. `O(n)`  
D. `O(n²)`  

### 14. Why is iteration usually safer for a very large array in Python?

A. Iteration always sorts the array  
B. Recursion can exceed Python’s recursion limit  
C. Iteration uses a hashmap  
D. Recursion cannot compare numbers  

### 15. What should happen when the input array is empty?

A. Return `0` in every implementation  
B. Return the index  
C. Raise a clear error such as `ValueError`  
D. Call the function again  

Send your answers like this:

```text
1.B
2.A
3.B
...
15.C
```
