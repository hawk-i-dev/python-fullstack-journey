**Day 43 DSA Quiz — Path Sum II: All Root-to-Leaf Paths**

Choose one answer per question. Return every matching **root-to-leaf** path as a list of values. Negative values and duplicates are allowed. Results follow left-first DFS order.

Use this tree for Questions 1–3:

```text
           5
          / \
         4   8
        /   / \
       11  13  4
      / \     / \
     7   2   5   1
```

1. What should `path_sum(root, 22)` return?

A. `True`  
B. `[[5, 4, 11, 2]]`  
C. `[[5, 4, 11, 2], [5, 8, 4, 5]]`  
D. `[[4, 11, 7], [5, 8, 4, 5]]`  

2. Why is `[5, 4]` not a valid result for target `9`?

A. It does not end at a leaf  
B. Paths must contain four nodes  
C. It contains a value smaller than the root  
D. Only right-side paths are allowed  

3. After completely exploring the root’s left subtree, what should the working path contain before entering node `8`?

A. `[]`  
B. `[5, 4, 11]`  
C. `[5, 8]`  
D. `[5]`  

4. Which pattern best describes today’s algorithm?

A. Sort → search → delete  
B. Choose → explore → undo  
C. Divide → discard → stop  
D. Enqueue → sort → dequeue  

5. What does `path.append(node.value)` do?

A. Permanently saves a matching answer  
B. Changes the tree’s connections  
C. Adds the current node to the route being explored  
D. Copies the entire path into `results`  

6. Why do we save `path.copy()`?

A. Later edits to the working path must not change the saved answer  
B. Copying automatically sorts the values  
C. Python cannot store nested lists otherwise  
D. Copying removes duplicate values  

7. What can happen if we use `results.append(path)` instead?

A. Every path becomes a tuple  
B. The tree becomes unbalanced  
C. The target changes automatically  
D. Saved entries reference the same working list and change as it changes  

8. What is the purpose of `path.pop()` before returning from a node?

A. Remove a node from the tree  
B. Restore the working path for the caller  
C. Remove the last saved answer  
D. Reset the target to zero  

9. Why don’t we explicitly undo `remaining -= node.value`?

A. Subtraction has no effect inside functions  
B. Every child receives the original target  
C. Each recursive call has its own local integer variable  
D. `path.pop()` also restores `remaining`  

10. Why must we continue after finding the first matching path?

A. The requirement is to collect every match  
B. The first match is always incorrect  
C. A tree must have exactly two matching paths  
D. The remaining target must become positive again  

11. What is wrong with this code inside the helper?

```python
if is_leaf and remaining == 0:
    results.append(path.copy())
    return
```

Assume the required `path.pop()` appears later in the function.

A. A leaf cannot match the target  
B. Copying the path is incorrect  
C. It changes the saved values to zero  
D. The early return skips cleanup and leaves the leaf in the working path  

12. What should an empty tree return?

A. `False`  
B. `[]`  
C. `[[]]`  
D. `None`  

13. What should target `3` return for this tree?

```text
      1
     / \
    2   2
```

A. `[[1, 2]]`, because equal value sequences are automatically merged  
B. `[[1, 2, 2]]`  
C. `[[1, 2], [1, 2]]`, because these are distinct node routes  
D. `[]`, because duplicate values are invalid  

14. Let `n` be the node count, `h` the height, and `S` the total number of values in the returned paths. Which complexity statement is correct?

A. Time `O(n + S)`, auxiliary space `O(h)`, output space `O(S)`  
B. Time `O(n)` regardless of output size, total space `O(1)`  
C. Time `O(log n)`, output space `O(h)`  
D. Time `O(S)` only, auxiliary space `O(n²)`  

15. A client only needs the number of matching root-to-leaf paths. What is the most appropriate adjustment?

A. Store every path and return the largest value  
B. Stop after the first match  
C. Deduplicate all node values before searching  
D. Count matching leaves without storing path lists  

Send your answers like:

```text
1.C
2.A
...
15.D
```
