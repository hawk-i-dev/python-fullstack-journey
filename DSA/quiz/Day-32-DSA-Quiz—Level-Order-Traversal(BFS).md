**Day 32 DSA Quiz — Level Order Traversal (BFS)**

Choose one answer for each question. Assume all inputs are valid binary trees.

Use this tree for Questions 3, 4, and 10:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
```

1. Which data structure naturally supports BFS?

A. Stack  
B. Queue  
C. Set  
D. Heap  

2. What does FIFO mean?

A. First In, Final Out  
B. Final In, First Out  
C. First In, First Out  
D. First In, First Only  

3. What is the grouped level order traversal of the tree above?

A. `[[1], [2, 3], [4, 5, 6]]`  
B. `[[1], [2, 4, 5], [3, 6]]`  
C. `[[4, 5, 6], [2, 3], [1]]`  
D. `[[1, 2, 3, 4, 5, 6]]`  

4. At the start of level 2, the queue contains `[2, 3]`. After processing node `2` and adding its children, what is the queue? The front is on the left.

A. `[4, 5, 3]`  
B. `[3, 5, 4]`  
C. `[2, 3, 4, 5]`  
D. `[3, 4, 5]`  

5. Why do we save `level_size = len(queue)` before processing a level?

A. To sort the nodes  
B. To process only that level’s nodes, even as children join the queue  
C. To count every node in the tree immediately  
D. To prevent duplicate values  

6. Which Python operation removes the oldest node from a `deque` when nodes are added with `append()`?

A. `queue.pop()`  
B. `queue.remove()`  
C. `queue.popleft()`  
D. `queue.clear()`  

7. Where should `level = []` be created?

A. Inside the outer `while` loop, before the inner `for` loop  
B. Once, outside the function  
C. Inside the `for` loop, before processing each node  
D. After returning the result  

8. What should `level_order(None)` return?

A. `[None]`  
B. `[[]]`  
C. `0`  
D. `[]`  

9. Why does the queue store node objects instead of only their values?

A. Integers cannot be stored in a queue  
B. We need access to each node’s children  
C. Node objects are automatically sorted  
D. Values cannot be duplicated  

10. What should `level_sums(root)` return for the tree above?

A. `[1, 5, 15]`  
B. `[1, 2, 3, 4, 5, 6]`  
C. `[21]`  
D. `[1, 5, 9]`  

11. What is the time complexity of level order traversal for a tree containing `n` nodes?

A. `O(log n)`  
B. `O(n²)`  
C. `O(n)`  
D. `O(1)`  

12. Excluding the returned output, what is the auxiliary space complexity of the BFS implementation? Let `w` be the maximum number of nodes in any level.

A. `O(1)` for every tree  
B. `O(log n)` for every tree  
C. `O(n²)`  
D. `O(w)`  

13. Consider this tree:

```text
      5
     / \
    5   5
```

What should level order traversal return?

A. `[[5]]`, because repeated values are removed  
B. `[[5], [5, 5]]`, because these are three distinct nodes  
C. `[[5, 5, 5]]`, because equal values belong together  
D. An error, because trees cannot contain duplicate values  

14. A client requests “only level 3.” What should you clarify before implementing it?

A. Whether the root is considered level `0` or level `1`  
B. Whether Python allows nested lists  
C. Whether every node value is unique  
D. Whether BFS must use recursion  

15. A developer writes this code to collect one level:

```python
level = []

while queue:
    node = queue.popleft()
    level.append(node.value)

    if node.left is not None:
        queue.append(node.left)

    if node.right is not None:
        queue.append(node.right)

result.append(level)
```

What is the main problem?

A. It visits only the root  
B. It reverses every level  
C. It collects all reachable tree nodes into one list, losing level boundaries  
D. It always runs forever, even for a finite valid tree  

Send your answers like:

```text
1.B
2.C
...
15.C
```
