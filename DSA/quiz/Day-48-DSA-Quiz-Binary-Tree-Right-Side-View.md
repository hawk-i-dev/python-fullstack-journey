**Day 48 DSA Quiz — Binary Tree Right Side View**

Choose one answer per question. Return the **rightmost existing node at each depth**, from top to bottom. Our BFS processes each level left to right.

Use this tree for Questions 1–3:

```text
        1
       / \
      2   3
       \   \
        5   4
```

1. What is the right-side view?

A. `[1, 2, 5]`  
B. `[1, 3, 4]`  
C. `[1, 2, 3, 5, 4]`  
D. `[4, 3, 1]`  

2. Which node should we save from each level during left-to-right BFS?

A. The first node  
B. The node with the largest value  
C. Every leaf  
D. The last node processed  

3. At the start of the third level, what is the queue’s order?

A. `[5, 4]`  
B. `[4, 5]`  
C. `[2, 3]`  
D. `[1, 5, 4]`  

4. What is the right-side view of this tree?

```text
        1
       / \
      2   3
       \
        5
         \
          6
```

A. `[1, 3]`  
B. `[1, 2, 5, 6]`  
C. `[1, 3, 5, 6]`  
D. `[1, 3, 6]`  

5. Why can following only right-child pointers fail?

A. Right pointers cannot be traversed  
B. Deeper visible nodes may belong to a left subtree  
C. Every tree must contain duplicate values  
D. Right-side view requires sorted values  

6. Why do we save `level_size = len(queue)` before processing a level?

A. To separate current-level nodes from children added during processing  
B. To prevent adding children  
C. To calculate the maximum node value  
D. To balance the tree  

7. Which child order supports saving the last node for the right-side view?

A. Right child before left child  
B. Smaller-valued child before larger-valued child  
C. Only enqueue right children  
D. Left child before right child  

8. Which condition selects the last node within the current level?

A. `index == 0`  
B. `index == level_size`  
C. `index == level_size - 1`  
D. `node.right is None`  

9. What is the right-side view here?

```text
      1
     / \
   100  2
```

A. `[1, 100]`  
B. `[1, 2]`  
C. `[100, 2]`  
D. `[1, 100, 2]`  

10. Which DFS strategy also produces the right-side view?

A. Visit left first and save every leaf  
B. Visit left first and save the first node at each depth  
C. Visit right first and save every node  
D. Visit right first and save the first node encountered at each depth  

11. In right-first DFS, why does this condition identify a newly reached depth?

```python
if depth == len(view):
```

A. No value has yet been saved for that depth  
B. Every node at that depth is a leaf  
C. The tree is balanced  
D. The current value equals its depth  

12. What should the right-side view of an empty tree be?

A. `[None]`  
B. `[0]`  
C. `[]`  
D. `False`  

13. What should this tree return?

```text
      5
     / \
    5   5
```

A. `[5]`, because duplicates must be removed  
B. `[5, 5]`, because each level contributes a value  
C. `[5, 5, 5]`, because every node must be saved  
D. An error, because values must be unique  

14. Let `n` be node count, `w` maximum level width, and `h` tree height. Which statement is correct, excluding output storage?

A. BFS takes `O(log n)` time and `O(1)` space  
B. Both approaches always use `O(1)` space  
C. BFS takes `O(n²)` time; DFS takes `O(n)` time  
D. Both take `O(n)` time; BFS uses `O(w)` space and recursive DFS uses `O(h)`  

15. A client asks for the highest-priority item at each level. What should you recognize?

A. This requires a different selection rule from choosing the rightmost node  
B. The rightmost node always has the highest priority  
C. Following right pointers always satisfies the requirement  
D. Sorting the final right-side-view list is sufficient  

Send your answers like:

```text
1.B
2.D
...
15.A
```
