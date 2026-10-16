**Day 40 DSA Quiz — Balanced Binary Tree**

Choose one answer per question. We measure **height in nodes**: an empty subtree has height `0`, and a leaf has height `1`. The helper returns `-1` for an unbalanced subtree.

1. A binary tree is height-balanced when:

A. Its left and right subtrees contain equal numbers of nodes  
B. Every node has either zero or two children  
C. At every node, the child subtree heights differ by at most `1`  
D. Only the root’s child heights are equal  

2. What should `is_balanced(None)` return?

A. `False`  
B. `True`  
C. `-1`  
D. `None`  

3. Both child subtrees are balanced, with heights `2` and `2`. What height should the helper return?

A. `3`  
B. `4`  
C. `2`  
D. `-1`  

4. Both child subtrees are balanced, with heights `3` and `1`. What should the helper return?

A. `4`  
B. `2`  
C. `3`  
D. `-1`  

5. Why do we use `abs(left_height - right_height)`?

A. To calculate the total node count  
B. To calculate the diameter  
C. To measure the difference regardless of which side is taller  
D. To make both heights equal  

6. Is this tree balanced?

```text
       1
      / \
     2   3
    /
   4
```

A. No, because the root’s child heights differ  
B. Yes, because every node’s child heights differ by at most `1`  
C. No, because node `2` has only one child  
D. Yes, because its values form a BST  

7. What does `-1` mean in our helper?

A. The subtree is unbalanced  
B. The subtree contains a negative value  
C. The subtree is empty  
D. The subtree has exactly one edge  

8. The left recursive call returns `-1`. What should happen next?

A. Use `-1` as the left height in the normal formula  
B. Replace it with `0`  
C. Return `True` because the left side is complete  
D. Immediately return `-1` to propagate the failure  

9. Is the following tree balanced?

```text
          1
         / \
        2   3
       /     \
      4       5
     /         \
    6           7
```

A. Yes, because both root subtrees have height `3`  
B. Yes, because it is symmetric  
C. No, because nodes below the root violate the height condition  
D. No, because repeated heights are forbidden  

10. Which formula returns the height of a balanced subtree?

A. `left_height + right_height`  
B. `1 + max(left_height, right_height)`  
C. `abs(left_height - right_height)`  
D. `1 + min(left_height, right_height)`  

11. Why is this public return statement incorrect?

```python
return bool(checked_height(root))
```

A. Height `0` becomes `False`, while the failure marker `-1` becomes `True`  
B. Python cannot convert integers into booleans  
C. Every positive height becomes `False`  
D. It changes the tree  

12. What is the worst-case time complexity of the one-pass solution?

A. `O(log n)`  
B. `O(n²)`  
C. `O(1)`  
D. `O(n)`  

13. What is the auxiliary space complexity of the recursive solution, where `h` is tree height?

A. `O(1)` for every tree  
B. `O(n²)`  
C. `O(h)`  
D. `O(log n)` for every tree  

14. Which statement about this tree is correct?

```text
      1
     / \
    9   2
```

A. It is both a valid BST and height-balanced  
B. It is height-balanced but is not a valid BST  
C. It is a valid BST but is not height-balanced  
D. It is neither a valid BST nor height-balanced  

15. A client asks you to make an unbalanced tree balanced. What should you explain about today’s function?

A. It only checks balance; changing the tree requires a separate balancing approach  
B. It automatically rotates nodes after finding `-1`  
C. It sorts the values and rebuilds the tree  
D. It deletes nodes until the heights match  

Send your answers like:

```text
1.C
2.B
...
15.A
```
