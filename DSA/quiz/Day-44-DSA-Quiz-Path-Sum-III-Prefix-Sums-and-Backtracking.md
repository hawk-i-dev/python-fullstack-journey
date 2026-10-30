**Day 44 DSA Quiz — Path Sum III: Prefix Sums + Backtracking**

Choose one answer per question. Count **nonempty downward paths** that follow parent-to-child connections. Paths may start and end anywhere. Negative values and duplicates are allowed.

Use this tree for Questions 1–3:

```text
           10
          /  \
         5   -3
        / \    \
       3   2    11
      / \   \
     3  -2   1
```

1. How many downward paths sum to `8`?

A. `2`  
B. `3`  
C. `4`  
D. `1`  

2. Which path is allowed under today’s rules?

A. Left child `3 → 5 → 2`, moving upward and then downward  
B. `5 → 11`, skipping the connecting nodes  
C. `3 → -3`, moving between separate branches  
D. `5 → 2 → 1`, moving downward  

3. Why does `5 → 3` count even though it neither starts at the root nor ends at a leaf?

A. Today’s paths may start at any node and end at any descendant  
B. Every path must contain exactly two nodes  
C. Positive values can ignore path restrictions  
D. All paths through node `5` automatically count  

4. If the running root-to-current sum is `18` and the target is `8`, which earlier prefix sum do we need?

A. `26`  
B. `8`  
C. `10`  
D. `18`  

5. If `prefix_count[10] == 2` in Question 4, how many matching paths end at the current node?

A. `1`  
B. `2`  
C. `10`  
D. `0`  

6. Why do we initialize the map with `{0: 1}`?

A. To represent the prefix before the root and count root-starting paths  
B. To count an empty path as an answer  
C. To force every target to zero  
D. To indicate that the tree contains a zero-valued node  

7. What should the prefix map represent when counting paths ending at the current node?

A. Prefix sums from every node visited so far  
B. Only prefix sums from leaf nodes  
C. Prefix sums from unrelated sibling branches  
D. Earlier prefix sums on the current ancestor route  

8. Why do we count matches **before** adding the current prefix to the map?

A. Adding first would sort the map  
B. The current prefix is never needed by descendants  
C. Adding first could count an empty path when the target is zero  
D. Counting first prevents all duplicate prefix sums  

9. Why do we need a frequency map instead of a set?

A. Sets cannot store negative integers  
B. Multiple ancestor prefixes can have the same sum and represent different starting positions  
C. A set always takes `O(n²)` lookup time  
D. A map automatically validates the tree  

10. How many downward paths sum to `0` in this chain?

```text
0
 \
  0
   \
    0
```

A. `3`  
B. `1`  
C. `5`  
D. `6`  

11. Why must we decrement the current prefix frequency after exploring its children?

A. To prevent completed-branch prefixes from creating invalid cross-branch matches  
B. To erase matching paths already counted  
C. To remove the current node from the tree  
D. To restore the target to zero  

12. A prefix has frequency `3`. After leaving one node that contributed that prefix, what should its frequency become?

A. `0`, because the key must always be deleted  
B. `3`, because ancestor state never changes  
C. `2`, because only one occurrence is being removed  
D. `1`, because frequencies cannot exceed one  

13. What should `count_path_sum(None, 0)` return?

A. `1`  
B. `0`  
C. `None`  
D. `[[]]`  

14. Assuming average constant-time dictionary operations and removal of zero-count entries, what are the complexities?

A. Time `O(log n)`; auxiliary space `O(1)`  
B. Time `O(n²)`; auxiliary space `O(n²)`  
C. Time `O(h)`; auxiliary space `O(1)`  
D. Expected time `O(n)`; auxiliary space `O(h)`  

15. For this tree, how many downward paths sum to `1`?

```text
          1
         / \
       -1   1
       /     \
      1      -1
```

A. `5`: three single-node paths and two full root-to-leaf paths  
B. `6`: every node and one extra path  
C. `3`: only the single-node paths  
D. `2`: only the full root-to-leaf paths  

Send your answers like:

```text
1.B
2.D
...
15.A
```
