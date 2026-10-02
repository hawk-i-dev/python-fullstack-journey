# Day 30 DSA Quiz — Binary Tree Maximum Depth

### 1. What is a binary tree?

A. A structure where every node has exactly two children  
B. A structure where every node has at most two children  
C. A sorted array  
D. A structure containing only leaf nodes  

### 2. What is a leaf node?

A. A node with no children  
B. The first node in a tree  
C. A node with two parents  
D. An empty tree  

### 3. What should `max_depth(None)` return?

A. `-1`  
B. `1`  
C. `0`  
D. `None`  

### 4. What is the recursive formula for maximum depth?

A.

```python
left_depth + right_depth
```

B.

```python
1 + min(left_depth, right_depth)
```

C.

```python
max(left_depth, right_depth)
```

D.

```python
1 + max(left_depth, right_depth)
```

### 5. Why do we add `1`?

A. To count the current node  
B. To count an empty child  
C. To sort the tree  
D. To remove a leaf  

### 6. Why do we use `max()` rather than adding both subtree depths?

A. A longest root-to-leaf path travels through only one child side  
B. Both subtrees are always empty  
C. Addition is not supported in Python  
D. `max()` changes the tree structure  

### 7. What is the maximum depth of this tree?

```text
        3
       / \
      9   20
         /  \
        15   7
```

A. `2`  
B. `3`  
C. `4`  
D. `5`  

### 8. What is the maximum depth of a one-node tree?

A. `0`  
B. `1`  
C. `2`  
D. It depends on the node value  

### 9. What is the time complexity of recursive maximum-depth calculation?

A. `O(log n)` for every possible tree  
B. `O(n²)`  
C. `O(n)`  
D. `O(1)`  

### 10. What is the recursion-stack space complexity?

Let `h` be the tree height.

A. `O(h)`  
B. `O(n²)`  
C. `O(1)` for every tree  
D. `O(2ⁿ)`  

### 11. What is the stack space for a balanced binary tree?

A. `O(n)` always  
B. `O(log n)`  
C. `O(n²)`  
D. `O(1)`  

### 12. Why is an undo operation such as `path.pop()` unnecessary?

A. The function returns numeric subtree results without modifying shared path state  
B. Binary trees cannot use recursion  
C. The tree contains no children  
D. Python performs every undo automatically  

### 13. Why might an iterative traversal be safer for a very deep skewed tree?

A. It sorts nodes automatically  
B. It avoids exceeding Python’s recursion limit  
C. It changes maximum depth to minimum depth  
D. It visits no nodes  

### 14. What must be checked when an organization hierarchy comes from external relationship data?

A. Only employee-name length  
B. Cycles, duplicate IDs and missing manager references  
C. The developer’s editor settings  
D. Whether every manager has exactly two reports  

### 15. Which client question affects the meaning of the returned result?

A. Whether hierarchy depth is measured using nodes or edges  
B. Which font is used in the application  
C. Whether the developer uses tabs  
D. Which terminal theme is enabled  

Send your answers like this:

```text
1.B
2.A
3.C
...
15.A
```
