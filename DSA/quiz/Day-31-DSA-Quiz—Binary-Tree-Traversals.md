# Day 31 DSA Quiz — Binary Tree Traversals

### 1. Preorder, inorder and postorder are forms of:

A. Depth-first traversal  
B. Binary search  
C. Breadth-first traversal  
D. Sliding window  

### 2. What is the preorder traversal order?

A. Left → Root → Right  
B. Root → Left → Right  
C. Left → Right → Root  
D. Right → Root → Left  

### 3. What is the inorder traversal order?

A. Root → Left → Right  
B. Left → Right → Root  
C. Left → Root → Right  
D. Right → Left → Root  

### 4. What is the postorder traversal order?

A. Root → Right → Left  
B. Right → Root → Left  
C. Root → Left → Right  
D. Left → Right → Root  

Use this tree for Questions 5–7:

```text
        1
       / \
      2   3
     / \
    4   5
```

### 5. What is its preorder traversal?

A. `[1, 2, 4, 5, 3]`  
B. `[4, 2, 5, 1, 3]`  
C. `[4, 5, 2, 3, 1]`  
D. `[1, 3, 2, 5, 4]`  

### 6. What is its inorder traversal?

A. `[1, 2, 4, 5, 3]`  
B. `[4, 2, 5, 1, 3]`  
C. `[4, 5, 2, 3, 1]`  
D. `[3, 1, 5, 2, 4]`  

### 7. What is its postorder traversal?

A. `[1, 2, 4, 5, 3]`  
B. `[4, 2, 5, 1, 3]`  
C. `[4, 5, 2, 3, 1]`  
D. `[3, 5, 4, 2, 1]`  

### 8. What is the recursive base case for all three traversals?

A. `node.value == 0`  
B. `node.left == node.right`  
C. `result == []`  
D. `node is None`  

### 9. When does inorder traversal produce sorted values?

A. When the tree is a valid Binary Search Tree  
B. For every binary tree  
C. Only when the tree is empty  
D. Only when all values are identical  

### 10. In iterative preorder, why is the right child pushed before the left child?

A. Right must always be processed first  
B. The stack is LIFO, so the left child will be popped first  
C. It sorts the node values  
D. It reduces the time complexity to `O(1)`  

### 11. What is the time complexity of each complete traversal?

A. `O(log n)` for every tree  
B. `O(n²)`  
C. `O(n)`  
D. `O(1)`  

### 12. What is the recursive auxiliary-space complexity when `h` is tree height?

A. `O(n²)`  
B. `O(1)` for every tree  
C. `O(2ⁿ)`  
D. `O(h)`  

### 13. Why should this definition be avoided?

```python
def preorder(root, result=[]):
```

A. The same mutable list can be reused across separate calls  
B. Python does not allow default parameters  
C. A list cannot store node values  
D. It forces postorder traversal  

### 14. Which traversal is most suitable when children must be processed before deleting or summarizing their parent?

A. Preorder  
B. Postorder  
C. Inorder  
D. Level order only  

### 15. Before building a traversal service, what should be clarified with the client?

A. Only the variable names  
B. Only whether recursion is allowed  
C. Required order, returned data, tree validity, maximum size and list-versus-stream output  
D. The developer’s terminal theme  

Send your answers like this:

```text
1.A
2.B
3.C
...
15.C
```
