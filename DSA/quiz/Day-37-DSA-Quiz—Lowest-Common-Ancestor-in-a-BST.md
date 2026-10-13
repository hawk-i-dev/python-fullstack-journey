**Day 37 DSA Quiz — Lowest Common Ancestor in a BST**

Choose one answer per question. Assume a valid strict BST with unique integer values. Our function returns `None` if either target is missing.

Use this tree for Questions 1–6:

```text
           6
         /   \
        2     8
       / \   / \
      0   4 7   9
         / \
        3   5
```

1. What does “lowest” mean in lowest common ancestor?

A. The smallest numerical value  
B. The leftmost node  
C. The deepest shared ancestor  
D. The node with the fewest children  

2. What is the LCA of `3` and `5`?

A. `4`  
B. `2`  
C. `6`  
D. `3`  

3. What is the LCA of `2` and `5`?

A. `4`  
B. `5`  
C. `6`  
D. `2`  

4. What is the LCA of `0` and `9`?

A. `2`  
B. `6`  
C. `8`  
D. `9`  

5. What should `lca(root, 3, 10)` return under our contract?

A. Node `6`, because the values split there  
B. Node `8`, because it is close to `10`  
C. `None`, because `10` is absent  
D. A new node containing `10`  

6. What should `lca(root, 4, 4)` return?

A. The existing node containing `4`  
B. `None`, because the targets must differ  
C. The parent of node `4`  
D. The root  

7. Both target values are smaller than the current node’s value. What should you do?

A. Return the current node  
B. Move right  
C. Search both subtrees  
D. Move left  

8. Both target values are larger than the current node’s value. What should you do?

A. Move left  
B. Move right  
C. Return `None`  
D. Return the current node  

9. Why can we stop when two **existing** targets lie on opposite sides of the current node?

A. The current node must be the root  
B. The targets must be leaves  
C. Neither child subtree can contain both targets  
D. Their values must be consecutive  

10. Why do we calculate these values?

```python
low = min(a, b)
high = max(a, b)
```

A. To handle either target order consistently  
B. To sort the whole tree  
C. To verify that both targets exist  
D. To balance the tree  

11. After confirming both targets exist, which condition identifies the LCA?

A. `current.value < low`  
B. `current.value > high`  
C. `current.value == low + high`  
D. `low <= current.value <= high`  

12. Two existence searches and one LCA traversal each take `O(h)` time. What is the combined time complexity?

A. `O(h³)`  
B. `O(h)`  
C. `O(1)`  
D. `O(n²)`  

13. What is the auxiliary space complexity of the lesson’s iterative implementation?

A. `O(h)`  
B. `O(n)`  
C. `O(1)`  
D. `O(log n)`  

14. A client wants the closest common manager of two employees. Why can’t we automatically apply the BST value-comparison method to employee IDs?

A. Reporting relationships generally do not follow BST ordering by employee ID  
B. LCA can only be calculated for numerical roots  
C. Every manager must have exactly two employees  
D. Employee IDs cannot be compared  

15. The tree can change between checking target existence and finding the LCA. What should a production implementation consider?

A. Removing all existence checks  
B. Always returning the root  
C. Replacing missing targets with nearby values  
D. Coordinating reads and updates so the query sees a consistent tree  

Send your answers like:

```text
1.C
2.A
...
15.D
```
