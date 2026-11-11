**Day 49 DSA Quiz — Maximum Width of a Binary Tree**

Choose one answer per question. Width includes missing positions **between the first and last existing nodes** at a level. Use root index `0`, left child `2*p + 1`, and right child `2*p + 2`.

1. A level contains nodes at positions `0`, `1`, and `3`. What is its positional width?

A. `3`  
B. `2`  
C. `4`  
D. `5`  

2. What is the difference between node count and positional width?

A. Node count includes every missing position  
B. Positional width includes internal gaps; node count includes only existing nodes  
C. They are always equal  
D. Positional width depends on node values  

3. A node has position `4`. What are its left and right child positions?

A. Left `8`, right `9`  
B. Left `5`, right `6`  
C. Left `9`, right `8`  
D. Left `9`, right `10`  

4. Which formula calculates a level’s positional width?

A. `last_position - first_position + 1`  
B. `last_position + first_position`  
C. `last_position - first_position`  
D. `number_of_existing_nodes`  

5. Why must the width formula include `+1`?

A. Every tree has an extra root  
B. Missing positions always add one node  
C. Both endpoint positions are included; a single-node level has width `1`  
D. It compensates for negative node values  

6. What is the maximum width of this tree?

```text
        1
       / \
      2   3
     /     \
    4       5
```

A. `2`  
B. `4`  
C. `3`  
D. `5`  

7. What should the BFS queue store?

A. Every possible missing position  
B. Only node values  
C. Only leaf nodes  
D. Existing nodes paired with position indexes  

8. Why do we avoid enqueueing all missing positions?

A. Position indexes already represent gaps, so storing missing entries wastes memory  
B. Missing positions do not affect width  
C. Queues cannot store `None`  
D. Every level must contain exactly two nodes  

9. A level’s raw positions are `[100, 101, 103]`. What are the normalized positions after subtracting the first position?

A. `[1, 2, 4]`  
B. `[0, 1, 2]`  
C. `[0, 1, 3]`  
D. `[100, 1, 3]`  

10. Why does subtracting the same base preserve width?

A. It makes every node adjacent  
B. It preserves the difference between the first and last positions  
C. It sorts node values  
D. It removes all genuine gaps  

11. Why must the same `base` be used for every node in a level?

A. Every node has the same value  
B. It ensures all normalized positions become zero  
C. It prevents children from entering the queue  
D. Different offsets would distort the nodes’ relative positions  

12. What is the maximum positional width of a right-only chain?

```text
1
 \
  2
   \
    3
     \
      4
```

A. `1`  
B. `4`  
C. `8`  
D. `0`  

13. Under constant-time index arithmetic, what are the complexities? Let `w` be the maximum count of **existing nodes** in any level.

A. Time `O(n²)`; auxiliary space `O(n²)`  
B. Time `O(log n)`; auxiliary space `O(1)`  
C. Time `O(n)`; auxiliary space `O(w)`  
D. Time `O(w)`; auxiliary space `O(1)`  

14. Which statement about normalization is correct?

A. It guarantees all indexes fit in a fixed-size integer  
B. It removes common offsets but cannot eliminate genuinely large positional gaps  
C. It changes the tree’s connections  
D. It makes positional width equal to node count  

15. A level has existing nodes at positions `[12, 13, 19]`. Which answer is correct?

A. Node count `3`, width `7`, normalized `[0, 1, 7]`  
B. Node count `8`, width `8`, normalized `[0, 1, 7]`  
C. Node count `3`, width `3`, normalized `[0, 1, 2]`  
D. Node count `3`, width `8`, normalized `[0, 1, 7]`  

Send your answers like:

```text
1.C
2.B
...
15.D
```
