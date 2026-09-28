# Day 25 DSA — Backtracking: Generate All Permutations

## 1. Problem

Given an array of unique numbers, generate every possible ordering.

```python
nums = [1, 2, 3]
```

Expected output:

```python
[
    [1, 2, 3],
    [1, 3, 2],
    [2, 1, 3],
    [2, 3, 1],
    [3, 1, 2],
    [3, 2, 1],
]
```

A permutation uses every element exactly once, but changes their order.

---

## 2. Subsets versus permutations

For subsets:

```text
Do we take this number?
```

For permutations:

```text
Which unused number should occupy this position?
```

Example:

```text
Subsets of [1, 2]:
[], [1], [2], [1, 2]

Permutations of [1, 2]:
[1, 2], [2, 1]
```

Order matters in permutations:

```text
[1, 2] != [2, 1]
```

---

## 3. Feynman explanation

Imagine three children waiting for three chairs.

```text
Children: 1, 2, 3
Chairs:   _  _  _
```

For the first chair, choose any unused child.

If child `1` sits first:

```text
1  _  _
```

For the second chair, choose between `2` and `3`.

```text
1  2  3
1  3  2
```

Then undo the choices and let another child sit first.

That produces every possible seating order.

---

## 4. The important 20%

Remember this backtracking loop:

```text
For every unused number:
    choose it
    mark it used
    explore
    unmark it
    undo the choice
```

Code pattern:

```python
for index in range(len(nums)):
    if used[index]:
        continue

    used[index] = True
    path.append(nums[index])

    backtrack()

    path.pop()
    used[index] = False
```

---

## 5. How to think before coding

### What is the choice?

Choose one number that has not already been used.

### What represents the current arrangement?

```python
path
```

### How do we prevent reusing a number?

```python
used[index]
```

### When is a permutation complete?

```python
len(path) == len(nums)
```

### What must be undone?

```python
path.pop()
used[index] = False
```

---

## 6. Variables required

```python
result  # all completed permutations
path    # permutation currently being built
used    # whether each array position is already selected
```

For:

```python
nums = [1, 2, 3]
```

Initially:

```python
path = []
used = [False, False, False]
```

If we choose `2`:

```python
path = [2]
used = [False, True, False]
```

---

## 7. Algorithm

```text
1. Create result.
2. Create an empty path.
3. Create a used list filled with False.
4. If path contains every number:
      save a copy of path
      return
5. Loop through every index.
6. If that index is already used, skip it.
7. Otherwise:
      mark the index used
      add its number to path
      recursively build the next position
      remove the number from path
      mark the index unused
8. Return result.
```

---

## 8. Decision tree for `[1, 2]`

```text
                         []
                       /    \
                 choose 1   choose 2
                    |          |
                   [1]        [2]
                    |          |
                 choose 2   choose 1
                    |          |
                 [1, 2]      [2, 1]
```

Leaves:

```python
[1, 2]
[2, 1]
```

---

## 9. Dry run

Input:

```python
nums = [1, 2]
```

Initial state:

```text
path = []
used = [False, False]
```

### Choose `1`

```text
path = [1]
used = [True, False]
```

`1` cannot be chosen again because it is marked as used.

Choose `2`:

```text
path = [1, 2]
used = [True, True]
```

The path length equals the input length:

```text
result = [[1, 2]]
```

Undo `2`:

```text
path = [1]
used = [True, False]
```

Undo `1`:

```text
path = []
used = [False, False]
```

### Choose `2`

```text
path = [2]
used = [False, True]
```

Choose `1`:

```text
path = [2, 1]
used = [True, True]
```

Save it:

```text
result = [[1, 2], [2, 1]]
```

---

## 10. Python implementation

```python
def generate_permutations(nums):
    result = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        # One complete permutation
        if len(path) == len(nums):
            result.append(path.copy())
            return

        for index in range(len(nums)):
            # This position is already present in path
            if used[index]:
                continue

            # Choose
            used[index] = True
            path.append(nums[index])

            # Explore
            backtrack()

            # Undo
            path.pop()
            used[index] = False

    backtrack()
    return result


print(generate_permutations([1, 2, 3]))
```

Output:

```text
[
    [1, 2, 3],
    [1, 3, 2],
    [2, 1, 3],
    [2, 3, 1],
    [3, 1, 2],
    [3, 2, 1],
]
```

---

## 11. Build the code from the algorithm

Create the storage:

```python
result = []
path = []
used = [False] * len(nums)
```

Write the completed-permutation base case:

```python
if len(path) == len(nums):
    result.append(path.copy())
    return
```

Try every number:

