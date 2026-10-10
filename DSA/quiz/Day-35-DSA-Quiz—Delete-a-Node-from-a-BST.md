**Day 35 DSA Quiz — Delete a Node from a BST**

Choose one answer per question. Assume a valid strict BST with no duplicates. For two-child deletion, use the **inorder successor**.

1. What should `delete_bst(root, key)` return?

A. Only the deleted value  
B. The updated subtree root  
C. Always the original root  
D. The tree’s height  

2. What replaces a deleted leaf node?

A. Its parent  
B. A node containing zero  
C. `None`  
D. Its inorder successor in every case  

3. A node containing `10` has only a right child containing `14`. What replaces `10` when it is deleted?

A. The subtree rooted at `14`  
B. `None`  
C. A new node containing `10`  
D. The parent’s left subtree  

4. When deleting a node with two children, where do we find its inorder successor?

A. The largest value in its right subtree  
B. The smallest value in its left subtree  
C. Its immediate right child in every case  
D. The smallest value in its right subtree  

Use this tree for Questions 5–7:

```text
        5
       / \
      3   9
         /
        7
         \
          8
```

5. Which value replaces `5` when deleting it?

A. `9`  
B. `7`  
C. `3`  
D. `8`  

6. After copying the successor’s value into the root, what must happen next?

A. Delete the entire right subtree  
B. Remove node `3`  
C. Delete the successor’s original occurrence from the right subtree  
D. Stop, because deletion is complete  

7. After deleting `5`, what happens to node `8`?

A. It becomes the left child of `9`  
B. It is deleted with `7`  
C. It becomes the right child of `3`  
D. It becomes the root  

8. Why do we write this assignment?

```python
root.left = delete_bst(root.left, key)
```

A. To sort the left subtree  
B. To copy every node  
C. To guarantee a balanced tree  
D. To reconnect the updated left subtree to its parent  

9. What should happen when the key does not exist?

A. Delete the closest value  
B. Leave the tree unchanged  
C. Return `None` for the entire nonempty tree  
D. Insert the missing key  

10. What is the worst-case time complexity of this deletion algorithm, where `h` is tree height?

A. `O(1)`  
B. `O(h²)`  
C. `O(h)`  
D. `O(log n)` regardless of tree shape  

11. What is the auxiliary space complexity of the recursive implementation?

A. `O(h)`  
B. `O(1)` for every tree  
C. `O(n²)`  
D. `O(log n)` for every tree  

12. Which statement about the inorder successor is correct?

A. It always has two children  
B. It always has a left child  
C. It must be a leaf  
D. It has no left child but may have a right child  

13. What is the result of this code?

```python
root = TreeNode(5)
root = delete_bst(root, 5)
```

A. `root.value` becomes `0`  
B. `root` becomes `None`  
C. The root remains unchanged  
D. The function must raise an error  

14. A node stores a product ID and its price. The BST is ordered by product ID. Why could copying only the successor’s ID during deletion cause a bug?

A. BSTs cannot store prices  
B. Copying an integer always breaks ordering  
C. The copied ID could become associated with the deleted product’s old price  
D. Every product must have the same price  

15. Which verification best checks that deletion worked correctly?

A. Confirm the expected remaining values and verify BST validity  
B. Check only that the root is not `None`  
C. Check only that the tree remains a valid BST  
D. Check only that the function did not raise an exception  

Send your answers like:

```text
1.B
2.C
...
15.A
```
