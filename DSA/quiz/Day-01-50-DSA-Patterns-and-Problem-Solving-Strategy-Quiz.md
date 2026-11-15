# Day 1–50 DSA: Patterns & Problem-Solving Strategy Quiz

30 MCQs • One correct answer per question.  
Focus on **why the approach works**, not just the pattern name.

**1. Before choosing a data structure, what should you clarify first?**

A. Which solution has the fewest lines  
B. The input, required output, constraints, and mutation rules  
C. Whether recursion can be used  
D. Which algorithm you used yesterday  

**2. What is an invariant?**

A. A variable that never changes  
B. The final answer  
C. A rule that applies only to recursion  
D. A property that remains true at specified points during an algorithm  

**3. You only need to check whether a value appeared earlier. Which structure is usually appropriate?**

A. Set  
B. Queue  
C. Recursion stack  
D. Binary search tree with balancing implemented manually  

**4. In Two Sum, why do we check for the complement before storing the current element?**

A. To sort the values  
B. To avoid negative numbers  
C. To avoid using the current element twice  
D. To guarantee constant total execution time  

**5. Why does counting subarrays with sum `k` require prefix-sum frequencies rather than just a set?**

A. Prefix sums cannot repeat  
B. Multiple earlier occurrences of the same prefix can produce different valid subarrays  
C. Sets cannot store integers  
D. Frequencies automatically sort the array  

**6. In sorted Two Sum, the current sum is too small. Why can we move the left pointer forward?**

A. The current smallest value cannot reach the target with any remaining partner  
B. The largest value must be part of the answer  
C. Moving either pointer always increases the sum  
D. The smallest value must be negative  

**7. When is the standard “shrink while sum is too large” window approach unsafe without further reasoning?**

A. When the array contains only positive values  
B. When the window uses two pointers  
C. When the input has more than ten elements  
D. When negative values can make expansion or shrinking change the sum unpredictably  

**8. Given `prefix[0] = 0` and `prefix[i + 1] = sum(nums[:i + 1])`, how do we find the inclusive sum from index `left` to `right`?**

A. `prefix[right] - prefix[left]`  
B. `prefix[right + 1] + prefix[left]`  
C. `prefix[right + 1] - prefix[left]`  
D. `prefix[left] - prefix[right + 1]`  

**9. Daily Temperatures contains a loop inside another loop. Why is its monotonic-stack solution still `O(n)`?**

A. Python automatically combines the loops  
B. Each index is pushed once and popped at most once  
C. Every nested loop is linear  
D. The temperatures must be sorted  

**10. RecentCounter keeps calls in the inclusive interval `[t - 3000, t]`. Which timestamps should it remove?**

A. Timestamps equal to `t`  
B. Timestamps greater than `t - 3000`  
C. Timestamps less than or equal to `t - 3000`  
D. Timestamps strictly less than `t - 3000`  

**11. Before changing `current.next` while reversing a linked list, what must you preserve?**

A. The original next-node reference  
B. The largest node value  
C. The list’s sorted order  
D. A copy of every node  

**12. What allows binary search over possible shipping capacities?**

A. Package weights must be sorted  
B. Every capacity gives the same result  
C. Once a capacity is feasible, larger capacities remain feasible  
D. Package indexes are always the answer  

**13. What should you define first when designing a recursive function?**

A. How many print statements it needs  
B. Exactly what one call promises to return  
C. Whether its name contains `dfs`  
D. The final recursion depth for every input  

**14. Why does memoization speed up naive Fibonacci?**

A. It stores answers to repeated subproblems  
B. It removes the need for base cases  
C. It sorts recursive calls  
D. It makes all arithmetic operations free  

**15. Why should backtracking usually save `path.copy()` rather than `path`?**

A. Lists cannot be stored inside lists  
B. Copying automatically removes duplicates  
C. Later changes to the working path should not change saved answers  
D. Copying makes every algorithm constant-space  

