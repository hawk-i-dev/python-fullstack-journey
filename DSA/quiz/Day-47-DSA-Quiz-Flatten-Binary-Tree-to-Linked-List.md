**Day 47 DSA Quiz — Flatten a Binary Tree to a Linked List**

Choose one answer per question. Flatten the tree using its **existing nodes in preorder**. Every left pointer must become `None`. Our function modifies the tree and returns `None`.

Use this tree for Questions 1–4:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

1. What should the flattened right-pointer chain be?

A. `3 → 2 → 4 → 1 → 5 → 6`  
B. `1 → 2 → 3 → 4 → 5 → 6`  
C. `1 → 5 → 6 → 2 → 4 → 3`  
D. `3 → 4 → 2 → 6 → 5 → 1`  

2. When processing node `1`, where does following right pointers from its left child stop?

A. Node `2`  
B. Node `3`  
C. Node `6`  
D. Node `4`  

3. What should happen to the original right subtree rooted at `5` before moving the left subtree?

A. Attach it to node `4`’s right pointer  
B. Delete it  
C. Attach it to node `3`’s left pointer  
D. Copy its values into newly created nodes  

4. Immediately after processing node `1`, which statement is correct?

A. The entire tree is already flattened  
B. Node `1` still has left child `2`  
C. Node `1` has right child `2`, and node `2` still has left child `3`  
D. Node `5` becomes the root  

5. What must be true of every left pointer after flattening?

A. It points to the previous node  
B. It is `None`  
C. It points to the root  
D. It preserves its original value  

6. Why is this assignment performed first?

```python
tail.right = current.right
```

A. To preserve the original right subtree before overwriting its connection  
B. To sort the node values  
C. To create a new root  
D. To remove duplicate nodes  

7. Which sequence correctly performs the rewiring?

A.
```python
current.left = None
current.right = current.left
tail.right = current.right
```

B.
```python
current.right = current.left
tail.right = current.right
current.left = None
```

C.
```python
tail.left = current.right
current.left = current.right
current.right = None
```

D.
```python
tail.right = current.right
current.right = current.left
current.left = None
```

8. The attachment point `tail` has no right child. Can it still have a left child?

A. No; it must always be a leaf  
B. No; the input would be invalid  
C. Yes; later iterations place that left subtree before the attached continuation  
D. Yes; but that left subtree must be deleted  

9. What is wrong with this call under our function contract?

```python
root = flatten(root)
```

A. It creates duplicate nodes  
B. It assigns `None` to `root` because the function returns `None`  
C. It prevents mutation  
D. It changes preorder into inorder  

10. What should happen when flattening an empty tree?

A. Raise `ValueError`  
B. Create a node containing zero  
C. Return an empty list  
D. Perform no work and return `None`  

11. What are the time and auxiliary-space complexities of the pointer-rewiring approach?

A. Time `O(n)`; auxiliary space `O(1)`  
B. Time `O(n²)`; auxiliary space `O(n)`  
C. Time `O(log n)`; auxiliary space `O(1)`  
D. Time `O(n)`; auxiliary space `O(n²)`  

12. Why does the nested right-pointer search not automatically make the algorithm `O(n²)`?

A. Nested loops always take linear time  
B. The inner loop executes only once overall  
C. The total right-chain search work across the traversal is linear  
D. Every binary tree has logarithmic height  

13. Which test best verifies the required transformation?

A. Check only that the root’s value is unchanged  
B. Check the same node objects appear in original preorder, every left pointer is `None`, and the chain ends correctly  
C. Check only that values are sorted  
D. Check only that the final node is a leaf  

14. A client asks to display a hierarchy as a flat list while retaining the original relationships. What should you recognize?

A. This mutating algorithm is always appropriate  
B. The original relationships can be discarded automatically  
C. Sorting values will preserve all relationships  
D. A traversal-based list may be appropriate instead of changing the tree’s connections  

15. What should happen if we flatten an already correctly flattened right-only chain again?

A. Its structure and node order remain unchanged  
B. Its order reverses  
C. Every node is copied  
D. All right pointers become `None`  

Send your answers like:

```text
1.B
2.D
...
15.A
```
