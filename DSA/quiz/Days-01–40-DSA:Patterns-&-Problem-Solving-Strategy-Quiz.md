**Days 1–40 DSA: Patterns & Problem-Solving Strategy Quiz**

These 30 MCQs test how you choose an approach, track information, and avoid common mistakes. Choose one answer per question.

1. Before choosing an algorithm, what should you establish first?

A. Which solution has the shortest code  
B. The required output, input conditions, and expected behavior  
C. Whether recursion can be used  
D. Which data structure you used yesterday  

2. You only need to check whether a number appeared earlier. Which structure is usually appropriate?

A. Queue  
B. Stack  
C. Binary tree  
D. Set  

3. In Two Sum, why do we check for the complement before storing the current element’s index?

A. To avoid using the same element twice  
B. To sort the input automatically  
C. To remove all duplicate values  
D. To ensure the answer uses adjacent elements  

4. Two strings have equal lengths. What additional condition makes them anagrams?

A. Their first characters match  
B. They contain no repeated characters  
C. Every character has the same frequency in both strings  
D. Their characters already appear in sorted order  

5. In a sorted array, a two-pointer pair sum is smaller than the target. Which move can increase the sum?

A. Move the right pointer left  
B. Move the left pointer right  
C. Move both pointers outward  
D. Stop because no answer exists  

6. A fixed-size window moves one position right. How should its sum be updated?

A. Old sum − outgoing value + incoming value  
B. Old sum + outgoing value − incoming value  
C. Old sum + both values  
D. Recalculate every possible subarray sum  

7. In a longest-unique-substring solution, why might shrinking the window require a `while` loop?

A. Every character must be removed twice  
B. The input must first be sorted  
C. The window must always become empty  
D. Removing one character may not eliminate the duplicate  

8. Using `prefix[0] = 0` and `prefix[i + 1] = prefix[i] + nums[i]`, what gives the sum of inclusive range `[L, R]`?

A. `prefix[R] - prefix[L]`  
B. `prefix[R + 1] + prefix[L]`  
C. `prefix[R + 1] - prefix[L]`  
D. `prefix[R] - prefix[L + 1]`  

9. Why does Subarray Sum Equals K store frequencies of earlier prefix sums?

A. Prefix sums are always unique  
B. Several earlier positions can have the same prefix sum and produce different valid subarrays  
C. Frequencies automatically remove negative values  
D. Only the largest prefix sum matters  

10. A problem asks for a contiguous subarray whose sum equals a target, and negative values are allowed. What is the best reasoning?

A. Every subarray problem requires a fixed window  
B. Sorting preserves all original contiguous subarrays  
C. Negative values make the problem impossible  
D. Consider prefix sums because expanding a window does not predictably increase its sum  

11. Why is a stack suitable for checking nested parentheses?

A. The most recently opened unfinished bracket must close first  
B. The oldest opening bracket must always close first  
C. Parentheses must be sorted  
D. A stack prevents all invalid input automatically  

12. In Daily Temperatures, why do we commonly store indexes in the monotonic stack?

A. Temperatures cannot be stored in Python lists  
B. Indexes make sorting unnecessary in every algorithm  
C. Index differences give the number of days waited  
D. Indexes must always be processed in ascending temperature order  

13. A RecentCounter keeps calls in the inclusive interval `[t - 3000, t]`. Which timestamps should it remove?

A. Timestamps equal to `t - 3000`  
B. Timestamps strictly less than `t - 3000`  
C. Every timestamp less than `t`  
D. Only the newest timestamp  

14. During linked-list reversal, why save `next_node = current.next` before changing `current.next`?

A. To preserve access to the unprocessed remainder of the list  
B. To sort the node values  
C. To create a second complete list  
D. To ensure the list contains no duplicates  

15. What makes binary search on eating speed or shipping capacity possible?

A. Every input array is sorted  
B. The answer must be an array index  
C. Feasibility changes predictably as the candidate value increases  
D. Every candidate must be tested individually  

16. Before implementing recursion, which set of questions is most useful?

A. Can I avoid all variables and loops?  
B. Can I make the function call itself twice?  
C. Can I remove the base case?  
D. What does the call mean, what does it return, when does it stop, and how does it make progress?  

17. When does memoization most directly help?

A. When every subproblem is different  
B. When the same subproblems are solved repeatedly  
C. When no intermediate answers exist  
D. When the input is already sorted  

18. Why should backtracking commonly save `path.copy()` instead of `path`?

A. Copying automatically sorts the path  
B. Python cannot store lists inside lists  
C. Later changes to the working path should not change saved answers  
D. Copying eliminates the need to undo choices  

19. In combination generation, what typically distinguishes allowing reuse of the chosen candidate from using it once?

A. Reuse recurses with index `i`; use-once recurses with `i + 1`  
B. Reuse always requires a queue  
C. Use-once means the remaining target never changes  
D. Both always restart at index `0`  

20. What state must Word Search restore after exploring a path that fails?

A. The sorted order of all letters  
B. The entire board’s dimensions  
C. Every previously completed answer  
D. The current path’s visited-cell state or temporary board marking  

21. Why does level-order BFS save `level_size = len(queue)` before processing a level?

A. To stop children being added to the queue  
B. To separate current-level nodes from children added during processing  
C. To sort each level  
D. To calculate the tree’s diameter  

22. Why can checking only immediate parent-child comparisons fail to validate a BST?

A. A node can satisfy its parent’s comparison but violate an ancestor’s bound  
B. BSTs cannot contain negative values  
C. Every valid BST must be balanced  
D. Only leaf values determine validity  

23. Which description of ordinary BST search is accurate?

A. It always takes `O(1)` time  
B. It always takes `O(log n)` time  
C. It takes `O(h)` time, which can become `O(n)` in a chain  
D. It requires visiting every node in every case  

24. When deleting a BST node with two children using its inorder successor, what must happen after copying the successor’s value?

A. Delete the entire right subtree  
B. Delete the left subtree  
C. Stop immediately  
D. Remove the successor’s original occurrence while preserving any right child it has  

25. In iterative inorder traversal for kth smallest, when should the visit counter increase?

A. Whenever a node is pushed onto the stack  
B. When a node is popped for its inorder visit  
C. Whenever a missing child is encountered  
D. Only when a leaf is found  

26. In general binary-tree LCA, only the left subtree returns a non-`None` result. What should the current non-target node return?

A. Always itself  
B. Always the root  
C. The left result  
D. The smaller-valued target  

27. A node’s left and right subtree heights are both `2`, measured in nodes. What are its height and its diameter candidate?

A. Height `3` nodes; diameter candidate `4` edges  
B. Height `4` nodes; diameter candidate `3` edges  
C. Height `2` nodes; diameter candidate `2` edges  
D. Height `4` nodes; diameter candidate `4` edges  

28. A balance-checking helper receives `-1` from its left child. What should it do?

A. Treat `-1` as the actual left height  
B. Replace it with `0`  
C. Return a normal height without checking the failure  
D. Propagate `-1` because the subtree is already unbalanced  

29. A client requests the “third-smallest price.” What should you clarify if prices can repeat?

A. Whether all prices can be converted to employee IDs  
B. Whether they mean the third item by price or the third distinct price  
C. Whether duplicates should always be deleted  
D. Whether the root must hold the third price  

30. Which verification gives the strongest evidence that BST deletion worked correctly?

A. The function completed without an exception  
B. The remaining tree is a valid BST  
C. The remaining values match expectations **and** the tree is a valid BST  
D. The root is still present  

Send your answers like:

```text
1.B
2.D
3.A
...
30.C
```
