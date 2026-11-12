# Day 50 DSA Quiz — Complete Binary Tree

15 MCQs • One correct answer per question.  
Trace the positions—not the node values.

**1. What makes a binary tree complete?**

A. Every node has two children  
B. Its values are sorted  
C. Every level except possibly the last is full, and the last is filled left to right  
D. Its left and right subtrees contain equal numbers of nodes  

**2. Why do we use BFS for the gap-checking approach?**

A. It automatically sorts node values  
B. It visits positions level by level, left to right  
C. It uses no additional memory  
D. It visits all leaves before their parents  

**3. Is this tree complete?**

```text
    10
   /
  5
```

A. No, because the root has only one child  
B. No, because the values are different  
C. Only if `5` has a left child  
D. Yes, the last level is left-packed  

**4. Is this tree complete?**

```text
    10
      \
       15
```

A. No, an empty left position comes before an occupied right position  
B. Yes, because it is a valid BST  
C. Yes, because it has only two levels  
D. Only if `15` is greater than `10`  

**5. What does `seen_gap = True` mean?**

A. A duplicate value was found  
B. The current level is full  
C. A missing position has already been processed  
D. The tree is definitely incomplete  

**6. What should happen when BFS processes its first `None`?**

A. Return `False` immediately  
B. Set `seen_gap = True` and continue  
C. Enqueue two more `None` entries  
D. Reset the queue  

**7. After a gap has been processed, a real node is removed from the queue. What should we do?**

A. Ignore the node  
B. Reset `seen_gap` to `False`  
C. Check whether its value is larger than the root  
D. Return `False`  

**8. What is the problem with this code in the gap-checking algorithm?**

```python
if node.left:
    queue.append(node.left)

if node.right:
    queue.append(node.right)
```

A. It hides missing positions needed to detect gaps  
B. It adds children in reverse order  
C. It changes the tree  
D. It creates duplicate nodes  

**9. What should `is_complete(None)` return under our lesson’s contract?**

A. `None`  
B. `False`  
C. Raise `ValueError`  
D. `True`  

**10. What does the function return for this tree?**

```text
        1
       / \
      2   3
     /   /
    4   6
```

A. `True`, because both subtrees have equal height  
B. `False`, because `2.right` is missing before node `6`  
C. `True`, because the last level contains only two nodes  
D. `False`, because the values are not consecutive  

**11. Why must the gap flag remain active across levels?**

A. Node values can repeat  
B. Each level needs its own independent result  
C. A node on a later level can violate a gap found earlier  
D. BFS cannot identify level boundaries  

**12. What is the result for this tree?**

```text
    1
   /
  2
 /
3
```

A. `False`: the root’s missing right position is processed before node `3`  
B. `True`: every node is as far left as possible  
C. `True`: all values are different  
D. `False`: complete trees must always have an odd number of nodes  

**13. What are the worst-case time and auxiliary-space complexities of our BFS implementation?**

A. Time `O(log n)`, space `O(1)`  
B. Time `O(n²)`, space `O(n)`  
C. Time `O(n)`, space `O(1)`  
D. Time `O(n)`, space `O(n)`  

**14. A client asks whether a tree is a valid min-heap. Our completeness check returns `True`. What remains to be checked?**

A. Every node must have exactly two children  
B. Every parent’s value must be less than or equal to its children’s values  
C. Every leaf must be at exactly the same depth  
D. Every value must be unique  

**15. For the diagnostic assignment, which node is the first real node processed after a gap?**

```text
        10
       /  \
      5    15
     / \     \
    3   7     20
```

A. `3`  
B. `7`  
C. `20`  
D. `15`  

Send your answers like:

```text
1.C
2.B
...
15.C
```
