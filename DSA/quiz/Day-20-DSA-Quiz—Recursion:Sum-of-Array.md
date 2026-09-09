Day 20 DSA Quiz — Recursion: Sum of Array

1. What is the goal of the Sum of Array problem?

A. Find the largest number  
B. Return the total sum of all numbers  
C. Sort the array  
D. Count only even numbers  

2. For `nums = [2, 4, 6, 8]`, what should `sum_array(nums)` return?

A. `18`  
B. `20`  
C. `24`  
D. `8`  

3. In recursive array sum, what is the smallest/base case?

A. First index  
B. Last number  
C. No numbers left  
D. Largest number  

4. If no numbers are left, what should we return for sum?

A. `0`  
B. `1`  
C. `None`  
D. `-1`  

5. What variable do we usually move forward in index-based recursion?

A. `total`  
B. `index`  
C. `left`  
D. `right`  

6. Correct stop condition is:

A. `index == 0`  
B. `index == len(nums)`  
C. `index == nums[index]`  
D. `index > nums[index]`  

7. Correct recursive return logic is:

A. `nums[index] + helper(index + 1)`  
B. `nums[index] + helper(index)`  
C. `helper(index - 1)`  
D. `nums[index] * helper(index + 1)`  

8. Why should we avoid `nums[1:]` repeatedly?

A. It changes values randomly  
B. It creates new arrays and wastes memory  
C. It sorts the array  
D. It skips the base case automatically  

9. For `nums = [5, 10, 15]`, output is:

A. `15`  
B. `25`  
C. `30`  
D. `35`  

10. Time complexity of recursive sum is:

A. `O(1)`  
B. `O(log n)`  
C. `O(n)`  
D. `O(n²)`  

11. Space complexity of recursive sum is:

A. `O(1)`  
B. `O(n)`  
C. `O(log n)`  
D. `O(n²)`  

12. Why is recursive space `O(n)`?

A. Because each call waits on call stack  
B. Because array gets sorted  
C. Because hashmap is required  
D. Because binary search is used  

13. Iterative sum using loop has space complexity:

A. `O(n)`  
B. `O(log n)`  
C. `O(1)`  
D. `O(n²)`  

14. Which is a common mistake?

A. Returning `0` at end  
B. Moving index forward  
C. Forgetting `index + 1`  
D. Using base case  

15. In production, for simple array sum, which is usually better?

A. Loop version  
B. Recursive version always  
C. Binary search  
D. Stack with strings  

Send answers like:

```text
1.B
2.B
3.C
...
15.A
```
