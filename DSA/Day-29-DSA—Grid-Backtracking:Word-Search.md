# Day 29 DSA — Grid Backtracking: Word Search

## 1. Problem

Given a grid of letters and a word, determine whether the word can be formed by moving between adjacent cells.

Allowed movements:

```text
Up
Down
Left
Right
```

A grid cell may be used only once in the same word path.

```python
board = [
    ["A", "B", "C", "E"],
    ["S", "F", "C", "S"],
    ["A", "D", "E", "E"],
]
```

Examples:

```python
word_exists(board, "ABCCED")  # True
word_exists(board, "SEE")     # True
word_exists(board, "ABCB")    # False
```

Diagonal movement is not allowed.

---

# 2. Feynman Explanation

Imagine a child tracing a word through letter tiles.

To find `"ABCCED"`:

```text
A → B → C
        ↓
        C
        ↓
D ← E
```

The child may move only to a neighboring tile.

After stepping on a tile, the child places a temporary footprint on it so the same tile cannot be used twice.

If the path fails:

```text
Remove the footprint
Return to the previous tile
Try another direction
```

That is grid backtracking.

---

# 3. The Important 20%

Remember these five steps:

```text
1. Try every cell as a starting point.
2. Reject out-of-bounds or non-matching cells.
3. Mark the current cell as visited.
4. Explore four directions.
5. Restore the cell before returning.
```

Core pattern:

```python
mark()
explore_up_down_left_right()
restore()
```

---

# 4. How to Recognize This Pattern

Look for phrases such as:

```text
Search inside a grid
Move between neighboring cells
Do not reuse a cell
Find whether a path exists
Return after finding one valid path
```

This usually suggests:

```text
DFS + backtracking + visited tracking
```

---

# 5. How to Think Before Coding

Ask these questions.

### What is the state?

```python
row
column
word_index
```

### What does `word_index` mean?

It identifies the character currently required:

```python
word[word_index]
```

### What makes a state invalid?

- Row is outside the board.
- Column is outside the board.
- Current cell does not match the required character.
- Current cell was already used.

### What creates success?

```python
word_index == len(word)
```

Every character has been matched.

### What must be undone?

The visited mark on the current cell.

---

# 6. Why Start from Every Cell?

The first character might occur anywhere.

For the word:

```text
SEE
```

we must try every cell containing `"S"`.

Therefore:

```python
for row in range(rows):
    for column in range(columns):
        if search(row, column, 0):
            return True
```

Stop immediately after finding one valid path.

---

# 7. Invalid-State Guard

Before using a cell, verify:

```python
if (
    row < 0
    or row >= rows
    or column < 0
    or column >= columns
    or board[row][column] != word[word_index]
):
    return False
```

This protects against:

- Invalid coordinates
- Wrong letters
- Previously visited cells marked with a sentinel

---

# 8. Visited Marking

Save the original character:

```python
original = board[row][column]
```

Temporarily mark the cell:

```python
board[row][column] = "#"
```

Explore neighboring cells.

Then restore it:

```python
board[row][column] = original
```

This temporary mutation must always be undone.

---

# 9. Why Restoration Matters

Suppose one path fails after using `"C"`.

Another starting point may legitimately need that `"C"`.

Without restoration, the grid remains damaged:

```text
"C" becomes "#"
```

Later searches would incorrectly think the cell is unavailable.

Backtracking means:

```text
Choose the cell
Explore
Restore the cell
```

---

# 10. Four Directions

From:

```text
(row, column)
```

the neighboring coordinates are:

```python
(row - 1, column)  # Up
(row + 1, column)  # Down
(row, column - 1)  # Left
(row, column + 1)  # Right
```

Diagonal movements would be:

```python
(row - 1, column - 1)
```

but they are not allowed in today’s requirement.

---

# 11. Dry Run

Board:

```text
A B C E
S F C S
A D E E
```

Word:

```text
ABCCED
```

Start at `(0, 0)`:

```text
board[0][0] = A
word[0] = A
```

Match and mark it.

Move right:

```text
B matches word[1]
```

Move right:

```text
C matches word[2]
```

Move down:

```text
C matches word[3]
```

Move down:

```text
E matches word[4]
```

Move left:

```text
D matches word[5]
```

Now:

```python
word_index == len(word)
```

Return:

```text
True
```

---

# 12. Python Implementation

