# Day 24 DSA — Backtracking Basics: Generate All Subsets

## 1. Problem

Given an array of unique numbers, generate every possible subset.

```python
nums = [1, 2]
```

Expected output:

```python
[
    [],
    [2],
    [1],
    [1, 2],
]
```

A subset may contain:

- No elements
- One element
- Several elements
- Every element

The order of the subsets does not matter.

---

## 2. What is backtracking?

Backtracking means:

```text
Choose something
Explore what happens
Undo the choice
Try another choice
```

It is recursion with decision-making and undoing.

The general pattern is:

```python
make_choice()
explore()
undo_choice()
```

---

## 3. Feynman explanation

Imagine you are packing toys for a trip.

For every toy, you have two choices:

```text
Leave it
Take it
```

For toys `[1, 2]`:

```text
Toy 1: leave or take?
Toy 2: leave or take?
```

Every complete sequence of choices creates one subset.

```text
Leave 1, leave 2 → []
Leave 1, take 2  → [2]
Take 1, leave 2  → [1]
Take 1, take 2   → [1, 2]
```

---

## 4. The important 20%

To generate subsets, remember only this:

> Every element creates two branches.

```text
Skip the element
Take the element
```

Core decision:

```python
backtrack(index + 1)       # Skip

path.append(nums[index])   # Take
backtrack(index + 1)
path.pop()                 # Undo
```

---

## 5. Decision tree

For:

```python
nums = [1, 2]
```

The decision tree is:

```text
                         []
                    skip 1 / \ take 1
                          /   \
                        []     [1]
                  skip 2 /\     /\ take 2
                        /  \   /  \
                      []  [2] [1] [1, 2]
```

Each leaf is one complete subset.

---

## 6. How to think before coding

Ask these questions.

### What is the choice?

For the current number:

```text
Skip it or take it
```

### What represents the current answer?

```python
path
```

### What moves us toward the end?

```python
index + 1
```

### When is one subset complete?

```python
index == len(nums)
```

### What must be undone?

If we add an element:

```python
path.append(nums[index])
```

we must later remove it:

```python
path.pop()
```

---

## 7. Variables required

```python
result  # stores all completed subsets
path    # stores the subset currently being built
index   # identifies the current element
```

Example:

```text
result = [[], [2]]
path = [1]
index = 2
```

---

## 8. Algorithm

```text
1. Create an empty result list.
2. Create an empty current path.
3. Start from index 0.
4. If index reaches the array length:
      copy path into result
      return
5. Explore the branch that skips the current element.
6. Add the current element to path.
7. Explore the branch that takes the current element.
8. Remove the current element from path.
9. Return result.
```

---

## 9. Dry run

Input:

```python
nums = [1, 2]
```

Start:

```text
path = []
index = 0
```

### Skip `1`

```text
path = []
index = 1
```

#### Skip `2`

```text
path = []
index = 2
```

The end is reached:

```text
result = [[]]
```

#### Take `2`

```text
path = [2]
index = 2
```

Save it:

```text
result = [[], [2]]
```

Undo the choice:

```text
path = []
```

### Take `1`

```text
path = [1]
index = 1
```

#### Skip `2`

Save:

```text
result = [[], [2], [1]]
```

#### Take `2`

```text
path = [1, 2]
```

Save:

```text
result = [[], [2], [1], [1, 2]]
```

Undo choices while returning:

```text
path = [1]
path = []
```

---

## 10. Python implementation

```python
def generate_subsets(nums):
    result = []
    path = []

    def backtrack(index):
        # One complete subset has been created
        if index == len(nums):
            result.append(path.copy())
            return

        # Choice 1: skip the current number
        backtrack(index + 1)

        # Choice 2: take the current number
        path.append(nums[index])
        backtrack(index + 1)

        # Undo the choice
        path.pop()

    backtrack(0)
    return result


print(generate_subsets([1, 2]))
```

Output:

```text
[[], [2], [1], [1, 2]]
```

---

## 11. Build the code from the algorithm

Create storage:

```python
result = []
path = []
```

Create the recursive function:

```python
def backtrack(index):
```

Save a completed subset:

```python
if index == len(nums):
    result.append(path.copy())
    return
```

Explore without the current element:

```python
backtrack(index + 1)
```

Choose the current element:

```python
path.append(nums[index])
```

Explore with it:

```python
backtrack(index + 1)
```

