**Day 34 DSA Quiz — Search & Insert in a BST**

Choose one answer per question. Assume a valid strict BST. Duplicate insertion leaves the tree unchanged.

Use this tree for Questions 1–5:

```text
           8
         /   \
        3     10
       / \      \
      1   6      14
         / \     /
        4   7   13
```

1. At node `8`, which direction should you move when searching for `7`?

A. Right  
B. Left  
C. Both directions  
D. Stop immediately  

2. Which sequence of values is inspected when searching for `7`?

A. `8 → 10 → 14 → 7`  
B. `8 → 3 → 1 → 7`  
C. `8 → 3 → 6 → 7`  
D. `8 → 6 → 7`  

3. What should `search_bst(root, 5)` return?

A. `None`  
B. `False` in every implementation  
C. The node containing `4`  
D. A newly created node containing `5`  

4. Where should a new value `5` be inserted?

A. As the left child of `4`  
B. As the right child of `7`  
C. As the left child of `10`  
D. As the right child of `4`  

5. What happens when we insert `6` again under today’s duplicate policy?

A. A second `6` is added to the left  
B. The tree remains unchanged  
C. The original `6` is deleted  
D. The root changes to `6`  

6. Why can BST search choose only one subtree at each comparison?

A. Every BST is perfectly balanced  
B. The other subtree has already been visited  
C. BST ordering excludes the other subtree as a possible location  
D. Each node has only one child  

7. What does this statement do?

```python
current = current.right
```

A. Moves the local reference to the right child  
B. Creates a new right child  
C. Deletes the current node  
D. Changes the root’s value  

8. What does this statement do when the right child is missing?

```python
current.right = TreeNode(value)
```

A. Moves `current` without changing the tree  
B. Searches the entire right subtree  
C. Balances the tree  
D. Creates and connects a new right child  

9. Why should callers use this assignment?

```python
root = insert_bst(root, value)
```

A. It sorts all existing values  
B. It captures the new root when the tree was empty  
C. It guarantees the tree is balanced  
D. It prevents every possible duplicate policy  

10. What is the time complexity of searching or inserting along one path, where `h` is tree height?

A. `O(1)` for every tree  
B. `O(n²)` for every tree  
C. `O(h)`  
D. `O(log n)` for every tree  

11. What is the auxiliary space complexity of the iterative search implementation?

A. `O(1)`  
B. `O(h)`  
C. `O(n)`  
D. `O(n²)`  

12. What can happen when ordinary BST insertion receives these values in order?

```python
[1, 2, 3, 4, 5]
```

A. The tree automatically becomes balanced  
B. All values become children of the root  
C. The tree rejects sorted input  
D. The tree becomes a chain of right children  

13. What is the total worst-case time to build an ordinary BST by inserting `n` distinct values in ascending order?

A. `O(n)`  
B. `O(n²)`  
C. `O(log n)`  
D. `O(1)`  

14. A client wants an existing product’s price updated when its ID is inserted again. The tree is ordered by product ID. What should happen on an equal ID?

A. Replace the ID with the new price  
B. Create another node with the same ID regardless of policy  
C. Update the price field while preserving the ordering key  
D. Delete the entire subtree  

15. After inserting `[20, 10, 30, 5, 15, 25, 35]`, what should `search_path(root, 26)` return? The function records every inspected value, including searches that fail.

A. `[20, 30, 25]`  
B. `[20, 10, 15]`  
C. `[20, 30, 35, 26]`  
D. `[]`  

Send your answers like:

```text
1.B
2.C
...
15.A
```