```python
def word_exists(board, word):
    if word == "":
        return True

    if not board or not board[0]:
        return False

    rows = len(board)
    columns = len(board[0])

    def search(row, column, word_index):
        # Every character has been matched
        if word_index == len(word):
            return True

        # Invalid position or wrong/visited cell
        if (
            row < 0
            or row >= rows
            or column < 0
            or column >= columns
            or board[row][column] != word[word_index]
        ):
            return False

        original = board[row][column]

        # Choose: mark this cell as visited
        board[row][column] = "#"

        # Explore four directions
        found = (
            search(row - 1, column, word_index + 1)
            or search(row + 1, column, word_index + 1)
            or search(row, column - 1, word_index + 1)
            or search(row, column + 1, word_index + 1)
        )

        # Undo: restore the original character
        board[row][column] = original

        return found

    for row in range(rows):
        for column in range(columns):
            if search(row, column, 0):
                return True

    return False
```

---

# 13. Test the Implementation

```python
board = [
    ["A", "B", "C", "E"],
    ["S", "F", "C", "S"],
    ["A", "D", "E", "E"],
]

print(word_exists(board, "ABCCED"))
print(word_exists(board, "SEE"))
print(word_exists(board, "ABCB"))
```

Output:

```text
True
True
False
```

Verify the board was restored:

```python
expected = [
    ["A", "B", "C", "E"],
    ["S", "F", "C", "S"],
    ["A", "D", "E", "E"],
]

assert board == expected
```

---

# 14. Build the Code Step by Step

Handle special inputs:

```python
if word == "":
    return True

if not board or not board[0]:
    return False
```

Create dimensions:

```python
rows = len(board)
columns = len(board[0])
```

Define the recursive state:

```python
def search(row, column, word_index):
```

Write the success case:

```python
if word_index == len(word):
    return True
```

Write the invalid-state guard.

Mark the current cell:

```python
original = board[row][column]
board[row][column] = "#"
```

Explore four directions.

Restore:

```python
board[row][column] = original
```

Try every starting position.

---

# 15. Why the Algorithm Works

The outer loops try every possible starting cell.

For each matching cell, DFS explores every legal direction.

Visited marking prevents one path from using the same cell twice.

Restoration allows other paths to use the cell later.

Therefore:

- Every valid path can be explored.
- Invalid paths stop early.
- A cell is never reused within one path.
- The original grid is restored.

---

# 16. Edge Cases

## Empty word

```python
word_exists([["A"]], "")
```

Result:

```text
True
```

The empty word requires no cells.

Confirm this behavior with the client because some applications may reject empty search terms.

## Empty board

```python
word_exists([], "A")
```

Result:

```text
False
```

## One cell

```python
word_exists([["A"]], "A")   # True
word_exists([["A"]], "B")   # False
```

## Word longer than the number of cells

```python
word_exists([["A", "B"]], "ABA")
```

Result:

```text
False
```

A production implementation can reject this early.

## Reusing the same cell

```python
board = [["A", "B"]]
word = "ABA"
```

Result:

```text
False
```

The first `"A"` cannot be reused.

---

# 17. Useful Production Pruning

## Length check

```python
if len(word) > rows * columns:
    return False
```

## Frequency check

If the word requires more of a letter than the board contains, return early.

```text
Board contains two A characters.
Word requires three A characters.
Result must be False.
```

## Start from the rarer end

If the final character is rarer than the first character, reverse the word before searching.

This can reduce the number of starting branches.

These improve performance without changing correctness.

---

# 18. Complexity

Let:

```text
R = number of rows
C = number of columns
L = word length
```

Every cell may become a starting point.

Each matched character explores up to four directions.

```text
Time: O(R × C × 4ᴸ)
```

The recursive path contains at most `L` calls:

```text
Auxiliary space: O(L)
```

This excludes storage for the input board.

---

# 19. Common Mistakes

## Forgetting to restore the cell

Incorrect:

```python
board[row][column] = "#"
return found
```

Correct:

```python
board[row][column] = original
return found
```

## Returning before restoration

Incorrect:

```python
if search(...):
    return True
```

If restoration has not happened, the board remains modified.

Store the result first, restore, then return.

## Allowing diagonal movement

Do not add diagonal directions unless the requirement explicitly allows them.

## Reusing the same cell

Visited tracking must belong to the current path.

## Checking the board before bounds

Incorrect:

```python
board[row][column]
```

before verifying `row` and `column` can cause an index error.

## Assuming the grid is rectangular

Production code should validate that every row has the same length.

---

# 20. Temporary Mutation Versus a Visited Set

Today’s implementation temporarily changes the board and restores it.

Advantages:

- No separate visited collection
- Simple and memory efficient

Risks:

- Easy to forget restoration
- Unsafe if the same board is shared concurrently
- A sentinel such as `"#"` might be a valid board character

An alternative is:

```python
visited = set()
```

Store coordinates:

```python
visited.add((row, column))
```

Then undo:

```python
visited.remove((row, column))
```

Use a visited set when input mutation—even temporary—is unacceptable.

---

