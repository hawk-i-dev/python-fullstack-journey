# Day 29 DSA Quiz — Grid Backtracking and Word Search

### 1. Which algorithmic pattern is commonly used for Word Search?

A. Prefix sum  
B. DFS with backtracking  
C. Binary search  
D. Monotonic stack  

### 2. Which movements are allowed in today’s problem?

A. Up, down, left and right  
B. Diagonal movements only  
C. Left and right only  
D. Any cell on the board  

### 3. Why must every grid cell be considered as a possible starting point?

A. The first word character may appear anywhere  
B. Every cell must be included in the answer  
C. It sorts the board  
D. It reduces the board size  

### 4. Which values describe the recursive state?

A. Target, total and result  
B. Left, right and middle  
C. Row, column and word index  
D. Start, end and minimum  

### 5. When has the complete word been matched?

A. `word_index == 0`  
B. `word_index == len(word)`  
C. `row == column`  
D. `len(board) == len(word)`  

### 6. Why is the current cell marked as visited?

A. To prevent it from being reused in the same path  
B. To permanently remove it from the board  
C. To sort the grid  
D. To enable diagonal movement  

### 7. What must happen after exploring the current cell’s neighbors?

A. Delete the row  
B. Restore the cell’s original value  
C. Clear the entire board  
D. Increase the number of columns  

### 8. Why should bounds be checked before accessing `board[row][column]`?

A. To avoid invalid-index access  
B. To reverse the word  
C. To count the rows  
D. To modify the board  

### 9. What does this early check accomplish?

```python
if len(word) > rows * columns:
    return False
```

A. It ensures every cell is reused  
B. It rejects a word that cannot fit without cell reuse  
C. It allows diagonal movement  
D. It sorts the word  

### 10. What is the worst-case time complexity?

Let:

```text
R = rows
C = columns
L = word length
```

A. `O(R + C + L)`  
B. `O(log L)`  
C. `O(R × C × 4ᴸ)`  
D. `O(L²)`  

### 11. What is the recursive auxiliary-space complexity?

A. `O(L)`  
B. `O(1)`  
C. `O(R × C × 4ᴸ)`  
D. `O(log L)`  

### 12. Why might a separate `visited` set be preferable in a corporate application?

A. It permanently changes the board  
B. It avoids temporary mutation of shared or concurrent input  
C. It enables the same cell to be reused  
D. It removes input validation  

### 13. If the client wants the coordinates of the successful route, what must change?

A. Store and return the path’s cell coordinates  
B. Remove backtracking  
C. Sort every board row  
D. Return only the word length  

### 14. If the client needs to search thousands of words on the same grid, what may be more appropriate?

A. Running an unrelated binary search  
B. A Trie-based multi-word search  
C. Removing visited tracking  
D. Converting the grid into a stack  

### 15. Which requirement must be clarified before implementation?

A. Whether diagonal movement, cell reuse, case sensitivity or actual path output is required  
B. The developer’s terminal color  
C. The preferred font for variable names  
D. Whether Git commits should contain vowels  

Send your answers like this:

```text
1.B
2.A
3.A
...
15.A
```
