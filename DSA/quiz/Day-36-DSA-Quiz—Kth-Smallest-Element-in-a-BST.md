**Day 36 DSA Quiz — Kth Smallest Element in a BST**

Choose one answer per question. Assume a valid strict BST with integer values, no duplicates, and positions starting at **1**.

Use this tree for Questions 1–4:

```text
         8
        / \
       3   10
      / \
     1   6
        / \
       4   7
```

1. What is the third-smallest value?

A. `3`  
B. `6`  
C. `4`  
D. `8`  

2. Which traversal visits these values in increasing order?

A. Inorder: Left → Root → Right  
B. Preorder: Root → Left → Right  
C. Postorder: Left → Right → Root  
D. Level order: BFS  

3. What does `k = 5` mean?

A. Search for the value `5`  
B. Return the fifth level  
C. Return any five nodes  
D. Return the fifth value in ascending order  

4. What should `kth_smallest(root, 7)` return?

A. `7`  
B. `10`  
C. `8`  
D. `None`  

5. In the iterative inorder solution, when should we increase `visited`?

A. Whenever a node is pushed  
B. Whenever we move right  
C. Immediately after popping a node for processing  
D. Only when reaching a leaf  

6. Why is counting nodes when they are pushed incorrect?

A. A parent can be pushed before its smaller left descendants  
B. A stack cannot store tree nodes  
C. Pushing automatically sorts the stack  
D. Only right children can be pushed  

7. The stack is `[8, 3, 1]`, shown bottom → top. Which node is popped next?

A. `8`  
B. `3`  
C. All three together  
D. `1`  

8. Why does the outer loop use `or`?

```python
while current is not None or stack:
```

A. Both conditions must always be true  
B. Either `current` or the stack may still contain work  
C. It ensures each node is visited twice  
D. It prevents invalid values of `k`  

9. After processing a node and finding that `visited != k`, what is the next step?

A. Restart from the root  
B. Clear the stack  
C. Move to that node’s right child  
D. Return `None` immediately  

10. Under the lesson’s function contract, what happens when `k` exceeds the number of nodes?

A. Raise `ValueError` after exhausting the traversal  
B. Return the largest value  
C. Return `-1`  
D. Insert more nodes  

11. Which bound describes the time for a valid `k`, where `h` is tree height?

A. Always `O(1)`  
B. Always `O(log n)`  
C. Always `O(k)`, regardless of tree shape  
D. `O(h + k)`, with worst-case `O(n)`  

12. What is the auxiliary space complexity of the iterative stack solution?

A. `O(1)` for every tree  
B. `O(h)`  
C. `O(n²)`  
D. `O(k)` for every tree  

13. Which traversal supports finding the kth **largest** value?

A. Left → Root → Right  
B. Root → Right → Left  
C. Right → Root → Left  
D. Left → Right → Root  

14. A client asks for the third-cheapest product, but the BST is ordered by product ID. What should you recognize?

A. Inorder traversal ranks IDs; ranking by price requires an appropriate price-ordering approach  
B. The third-smallest ID always has the third-smallest price  
C. Changing `k` from `3` to `2` fixes the problem  
D. BFS automatically ranks products by price  

15. For frequent ranking queries, what extra information can support selection in `O(h)` time?

A. Only the tree’s maximum value  
B. Only each node’s parent value  
C. The number of previous searches  
D. Each node’s subtree size, maintained after updates  

Send your answers like:

```text
1.C
2.A
...
15.D
```