```python
for index in range(len(nums)):
```

Ignore numbers already selected:

```python
if used[index]:
    continue
```

Choose a number:

```python
used[index] = True
path.append(nums[index])
```

Explore the remaining positions:

```python
backtrack()
```

Undo both parts of the choice:

```python
path.pop()
used[index] = False
```

---

## 12. Why use a `used` list?

Consider:

```python
nums = [1, 2, 3]
path = [1]
```

Without tracking usage, recursion could select `1` again:

```text
[1, 1, 1]
```

But a permutation must use each input position exactly once.

The `used` list prevents this:

```python
used = [True, False, False]
```

---

## 13. Why use indexes instead of checking values?

Avoid this approach:

```python
if nums[index] in path:
    continue
```

Problems:

- Searching `path` costs `O(n)`.
- It behaves incorrectly when duplicate values are allowed.

The `used` list tracks input positions directly in `O(1)` time.

---

## 14. Why do we undo twice?

A choice changes two pieces of state:

```python
used[index] = True
path.append(nums[index])
```

Therefore, both changes must be reversed:

```python
path.pop()
used[index] = False
```

If either undo step is missing, another branch receives incorrect state.

A useful rule is:

```text
Undo every change made during choose.
```

---

## 15. Why the algorithm works

At each position, the algorithm tries every currently unused number.

After selecting one number, it recursively fills the next position.

When the path contains `n` numbers, one valid complete ordering has been created.

Because every unused option is explored at every position, all permutations are generated.

---

## 16. Number of permutations

For `n` unique elements:

```text
Number of permutations = n!
```

Examples:

```text
0! = 1
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
10! = 3,628,800
```

The growth is extremely fast.

---

## 17. Edge cases

### Empty array

```python
generate_permutations([])
```

Output:

```python
[[]]
```

There is one way to arrange zero elements: the empty arrangement.

### One element

```python
generate_permutations([7])
```

Output:

```python
[[7]]
```

### Two elements

```python
generate_permutations([1, 2])
```

Output:

```python
[[1, 2], [2, 1]]
```

---

## 18. Duplicate-value warning

Today’s solution assumes all input values are unique.

For this input:

```python
[1, 1, 2]
```

the basic algorithm produces duplicate permutations because the two `1` values occupy different indexes.

Handling duplicate values requires:

- Sorting the input
- Skipping equivalent choices at the same decision level

We will treat that as a separate advanced problem.

---

## 19. Complexity

For `n` unique elements:

```text
Number of permutations: n!
```

Copying each completed permutation costs `O(n)`.

Therefore:

```text
Time: O(n × n!)
```

The recursive depth and current path use:

```text
Auxiliary space: O(n)
```

The returned output requires:

```text
Output space: O(n × n!)
```

---

## 20. Common mistakes

### Forgetting `path.copy()`

Incorrect:

```python
result.append(path)
```

Correct:

```python
result.append(path.copy())
```

### Forgetting to skip used positions

Incorrect:

```python
for index in range(len(nums)):
    path.append(nums[index])
```

This allows the same position to be selected repeatedly.

### Forgetting to unmark the index

Incorrect:

```python
path.pop()
```

Correct:

```python
path.pop()
used[index] = False
```

### Unmarking before recursion

Incorrect:

```python
used[index] = True
used[index] = False
backtrack()
```

The choice must remain active while exploring its branch.

### Returning inside the loop

Incorrect:

```python
for index in range(len(nums)):
    ...
    return
```

This explores only the first branch.

---

## 21. Production considerations

Permutation generation becomes expensive very quickly:

```text
8!  = 40,320
10! = 3,628,800
12! = 479,001,600
```

Before generating them, ask:

- Do we need every permutation?
- Can we stop after finding one valid arrangement?
- Can invalid branches be rejected early?
- Should the input length be limited?
- Can results be processed lazily instead of stored?

Correct code can still exhaust memory or CPU.

---

## 22. Practice problem

Complete this implementation:

```python
def generate_permutations(nums):
    result = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        # Base case

        # Try every input position

        # Skip used positions

        # Choose

        # Explore

        # Undo
        pass

    backtrack()
    return result
```

Expected results:

```python
generate_permutations([])       # [[]]
generate_permutations([5])      # [[5]]
generate_permutations([1, 2])   # 2 permutations
generate_permutations([1, 2, 3])  # 6 permutations
```

## Day 25 takeaway

```text
Subsets:
Skip or take the current element.

Permutations:
Choose any unused element for the current position.
```

Core permutation template:

```python
for each unused choice:
    mark used
    choose
    explore
    undo
    mark unused
```
