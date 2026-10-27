**Day 41 DSA Quiz — Binary Tree Maximum Path Sum**

Choose one answer per question. A path follows connected nodes without revisiting or branching. It must contain **at least one node**. Our function raises `ValueError` for an empty tree.

Use this tree for Questions 1–4:

```text
        -10
        /  \
       9    20
           / \
          15  7
```

1. What is the maximum path sum?

A. `34`  
B. `42`  
C. `25`  
D. `51`  

2. Which path produces that maximum?

A. `9 → -10 → 20 → 15`  
B. `-10 → 20 → 7`  
C. `9 → 15 → 7`  
D. `15 → 20 → 7`  

3. At node `20`, what gain should be returned to its parent?

A. `35`  
B. `42`  
C. `27`  
D. `20`  

4. Why does the function return `best` instead of only `gain(root)`?

A. The root’s gain always includes every node  
B. The root’s gain is always negative  
C. The best complete path may occur below the root  
D. Returning a gain would modify the tree  

5. What does this expression mean?

```python
max(0, gain(node.left))
```

A. Change negative node values to zero  
B. Omit the left branch if its total contribution is harmful  
C. Skip every negative node while keeping its descendants connected  
D. Delete the left subtree  

6. After clamping child gains to zero or above, which formula gives the complete candidate through a node?

A. `node.value + left_gain + right_gain`  
B. `1 + max(left_gain, right_gain)`  
C. `left_gain + right_gain`  
D. `node.value + min(left_gain, right_gain)`  

7. Which formula gives the extendable gain returned to the parent?

A. `node.value + left_gain + right_gain`  
B. `max(left_gain, right_gain)`  
C. `1 + left_gain + right_gain`  
D. `node.value + max(left_gain, right_gain)`  

8. Why can’t we return both child branches as the contribution to the parent?

A. A path cannot contain duplicate values  
B. Right children cannot contribute  
C. Connecting the parent and both branches would create a fork  
D. Both child gains must always be negative  

9. A node has value `5`. Its children return gains `-3` and `8`. What are its complete candidate and returned gain?

A. Candidate `10`; returned gain `13`  
B. Candidate `13`; returned gain `13`  
C. Candidate `13`; returned gain `8`  
D. Candidate `10`; returned gain `10`  

10. A node has value `4`. Its children return gains `6` and `2`. What are its complete candidate and returned gain?

A. Candidate `10`; returned gain `12`  
B. Candidate `12`; returned gain `12`  
C. Candidate `8`; returned gain `6`  
D. Candidate `12`; returned gain `10`  

11. What is the maximum path sum of this tree?

```text
       -3
       / \
     -5  -2
```

A. `-2`  
B. `0`  
C. `-3`  
D. `-10`  

12. Why is initializing `best = 0` incorrect under our contract?

A. Python cannot compare zero with negative integers  
B. It prevents recursion from reaching leaves  
C. It can select an empty path over every valid path in an all-negative tree  
D. It makes every positive answer smaller  

13. What is the maximum path sum here?

```text
       5
      /
    -2
    /
   10
```

A. `15`, because we can skip the connecting node `-2`  
B. `13`, because the connected path includes `-2`  
C. `10`, because negative nodes always block a path  
D. `8`, because the root cannot be included  

14. What are the worst-case time and auxiliary-space complexities, where `n` is node count and `h` is height?

A. Time `O(log n)`; space `O(1)`  
B. Time `O(n²)`; space `O(n²)`  
C. Time `O(n)`; space `O(1)` for every tree  
D. Time `O(n)`; space `O(h)`  

15. A client changes the requirement to “find the maximum root-to-leaf sum.” What should you recognize?

A. The path constraints changed; optional-branch logic must be reconsidered because the path must reach a leaf  
B. The existing any-to-any algorithm always returns the correct answer unchanged  
C. The root can still be omitted whenever it is negative  
D. All negative values should be removed first  

Send your answers like:

```text
1.B
2.D
...
15.A
```