**16. What is the correct general backtracking sequence?**

A. Undo → save → choose  
B. Explore all branches without restoring state  
C. Sort → binary search → return  
D. Choose → explore → undo  

**17. Why must BST validation consider ancestor bounds?**

A. A BST cannot contain negative values  
B. A node can satisfy its parent’s comparison but violate an earlier ancestor’s constraint  
C. Immediate-child comparisons always prove validity  
D. Every BST must be complete  

**18. A search follows one route through a BST of height `h`. What is its worst-case time?**

A. Always `O(1)`  
B. Always `O(log n)`  
C. `O(h)`, which can be `O(n)` in a skewed tree  
D. Always `O(n²)`  

**19. Why do recursive BST updates use assignments such as this?**

```python
root.left = delete_bst(root.left, key)
```

A. The operation may return a different root for that subtree  
B. Deletion always creates a new tree  
C. It sorts the left subtree  
D. It prevents recursion from visiting leaves  

**20. In the tree-diameter algorithm, what does the helper return to its parent?**

A. The largest value in the subtree  
B. The number of leaves  
C. The global diameter  
D. The subtree height  

**21. Why does maximum path sum return only one child branch upward?**

A. The right branch is always smaller  
B. Extending a two-branch path to the parent would create a fork, not a simple path  
C. A valid path must end at a leaf  
D. Negative values are forbidden  

**22. Which statement correctly distinguishes Path Sum II from Path Sum III?**

A. II lists root-to-leaf paths; III counts qualifying downward paths that may start anywhere  
B. Both list only root-to-leaf paths  
C. II counts paths; III modifies the tree  
D. III allows a path to move freely between sibling branches  

**23. Why must Path Sum III decrement the current prefix count when leaving a node?**

A. To make all prefix sums unique  
B. To remove negative values  
C. To prevent sibling branches from using prefixes outside their ancestor path  
D. To sort the remaining prefixes  

**24. Why does our preorder serialization include null markers?**

A. To ensure every value is unique  
B. To make the tree a BST  
C. To balance the reconstructed tree  
D. To preserve missing-child positions and therefore the structure  

**25. While flattening a tree in place, why must you preserve or reconnect the original right subtree before overwriting the right link?**

A. To sort its values  
B. To avoid losing access to nodes that still belong in the result  
C. To make every node a leaf  
D. To remove duplicate values  

**26. What is the right-side view of this tree?**

```text
        1
       / \
      2   3
       \
        5
         \
          6
```

A. `[1, 3]`  
B. `[1, 2, 5, 6]`  
C. `[1, 3, 5, 6]`  
D. `[3, 5, 6]`  

**27. A level has real nodes at positions `[0, 1, 7]`. What are its real-node count and positional width?**

A. Count `3`, width `8`  
B. Count `3`, width `3`  
C. Count `8`, width `7`  
D. Count `7`, width `8`  

**28. In the completeness-checking BFS, what proves that a tree is incomplete?**

A. Encountering the first `None`  
B. Finding duplicate values  
C. Finding a node with only a left child  
D. Processing a real node after an earlier missing position  

**29. Why is “every node is visited once, so total time is `O(n)`” insufficient for an algorithm that returns many copied paths?**

A. Visiting a node always takes `O(n)`  
B. Copying and producing the output also consumes time  
C. Recursion cannot be linear  
D. Every path algorithm is exponential  

**30. A client changes “return whether the tree is complete” to “report the first node that violates completeness.” What is the best approach?**

A. Replace BFS with binary search immediately  
B. Change the return type without checking existing callers  
C. Clarify the new contract, add tests, adapt the diagnostic output, and check compatibility  
D. Rewrite every tree operation before testing  

Send your answers like:

```text
1.B
2.D
3.A
...
30.C
```

**Optional thinking challenge:** For questions 7, 21, and 23, add one sentence explaining *why*.
