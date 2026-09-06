## Day 15 DSA Quiz — Binary Search + Search Insert Position

Reply like:

```text
1.A
2.B
3.C
...
15.A
```

### 1. Binary search works correctly only when data is:

A. Unsorted  
B. Sorted  
C. Empty only  
D. Duplicated only

### 2. Binary search removes how much search space each step?

A. One element only  
B. Half of the search space  
C. The whole array always  
D. Nothing

### 3. Middle index is usually calculated as:

A. `(left + right) // 2`  
B. `left + right`  
C. `right - left`  
D. `left * right`

### 4. If `nums[mid] == target`, return:

A. `left`  
B. `right`  
C. `mid`  
D. `-1`

### 5. If `nums[mid] < target`, what should we do?

A. `left = mid + 1`  
B. `right = mid - 1`  
C. return `mid`  
D. sort again

### 6. If `nums[mid] > target`, what should we do?

A. `left = mid + 1`  
B. `right = mid - 1`  
C. return `right`  
D. return `0`

### 7. In Search Insert Position, if target is not found, return:

A. `right`  
B. `left`  
C. `mid`  
D. `-1`

### 8. For `nums = [1, 3, 5, 6]`, `target = 5`, output is:

A. 0  
B. 1  
C. 2  
D. 3

### 9. For `nums = [1, 3, 5, 6]`, `target = 2`, output is:

A. 0  
B. 1  
C. 2  
D. 4

### 10. For `nums = [1, 3, 5, 6]`, `target = 7`, output is:

A. 0  
B. 2  
C. 3  
D. 4

### 11. For `nums = [1, 3, 5, 6]`, `target = 0`, output is:

A. 0  
B. 1  
C. 3  
D. 4

### 12. Binary search time complexity is:

A. O(1)  
B. O(n)  
C. O(log n)  
D. O(n²)

### 13. Binary search space complexity for iterative solution is:

A. O(1)  
B. O(n)  
C. O(log n)  
D. O(n²)

### 14. Which is a common mistake?

A. Returning `left` when target is missing  
B. Using binary search on unsorted array  
C. Moving pointers correctly  
D. Calculating middle

### 15. 80/20 rule for binary search is:

A. Use when data is sorted and each check removes half  
B. Use when array is random  
C. Use when counting characters  
D. Use when stack is needed
