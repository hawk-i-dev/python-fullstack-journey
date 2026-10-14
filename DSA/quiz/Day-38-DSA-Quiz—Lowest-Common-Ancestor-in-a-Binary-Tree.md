**Day 38 DSA Quiz — Lowest Common Ancestor in a Binary Tree**

Choose one answer per question. Assume valid binary-tree structure. Targets are **node objects**, and the function returns `None` if either target is absent.

Use this tree for Questions 1–5. Numbers identify the existing nodes:

```text
           3
         /   \
        5     1
       / \   / \
      6   2 0   8
         / \
        7   4
```

1. Why can’t we use yesterday’s BST value-comparison shortcut here?

A. The tree contains too many nodes  
B. Its values do not follow BST ordering  
C. Binary trees cannot have ancestors  
D. The root must be the smallest value  

2. What is the LCA of nodes `7` and `4`?

A. Node `5`  
B. Node `3`  
C. Node `2`  
D. Node `7`  

3. What is the LCA of nodes `5` and `4`?

A. Node `5`  
B. Node `2`  
C. Node `3`  
D. Node `4`  

4. What is the LCA of nodes `6` and `8`?

A. Node `5`  
B. Node `1`  
C. Node `2`  
D. Node `3`  

5. While finding the LCA of `7` and `4`, node `5` receives `None` from its left subtree and node `2` from its right subtree. What should it return?

A. Node `5`  
B. Node `2`  
C. `None`  
D. Node `3`  

6. At a non-target node, both recursive child calls return non-`None` results. What should the helper return?

A. The smaller-valued result  
B. The left result only  
C. The current node  
D. The root of the entire tree  

7. What does this comparison check?

```python
node is p
```

A. Whether both references identify the exact same object  
B. Whether both nodes contain equal values  
C. Whether `node` is smaller than `p`  
D. Whether both nodes have the same children  

8. The tree already contains a node whose value is `4`. What happens if we pass a newly created `TreeNode(4)` as one target?

A. It automatically matches the existing node  
B. It replaces the existing node  
C. It is inserted before the search  
D. It is considered absent because it is a different object  

9. Why does our implementation check membership before running the compact LCA helper?

A. To balance the tree  
B. To avoid returning the one existing target when the other is missing  
C. To remove duplicate values  
D. To convert the tree into a BST  

10. What should happen when `p` and `q` reference the same existing node?

A. Return `None`  
B. Return its parent  
C. Return that node  
D. Raise an error  

11. What is the overall worst-case time complexity, including the two membership checks?

A. `O(n)`  
B. `O(n³)`  
C. `O(log n)` for every tree  
D. `O(1)`  

12. What is the auxiliary space complexity of the recursive implementation, where `h` is tree height?

A. `O(1)`  
B. `O(n²)`  
C. `O(log n)` for every tree  
D. `O(h)`  

13. Which approach avoids recursive calls while supporting LCA queries in an ordinary binary tree?

A. Compare target values and choose one branch  
B. Build a parent map, collect one target’s ancestors, then walk upward from the other  
C. Sort all node values and return the middle value  
D. Return the first leaf encountered  

14. A client’s organization allows employees to have multiple managers. What should you recognize?

A. The hierarchy is automatically a BST  
B. Employee IDs determine the closest common manager  
C. The structure may be a graph, and a unique LCA may not exist  
D. The binary-tree algorithm works unchanged in every case  

15. Which test verifies that the function returns the actual expected node object?

A. `assert answer is expected_node`  
B. `assert answer.value >= expected_node.value`  
C. `assert answer is not None`  
D. `assert answer.value in all_values`  

Send your answers like:

```text
1.B
2.C
...
15.A
```