# 21. Real-World Assignment — Warehouse Code Path Validator

A warehouse uses a grid of shelf labels.

A product code is valid if it can be traced through adjacent shelves:

```text
Up, down, left or right
```

Each shelf may be used only once in a code path.

Implement:

```python
def product_code_exists(shelf_grid, product_code):
    pass
```

Example:

```python
shelf_grid = [
    ["A", "B", "C", "E"],
    ["S", "F", "C", "S"],
    ["A", "D", "E", "E"],
]
```

Expected:

```python
product_code_exists(shelf_grid, "ABCCED")  # True
product_code_exists(shelf_grid, "ABCB")    # False
```

---

# 22. Institute-Level Assignment

Requirements:

- Use DFS and backtracking.
- Support four directions.
- Prevent cell reuse.
- Restore visited cells.
- Explain the dry run.
- Calculate complexity.
- Add tests and a README.

Submission:

```text
day-29-word-search/
├── assignment.py
├── test_assignment.py
└── README.md
```

Viva questions:

1. Why must every cell be considered as a start?
2. Why mark a cell as visited?
3. Why restore it?
4. Why is bounds checking performed first?
5. What changes if diagonal movement is allowed?

---

# 23. Corporate-Level Requirements

Add:

- Type hints
- Docstrings
- Grid validation
- Rectangular-grid validation
- Character validation
- Word-length limits
- Grid-size limits
- Timeout or cancellation strategy
- Unit tests
- No observable input mutation
- Metrics for search size and duration

Suggested signature:

```python
def product_code_exists(
    shelf_grid: list[list[str]],
    product_code: str,
) -> bool:
    """Return whether a product code exists in the shelf grid."""
    pass
```

For shared application state, prefer a separate visited set rather than temporary grid mutation.

---

# 24. Client-Level Questions

Ask before implementation:

1. Are diagonal movements allowed?
2. Can a shelf be reused?
3. Is matching case-sensitive?
4. Can the code be empty?
5. Should the function return `True/False` or the actual path?
6. Do we need the first path or every valid path?
7. Will we search one code or thousands of codes?
8. Can the grid change during a search?
9. Is the grid always rectangular?
10. What characters may appear?
11. What are the maximum grid and code sizes?
12. Should searches time out?
13. Is result caching allowed?
14. Should partial matches be returned?

---

# 25. Requirement-Change Examples

## Client allows diagonal movement

Add four diagonal directions.

Branching increases, so performance may decrease.

## Client wants the actual path

Store and return coordinates:

```python
[(0, 0), (0, 1), ...]
```

## Client searches many words

Searching separately for every word may be inefficient.

A Trie-based multi-word search may be more appropriate.

## Client allows cell reuse

Visited tracking is no longer required, but cycles and maximum word length must still be controlled.

---

# 26. Acceptance Criteria

```text
ABCCED returns True.
SEE returns True.
ABCB returns False.

Movement is only up, down, left and right.
A cell is used at most once per path.
The original grid is unchanged after the call.
An empty grid returns False for a non-empty word.
A word longer than the number of cells returns False.
Invalid grid structures are rejected clearly.
```

---

# 27. Testing Checklist

Test:

```python
product_code_exists([], "A")
product_code_exists([["A"]], "")
product_code_exists([["A"]], "A")
product_code_exists([["A"]], "B")
product_code_exists([["A", "B"]], "ABA")
product_code_exists(board, "ABCCED")
product_code_exists(board, "SEE")
product_code_exists(board, "ABCB")
```

Check that input is restored:

```python
original = [row.copy() for row in board]

product_code_exists(board, "ABCCED")

assert board == original
```

Test malformed grids:

```python
[["A", "B"], ["C"]]
```

This is not rectangular.

---

# 28. Git and Review Expectations

Suggested commits:

```text
feat: implement warehouse product-code search
test: cover paths, reuse and edge cases
refactor: add validation and pruning
docs: document movement and client requirements
```

During review, verify:

- Are bounds checked before indexing?
- Is every temporary mark restored?
- Is cell reuse prevented?
- Is the original grid unchanged?
- Are movement rules explicit?
- Are input limits documented?
- Could a shared grid create concurrency issues?

---

# 29. Your Day 29 Assignment

Complete three levels.

### Level 1 — Algorithm

Implement `product_code_exists()`.

### Level 2 — Production quality

Add validation, pruning, tests, type hints and input limits.

### Level 3 — Requirement analysis

Document:

- Client clarification questions
- Acceptance criteria
- Mutation and concurrency risks
- How the design changes for many simultaneous words

## Day 29 takeaway

```text
Try every starting cell.

Match → mark → explore four directions → restore.

Never reuse a cell in the same path.

Backtracking must leave the grid exactly as it found it.
```
