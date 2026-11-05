**Day 46 DSA Quiz — Serialize and Deserialize a Binary Tree**

Choose one answer per question. Our format uses **preorder**, commas between tokens, and `#` for a missing child. Values are integers and may repeat.

1. What does serialization do?

A. Balances a binary tree  
B. Converts a tree into a representation that can be stored or transmitted  
C. Removes duplicate values  
D. Sorts every node  

2. Why must our format include missing-child markers?

A. To ensure all values are positive  
B. To reduce every tree to one token  
C. To make inorder traversal sorted  
D. To preserve the tree’s shape, not just its values  

3. What is the serialization of this tree?

```text
    1
   /
  2
```

A. `1,2,#,#,#`  
B. `1,#,2,#,#`  
C. `2,1,#,#,#`  
D. `1,2`  

4. What is the serialization of this tree?

```text
    1
     \
      2
```

A. `1,2,#,#,#`  
B. `2,#,1,#,#`  
C. `1,#,2,#,#`  
D. `1,#,#,2`  

5. What represents an empty tree?

A. `0`  
B. `#`  
C. An empty string  
D. `None,None`  

6. When the decoder reads `#`, what should the current `build()` call return?

A. `None`  
B. A node containing zero  
C. The previous node  
D. An empty list  

7. After reading an integer token and creating its node, what must the decoder do?

A. Build the right subtree, then the left  
B. Sort the remaining tokens  
C. Stop immediately  
D. Build the left subtree, then the right  

8. What does one `build()` call consume?

A. Exactly one token in every case  
B. All remaining tokens in every case  
C. Exactly the tokens describing one subtree  
D. Only the tokens at one tree level  

9. Why is `1,#` invalid?

A. Node values cannot equal `1`  
B. The root’s right subtree is missing from the encoding  
C. A tree cannot have missing children  
D. The encoding contains too many markers  

10. Why is `1,#,#,9` invalid?

A. The root must have two existing children  
B. The value `9` is too large  
C. The markers must appear before the root  
D. A complete tree has already been decoded, leaving an extra token  

11. Can this format reconstruct a tree containing repeated values?

A. Yes; the markers and traversal order preserve each node’s position  
B. No; every value must be unique  
C. Only if repeated values are removed first  
D. Only if the tree is a BST  

12. After deserializing a nonempty tree, why is `restored is original` normally `False`?

A. Deserialization always changes the root’s value  
B. The restored tree must have a different shape  
C. Deserialization creates new node objects  
D. Python cannot compare node references  

13. How many tokens does this format produce for a tree with `5` nodes?

A. `5`  
B. `11`  
C. `10`  
D. `6`  

14. Under the bounded-size integer assumption, which complexity statement is correct for the lesson’s implementation?

A. Time `O(log n)`; total auxiliary space `O(1)`  
B. Time `O(n²)`; recursive stack `O(n²)`  
C. Time `O(1)`; output space `O(n)`  
D. Time `O(n)`; token storage `O(n)` and recursive stack `O(h)`  

15. Which tree does this string describe?

```text
8,#,4,2,#,#,#
```

A. Root `8`, right child `4`, and `4`’s left child `2`  
B. Root `8`, left child `4`, and `4`’s right child `2`  
C. Root `8`, with children `4` and `2`  
D. Root `4`, with children `8` and `2`  

Send your answers like:

```text
1.B
2.D
...
15.A
```
