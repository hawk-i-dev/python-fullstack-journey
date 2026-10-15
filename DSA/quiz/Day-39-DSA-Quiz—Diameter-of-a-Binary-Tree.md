**Day 39 DSA Quiz — Diameter of a Binary Tree**

Choose one answer per question. Assume a valid binary tree. **Diameter counts edges**, while our helper measures **height in nodes**.

Use this tree for Questions 1–3:

```text
        1
       / \
      2   3
     / \
    4   5
```

1. What is the diameter of this tree?

A. `2`  
B. `4`  
C. `3`  
D. `5`  

2. Which is a longest path?

A. `4 → 2 → 1 → 3`  
B. `4 → 2 → 5 → 3`  
C. `1 → 2`  
D. `4 → 5 → 3`  

3. What height does the helper return for the subtree rooted at node `2`?

A. `0`  
B. `1`  
C. `3`  
D. `2`  

4. What does the height helper return for an empty subtree?

A. `-1`  
B. `0`  
C. `1`  
D. `None`  

5. Which formula calculates the diameter candidate through the current node?

A. `1 + max(left_height, right_height)`  
B. `left_height * right_height`  
C. `left_height + right_height`  
D. `1 + left_height + right_height`  

6. Which formula calculates the height returned to the parent?

A. `1 + max(left_height, right_height)`  
B. `left_height + right_height`  
C. `max(left_height, right_height)`  
D. `1 + min(left_height, right_height)`  

7. Why must the helper return one branch’s height rather than the sum of both branch heights?

A. Right subtrees never contribute to a diameter  
B. Every tree must be a BST  
C. The diameter must end at the root  
D. The parent can extend one downward branch without creating a fork  

8. What is the diameter of the following tree?

```text
          1
         /
        2
       / \
      3   4
     /     \
    5       6
```

A. `3`  
B. `4`  
C. `5`  
D. `6`  

9. What does the tree in Question 8 demonstrate?

A. The diameter always passes through the root  
B. Diameter and height are always equal  
C. The longest path can lie entirely inside a subtree  
D. Only right children affect diameter  

10. Why do we write this update?

```python
best = max(best, left_height + right_height)
```

A. To preserve the largest candidate found so far  
B. To sort the node values  
C. To ensure every candidate becomes the final answer  
D. To calculate the number of leaves  

11. What is the worst-case time complexity of the solution that calculates each node’s height once?

A. `O(log n)`  
B. `O(n²)`  
C. `O(1)`  
D. `O(n)`  

12. What is the auxiliary space complexity of the recursive solution, where `h` is tree height?

A. `O(1)` for every tree  
B. `O(h)`  
C. `O(n²)`  
D. `O(log n)` for every tree  

13. What does `nonlocal best` allow the nested helper to do?

A. Reset the entire tree  
B. Access a global variable shared by every program  
C. Reassign the enclosing function’s `best` variable  
D. Avoid using recursion  

14. Why should `best = 0` be initialized inside each call to `diameter()`?

A. To prevent an earlier call’s result from affecting the current call  
B. To ensure every tree has diameter zero  
C. To balance the tree  
D. To remove repeated node values  

15. A client says each connection has a different travel time and wants the route with the greatest total time. What should you recognize?

A. Counting edges already accounts for different travel times  
B. Sorting node values will solve the requirement  
C. Multiplying the edge count by the largest weight is always correct  
D. The algorithm must account for edge weights instead of simply counting connections  

Send your answers like:

```text
1.C
2.A
...
15.D
```
