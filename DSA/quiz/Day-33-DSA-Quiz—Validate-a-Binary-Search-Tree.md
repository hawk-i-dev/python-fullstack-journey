**Day 33 DSA Quiz — Validate a Binary Search Tree**

Choose one answer per question. Assume valid binary-tree structure, integer values, and a **strict BST: duplicates are not allowed**.

1. Which rule must every node in a BST satisfy?

A. Only its immediate left child must be smaller  
B. All values in its left subtree are smaller, and all values in its right subtree are greater  
C. Every left subtree must contain fewer nodes  
D. Both subtrees must have the same height  

2. Is this a valid BST?

```text
       10
      /  \
     5    15
         /  \
        6    20
```

A. Yes, because `6 < 15`  
B. Yes, because the root’s children are correctly placed  
C. No, because `6` is in `10`’s right subtree but is smaller than `10`  
D. No, because `15` has two children  

3. What is the purpose of carrying `lower` and `upper` bounds?

A. To preserve restrictions inherited from ancestors  
B. To count the tree’s levels  
C. To sort the tree’s values  
D. To ensure the tree is balanced  

4. A node has value `x` and allowed range `(lower, upper)`. What bounds should its **left child** receive?

A. `(x, upper)`  
B. `(None, x)` every time  
C. `(lower, upper)`  
D. `(lower, x)`  

5. What bounds should that node’s **right child** receive?

A. `(lower, x)`  
B. `(x, upper)`  
C. `(x, None)` every time  
D. `(upper, x)`  

6. Why does the helper return `True` when `node is None`?

A. A missing node is treated as having value zero  
B. It means the entire tree has been checked  
C. An empty subtree violates no BST rules  
D. Every missing node should be inserted later  

7. Which condition correctly rejects a value that violates an existing lower bound?

A. `node.value <= lower`  
B. `node.value > lower`  
C. `node.value == upper`  
D. `node.value < upper`  

8. Why should the code use `lower is not None` instead of simply `if lower`?

A. `None` is greater than all integers  
B. Negative integers are always falsy  
C. Bounds must always be positive  
D. Zero is a valid bound but is falsy in Python  

9. What should this function return for the following tree?

```text
       5
      / \
     3   5
```

A. `True`, because duplicates are always permitted on the right  
B. `False`, because today’s strict BST rule forbids duplicates  
C. `True`, because only left-side duplicates are forbidden  
D. `None`, because the result is ambiguous  

10. Why do we combine the recursive results using `and`?

```python
check(node.left, lower, node.value) and \
check(node.right, node.value, upper)
```

A. Only one subtree needs to be valid  
B. It sorts the left subtree before checking the right  
C. Both subtrees must satisfy their constraints  
D. It guarantees that both calls execute even if the first returns `False`  

11. What is the worst-case time complexity for validating a BST containing `n` nodes?

A. `O(n)`  
B. `O(log n)`  
C. `O(n²)`  
D. `O(1)`  

12. What is the auxiliary space complexity of the recursive bounds solution, where `h` is tree height?

A. `O(1)` for every tree  
B. `O(n²)`  
C. `O(log n)` for every tree  
D. `O(h)`  

13. Which inorder traversal could come from a valid strict BST?

A. `[2, 5, 5, 9]`  
B. `[-8, -3, 0, 7]`  
C. `[1, 6, 4, 9]`  
D. `[9, 7, 3, 1]`  

14. A valid BST forms a very long chain and the recursive validator raises a recursion-depth error. Which change preserves the bounds approach while avoiding recursive calls?

A. Check only the root and its children  
B. Remove the bounds checks  
C. Use an explicit stack containing `(node, lower, upper)`  
D. Return `True` whenever the tree is deep  

15. A client changes the requirement to “allow duplicates only in right subtrees.” What should you do?

A. Adjust bound inclusivity and test equal values on both sides and at deeper levels  
B. Accept every tree containing duplicate values  
C. Remove all equality checks without considering subtree placement  
D. Check only that inorder values are nondecreasing, which fully guarantees this placement rule  

Send your answers like:

```text
1.B
2.C
...
15.A
```
