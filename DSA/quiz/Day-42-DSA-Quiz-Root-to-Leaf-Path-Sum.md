**Day 42 DSA Quiz — Root-to-Leaf Path Sum**

Choose one answer per question. A valid path must start at the **root** and end at a **leaf**. Negative values are allowed. An empty tree returns `False`.

Use this tree for Questions 1–4:

```text
           5
          / \
         4   8
        /   / \
       11  13  4
      / \      \
     7   2      1
```

1. Which path matches target `22`?

A. `5 → 8 → 13`  
B. `5 → 4 → 11 → 2`  
C. `4 → 11 → 7`  
D. `5 → 4 → 11`  

2. Why does target `9` return `False`, even though `5 + 4 = 9`?

A. Paths cannot contain two nodes  
B. Every path must go through the right subtree  
C. The target must be greater than the root  
D. Node `4` on that path is not a leaf  

3. What should `has_path_sum(root, 26)` return?

A. `True`, because `5 → 8 → 13` matches  
B. `False`, because only the left subtree is searched  
C. `True`, because an internal node has value `26`  
D. `False`, because the path has fewer than four nodes  

4. Which condition correctly identifies a leaf?

A. At least one child is missing  
B. The node’s value is zero  
C. Both children are missing  
D. Both children exist  

5. What does the function calculate after reaching an existing node?

A. `remaining = target + node.value`  
B. `remaining = target - node.value`  
C. `remaining = max(target, node.value)`  
D. `remaining = target - 1`  

6. At a leaf, what confirms success?

A. The remaining target is zero after subtracting the leaf’s value  
B. The leaf’s value is positive  
C. The remaining target is negative  
D. The path contains at least three nodes  

7. Why do we combine the child results using `or`?

A. Both subtrees must contain matching paths  
B. The tree must be balanced  
C. The right subtree must always be searched first  
D. One successful root-to-leaf route is enough  

8. What should `has_path_sum(None, 0)` return?

A. `True`, because the target is zero  
B. `0`  
C. `False`, because there is no root-to-leaf path  
D. An empty list  

9. What is the result for target `2` in this tree?

```text
      5
     /
   -3
```

A. `False`, because the remainder becomes negative at `5`  
B. `True`, because `5 + (-3) = 2` and `-3` is a leaf  
C. `False`, because negative values are forbidden  
D. `True`, because every two-node tree matches  

10. Why is this shortcut unsafe when negative values are allowed?

```python
if remaining < 0:
    return False
```

A. Python cannot compare negative integers  
B. Every negative remainder guarantees success  
C. It changes the node’s value  
D. A later negative node can bring the remaining target back to zero  

11. What does target `5` return for this tree?

```text
      5
     /
    2
```

A. `False`: the only root-to-leaf path sums to `7`  
B. `True`: the root already matches  
C. `True`: the root has a missing right child  
D. `False`: paths cannot contain positive values  

12. During the search for target `22` in the main tree, node `11` passes remaining target `2` to node `7`. That branch fails. What target does sibling node `2` receive?

A. `-5`, the failed branch’s remainder  
B. `22`, the original target  
C. `2`, because sibling calls have separate local state  
D. `0`, because the first child was processed  

13. What are the worst-case time and auxiliary-space complexities, where `n` is node count and `h` is height?

A. Time `O(log n)`; space `O(1)`  
B. Time `O(n)`; space `O(h)`  
C. Time `O(n²)`; space `O(n²)`  
D. Time `O(h)`; space `O(1)`  

14. Why can’t we directly reuse Day 41’s optional-branch logic, `max(0, gain)`, for today’s problem?

A. Today’s tree must be a BST  
B. Today’s values must all be negative  
C. Today’s function must modify the tree  
D. Today’s route must satisfy an exact target and continue from the root to a leaf  

15. A client says a route is complete at an “approved terminal state,” which may still have child records. What should you clarify?

A. Whether business completion differs from being a structural leaf  
B. Whether all terminal states can be deleted  
C. Whether every node can be treated as a leaf  
D. Whether negative values should automatically be ignored  

Send your answers like:

```text
1.B
2.D
...
15.A
```
