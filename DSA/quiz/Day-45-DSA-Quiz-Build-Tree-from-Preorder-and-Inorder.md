**Day 45 DSA Quiz — Build a Tree from Preorder and Inorder**

Choose one answer per question. Assume unique integer values. Our implementation returns `None` for two empty lists and raises `ValueError` for invalid traversals.

Use these sequences for Questions 1–4:

```python
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]
```

1. Which value becomes the root?

A. `9`  
B. `3`  
C. `20`  
D. `7`  

2. Which inorder section represents the root’s right subtree?

A. `[9]`  
B. `[3, 15, 20, 7]`  
C. `[20, 15, 7]`  
D. `[15, 20, 7]`  

3. Which value becomes the root of that right subtree?

A. `20`  
B. `15`  
C. `7`  
D. `9`  

4. What are the left and right children of node `20`?

A. Left `7`, right `15`  
B. Left `9`, right `7`  
C. Left `15`, right `7`  
D. Left `3`, right `15`  

5. What information does preorder provide during reconstruction?

A. The nodes are numerically sorted  
B. The next unconsumed value is the current subtree’s root  
C. The last value is always the overall root  
D. The tree is balanced  

6. What information does inorder provide after locating the current root?

A. Which values belong to its left and right subtrees  
B. The number of leaves without further processing  
C. The next root without using preorder  
D. Whether the tree must be a BST  

7. What does `build(left, right)` represent in our implementation?

A. Search for values between `left` and `right`  
B. Build a tree with exactly two nodes  
C. Sort the preorder list  
D. Reconstruct the subtree occupying those inclusive inorder indexes  

8. What should happen when `left > right`?

A. Raise an error for every such call  
B. Consume the next preorder value  
C. Return `None` without advancing the preorder cursor  
D. Create a node containing zero  

9. Why must we build the left subtree before the right subtree when using a shared forward preorder cursor?

A. Left subtrees always have smaller values  
B. Preorder lists the entire left subtree before the right subtree  
C. Right subtrees must contain more nodes  
D. Building right first would always be slower but still correct  

10. If the root’s inorder position is `middle`, which child intervals are correct?

A. Left `[left, middle]`; right `[middle, right]`  
B. Left `[left + 1, middle]`; right `[middle, right - 1]`  
C. Left `[middle + 1, right]`; right `[left, middle - 1]`  
D. Left `[left, middle - 1]`; right `[middle + 1, right]`  

11. Why use a value-to-inorder-index hashmap?

A. To avoid repeatedly searching inorder for each root  
B. To automatically balance the reconstructed tree  
C. To make duplicate values unambiguous  
D. To eliminate the need for subtree boundaries  

12. Consider:

```python
preorder = [1, 2, 3]
inorder  = [3, 1, 2]
```

Why are these inconsistent?

A. They contain different values  
B. Their lengths differ  
C. Inorder requires `3` in the root’s left subtree, but the next preorder root is `2`  
D. Preorder must always be numerically sorted  

13. Why does our implementation require unique values?

A. Binary trees cannot contain duplicates  
B. A value-to-index map needs an unambiguous position, and duplicates can make reconstruction ambiguous  
C. Negative numbers cannot be dictionary keys  
D. Unique values guarantee the tree is balanced  

14. Assuming average constant-time set and dictionary operations, what are the complexities of the validated implementation?

A. Time `O(n²)`; auxiliary space `O(1)`  
B. Time `O(log n)`; auxiliary space `O(log n)`  
C. Time `O(n)`; auxiliary space `O(1)`  
D. Expected time `O(n)`; auxiliary space `O(n)`, plus `O(n)` output-tree space  

15. Which verification best checks that reconstruction matches the inputs?

A. Regenerate both preorder and inorder traversals and compare them with the original lists  
B. Check only that the root has the correct value  
C. Check only that the tree contains the correct number of nodes  
D. Confirm that an inorder traversal is numerically sorted  

Send your answers like:

```text
1.B
2.D
...
15.A
```