Undo the choice:

```python
path.pop()
```

Start recursion:

```python
backtrack(0)
```

---

## 12. Why do we need `path.copy()`?

Incorrect:

```python
result.append(path)
```

This stores a reference to the same list.

Later, when `path` changes, the saved entries also appear to change.

Correct:

```python
result.append(path.copy())
```

This saves a snapshot of the current subset.

Think of it like taking a photograph of `path`.

---

## 13. Why do we need `path.pop()`?

Suppose this branch creates:

```python
path = [1, 2]
```

When returning to try another branch, `2` should no longer be selected.

```python
path.pop()
```

changes:

```text
[1, 2] → [1]
```

Without `pop()`, choices from one branch leak into other branches.

This is the “backtrack” part of backtracking.

---

## 14. Why the algorithm works

Every number receives exactly two possible decisions:

```text
Skip
Take
```

For `n` numbers, all combinations of these decisions are explored.

Therefore, every possible subset is produced exactly once when the input contains unique values.

---

## 15. Number of subsets

Every element has two choices.

For `n` elements:

```text
Number of subsets = 2ⁿ
```

Examples:

```text
n = 0 → 1 subset
n = 1 → 2 subsets
n = 2 → 4 subsets
n = 3 → 8 subsets
n = 4 → 16 subsets
```

For three numbers:

```python
[1, 2, 3]
```

There are:

```text
2³ = 8 subsets
```

---

## 16. Edge cases

### Empty array

```python
generate_subsets([])
```

Output:

```python
[[]]
```

There is one subset: the empty subset.

### One element

```python
generate_subsets([7])
```

Output:

```python
[[], [7]]
```

### Three elements

```python
generate_subsets([1, 2, 3])
```

Output contains eight subsets:

```python
[
    [],
    [3],
    [2],
    [2, 3],
    [1],
    [1, 3],
    [1, 2],
    [1, 2, 3],
]
```

---

## 17. Complexity

For `n` elements, there are `2ⁿ` subsets.

Copying a subset can take up to `O(n)` time.

```text
Time: O(n × 2ⁿ)
```

Recursive depth:

```text
O(n)
```

Excluding the returned output:

```text
Auxiliary space: O(n)
```

The output itself requires:

```text
O(n × 2ⁿ)
```

in the worst case.

---

## 18. Common mistakes

### Forgetting to copy `path`

Incorrect:

```python
result.append(path)
```

Correct:

```python
result.append(path.copy())
```

### Forgetting to undo the choice

Incorrect:

```python
path.append(nums[index])
backtrack(index + 1)
```

Correct:

```python
path.append(nums[index])
backtrack(index + 1)
path.pop()
```

### Using the wrong base case

Incorrect:

```python
if index == len(nums) - 1:
```

This can save the path before deciding about the final element.

Correct:

```python
if index == len(nums):
```

### Returning after the skip branch

Incorrect:

```python
return backtrack(index + 1)
```

This prevents the take branch from running.

### Adding the wrong value

Incorrect:

```python
path.append(index)
```

Correct:

```python
path.append(nums[index])
```

---

## 19. Production considerations

Backtracking can produce enormous output.

```text
20 elements → 1,048,576 subsets
30 elements → 1,073,741,824 subsets
```

Before generating combinations in a real application, ask:

- Do we truly need every subset?
- Can we stop after finding one valid answer?
- Can we prune invalid branches early?
- Is there a safe input-size limit?
- Can results be generated lazily?

The algorithm may be correct but still unsuitable for large input.

---

## 20. Practice problem

Complete the function without using a library that directly generates combinations:

```python
def generate_subsets(nums):
    result = []
    path = []

    def backtrack(index):
        # Base case

        # Skip current number

        # Take current number

        # Undo the choice
        pass

    backtrack(0)
    return result
```

Expected results:

```python
generate_subsets([])       # [[]]
generate_subsets([5])      # [[], [5]]
generate_subsets([1, 2])   # 4 subsets
generate_subsets([1, 2, 3])  # 8 subsets
```

Additional challenge:

```python
def count_subsets(nums):
    # Return only the number of possible subsets.
    pass
```

Expected:

```python
count_subsets([1, 2, 3, 4])  # 16
```

## Day 24 takeaway

```text
For every element:
    skip it
    take it

After taking:
    undo with pop()
```

The backtracking template is:

```python
choose()
explore()
undo()
```
