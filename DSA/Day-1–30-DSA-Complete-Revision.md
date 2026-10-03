# Day 1–30 DSA Complete Revision

You have now covered the essential array, hashing, pointer, window, stack, queue, linked-list, binary-search, recursion, backtracking, grid and tree patterns.

## Phase 1 — Arrays, Hashing and Two Pointers

| Day | Concept | Problem | Recognition clue | Complexity |
|---|---|---|---|---|
| 1 | Set and Big-O | Contains Duplicate | “Have I seen this before?” | `O(n)` time, `O(n)` space |
| 2 | Hash map lookup | Two Sum | Find a complement | `O(n)` time, `O(n)` space |
| 3 | Frequency map | Valid Anagram | Compare character counts | `O(n)` time, `O(k)` space |
| 4 | Two pointers | Valid Palindrome | Compare both ends | `O(n)` time, `O(1)` space |
| 5 | Sorted two pointers | Two Sum II | Sorted array and pair target | `O(n)` time, `O(1)` space |

### Day 1 — Contains Duplicate

```python
def contains_duplicate(nums):
    seen = set()

    for number in nums:
        if number in seen:
            return True
        seen.add(number)

    return False
```

Core idea:

```text
Set lookup is approximately O(1).
```

### Day 2 — Two Sum

```python
def two_sum(nums, target):
    seen = {}

    for index, number in enumerate(nums):
        required = target - number

        if required in seen:
            return [seen[required], index]

        seen[number] = index

    return []
```

Core idea:

```text
Store information that will help a future element.
```

### Day 3 — Valid Anagram

```python
def is_anagram(first, second):
    if len(first) != len(second):
        return False

    frequency = {}

    for character in first:
        frequency[character] = frequency.get(character, 0) + 1

    for character in second:
        if character not in frequency:
            return False

        frequency[character] -= 1

        if frequency[character] < 0:
            return False

    return True
```

### Day 4 — Valid Palindrome

```python
def is_palindrome(text):
    left = 0
    right = len(text) - 1

    while left < right:
        if text[left] != text[right]:
            return False

        left += 1
        right -= 1

    return True
```

### Day 5 — Two Sum II

```python
def two_sum_sorted(nums, target):
    left = 0
    right = len(nums) - 1

    while left < right:
        current = nums[left] + nums[right]

        if current == target:
            return [left, right]

        if current < target:
            left += 1
        else:
            right -= 1

    return []
```

---

# Phase 2 — Sliding Window and Prefix Sum

| Day | Concept | Problem | Recognition clue | Complexity |
|---|---|---|---|---|
| 6 | Fixed sliding window | Maximum sum of size `k` | Contiguous window with fixed length | `O(n)` |
| 7 | Variable sliding window | Longest unique substring | Expand and shrink based on a rule | `O(n)` |
| 8 | Prefix sum | Range Sum Query | Many range-sum requests | Build `O(n)`, query `O(1)` |
| 9 | Prefix sum + hash map | Subarray Sum Equals K | Count subarrays with exact sum | `O(n)` time, `O(n)` space |

### Day 6 — Fixed Sliding Window

```python
def maximum_window_sum(nums, k):
    if k <= 0 or k > len(nums):
        raise ValueError("Invalid window size")

    window_sum = sum(nums[:k])
    maximum = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]
        maximum = max(maximum, window_sum)

    return maximum
```

Core idea:

```text
Add the incoming element.
Remove the outgoing element.
```

### Day 7 — Variable Sliding Window

```python
def longest_unique_substring(text):
    last_position = {}
    left = 0
    longest = 0

    for right, character in enumerate(text):
        if character in last_position:
            left = max(left, last_position[character] + 1)

        last_position[character] = right
        longest = max(longest, right - left + 1)

    return longest
```

Core idea:

```text
Expand right.
Shrink or move left when the condition breaks.
```

### Day 8 — Prefix Sum

```python
def build_prefix(nums):
    prefix = [0]

    for number in nums:
        prefix.append(prefix[-1] + number)

    return prefix


def range_sum(prefix, left, right):
    return prefix[right + 1] - prefix[left]
```

### Day 9 — Subarray Sum Equals K

```python
def count_subarrays(nums, target):
    prefix_frequency = {0: 1}
    prefix_sum = 0
    count = 0

    for number in nums:
        prefix_sum += number
        required = prefix_sum - target
        count += prefix_frequency.get(required, 0)
        prefix_frequency[prefix_sum] = (
            prefix_frequency.get(prefix_sum, 0) + 1
        )

    return count
```

Core equation:

```text
current_prefix - previous_prefix = target
previous_prefix = current_prefix - target
```

---

# Phase 3 — Stack and Queue

| Day | Concept | Problem | Recognition clue | Complexity |
|---|---|---|---|---|
| 10 | Stack | Valid Parentheses | Match recent opening symbol | `O(n)` |
| 11 | Auxiliary stack | Min Stack | Current minimum in `O(1)` | Every operation `O(1)` |
| 12 | Monotonic stack | Daily Temperatures | Next greater value | `O(n)` |
| 13 | Queue | RecentCounter | Remove expired events | Amortized `O(1)` |

### Day 10 — Valid Parentheses

```python
def valid_parentheses(text):
    stack = []
    matching = {
        ")": "(",
        "]": "[",
        "}": "{",
    }

    for character in text:
        if character in "([{":
            stack.append(character)
        elif not stack or stack.pop() != matching[character]:
            return False

    return not stack
```

### Day 11 — Min Stack

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.minimums = []

    def push(self, value):
        self.stack.append(value)

        if not self.minimums or value <= self.minimums[-1]:
            self.minimums.append(value)

    def pop(self):
        value = self.stack.pop()

        if value == self.minimums[-1]:
            self.minimums.pop()

        return value

    def get_min(self):
        return self.minimums[-1]
```

### Day 12 — Daily Temperatures

```python
def daily_temperatures(temperatures):
    answer = [0] * len(temperatures)
    stack = []

    for index, temperature in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < temperature:
            previous = stack.pop()
            answer[previous] = index - previous

        stack.append(index)

    return answer
```

### Day 13 — RecentCounter

```python
from collections import deque


class RecentCounter:
    def __init__(self):
        self.requests = deque()

    def ping(self, time):
        self.requests.append(time)

        while self.requests[0] < time - 3000:
            self.requests.popleft()

        return len(self.requests)
```

---

# Phase 4 — Linked List and Binary Search

| Day | Concept | Problem | Recognition clue | Complexity |
|---|---|---|---|---|
| 14 | Pointer manipulation | Reverse Linked List | Change node connections | `O(n)` time, `O(1)` space |
| 15 | Binary search | Search Insert Position | Sorted input and exact position | `O(log n)` |
| 16 | Binary search on answer | Koko Eating Bananas | Find minimum feasible value | `O(n log M)` |
| 17 | Binary search on answer | Ship Packages | Find minimum valid capacity | `O(n log S)` |

### Day 14 — Reverse Linked List

```python
def reverse_list(head):
    previous = None
    current = head

    while current:
        next_node = current.next
        current.next = previous
        previous = current
        current = next_node

    return previous
```

Remember:

```text
Save next.
Reverse current.
Move previous.
Move current.
```

### Day 15 — Search Insert Position

```python
def search_insert(nums, target):
    left = 0
    right = len(nums)

    while left < right:
        middle = (left + right) // 2

        if nums[middle] < target:
            left = middle + 1
        else:
            right = middle

    return left
```

### Day 16 — Koko Eating Bananas

```python
from math import ceil


def minimum_speed(piles, hours):
    left = 1
    right = max(piles)

    while left < right:
        speed = (left + right) // 2
        required_hours = sum(
            ceil(pile / speed)
            for pile in piles
        )

        if required_hours <= hours:
            right = speed
        else:
            left = speed + 1

    return left
```

Pattern:

```text
Search space contains possible answers.
A feasibility function returns True or False.
```

### Day 17 — Ship Packages Within D Days

```python
def ship_within_days(weights, days):
    left = max(weights)
    right = sum(weights)

    def can_ship(capacity):
        required_days = 1
        current_load = 0

        for weight in weights:
            if current_load + weight > capacity:
                required_days += 1
                current_load = 0

            current_load += weight

        return required_days <= days

    while left < right:
        capacity = (left + right) // 2

        if can_ship(capacity):
            right = capacity
        else:
            left = capacity + 1

    return left
```

---

# Phase 5 — Recursion Fundamentals

| Day | Concept | Problem | Key recursive idea | Complexity |
|---|---|---|---|---|
| 18 | Basic recursion | Factorial | `n × factorial(n-1)` | `O(n)` |
| 19 | Memoization | Fibonacci | Cache repeated subproblems | `O(n)` |
| 20 | Array recursion | Sum of Array | Current value + remaining sum | `O(n)` |
| 21 | Array recursion | Find Maximum | Compare current with right maximum | `O(n)` |
| 22 | Recursive two pointers | Reverse String | Swap outside, move inward | `O(n)` |
| 23 | Recursive two pointers | Palindrome | Compare outside, move inward | `O(n)` |

### Day 18 — Factorial

```python
def factorial(number):
    if number == 0:
        return 1

    return number * factorial(number - 1)
```

### Day 19 — Fibonacci with Memoization

```python
def fibonacci(number, memo=None):
    if memo is None:
        memo = {}

    if number <= 1:
        return number

    if number in memo:
        return memo[number]

    memo[number] = (
        fibonacci(number - 1, memo)
        + fibonacci(number - 2, memo)
    )

    return memo[number]
```

### Day 20 — Recursive Array Sum

```python
def recursive_sum(nums, index=0):
    if index == len(nums):
        return 0

    return nums[index] + recursive_sum(nums, index + 1)
```

### Day 21 — Recursive Maximum

```python
def recursive_max(nums, index=0):
    if not nums:
        raise ValueError("Array cannot be empty")

    if index == len(nums) - 1:
        return nums[index]

    right_maximum = recursive_max(nums, index + 1)

    return max(nums[index], right_maximum)
```

### Day 22 — Recursive String Reversal

```python
def reverse_string(text):
    characters = list(text)

    def reverse(left, right):
        if left >= right:
            return

        characters[left], characters[right] = (
            characters[right],
            characters[left],
        )

        reverse(left + 1, right - 1)

    reverse(0, len(characters) - 1)
    return "".join(characters)
```

### Day 23 — Recursive Palindrome

```python
def is_palindrome(text):
    def check(left, right):
        if left >= right:
            return True

        if text[left] != text[right]:
            return False

        return check(left + 1, right - 1)

    return check(0, len(text) - 1)
```

---

# Phase 6 — Backtracking

| Day | Concept | Problem | Main decision | Complexity |
|---|---|---|---|---|
| 24 | Binary-choice backtracking | Subsets | Skip or take | `O(n × 2ⁿ)` |
| 25 | Used-choice backtracking | Permutations | Choose any unused item | `O(n × n!)` |
| 26 | Reusable-choice backtracking | Combination Sum | Recurse with same index | Exponential |
| 27 | Duplicate-safe backtracking | Combination Sum II | Use once and skip same-level duplicates | `O(n × 2ⁿ)` |
| 28 | Constraint backtracking | Generate Parentheses | Add only valid symbols | `O(Cₙ × n)` |
| 29 | Grid backtracking | Word Search | Match, mark, explore, restore | `O(R × C × 4ᴸ)` |

## Universal Backtracking Template

```python
def backtrack(state):
    if solution_complete(state):
        save_solution()
        return

    for choice in available_choices(state):
        if choice_is_invalid(choice):
            continue

        make_choice(choice)
        backtrack(updated_state)
        undo_choice(choice)
```

### Day 24 — Subsets

```python
def generate_subsets(nums):
    result = []
    path = []

    def backtrack(index):
        if index == len(nums):
            result.append(path.copy())
            return

        backtrack(index + 1)

        path.append(nums[index])
        backtrack(index + 1)
        path.pop()

    backtrack(0)
    return result
```

### Day 25 — Permutations

```python
def generate_permutations(nums):
    result = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        if len(path) == len(nums):
            result.append(path.copy())
            return

        for index in range(len(nums)):
            if used[index]:
                continue

            used[index] = True
            path.append(nums[index])

            backtrack()

            path.pop()
            used[index] = False

    backtrack()
    return result
```

### Day 26 — Combination Sum

```python
def combination_sum(candidates, target):
    numbers = sorted(candidates)
    result = []
    path = []

    def backtrack(start, remaining):
        if remaining == 0:
            result.append(path.copy())
            return

        for index in range(start, len(numbers)):
            number = numbers[index]

            if number > remaining:
                break

            path.append(number)
            backtrack(index, remaining - number)
            path.pop()

    backtrack(0, target)
    return result
```

Remember:

```text
Same index → reuse allowed
```

### Day 27 — Combination Sum II

```python
def combination_sum_once(candidates, target):
    numbers = sorted(candidates)
    result = []
    path = []

    def backtrack(start, remaining):
        if remaining == 0:
            result.append(path.copy())
            return

        for index in range(start, len(numbers)):
            number = numbers[index]

            if index > start and number == numbers[index - 1]:
                continue

            if number > remaining:
                break

            path.append(number)
            backtrack(index + 1, remaining - number)
            path.pop()

    backtrack(0, target)
    return result
```

Remember:

```text
index + 1 → use once

index > start and equal previous value
→ skip duplicate at the same level
```

### Day 28 — Generate Parentheses

```python
def generate_parentheses(n):
    result = []
    path = []

    def backtrack(open_count, close_count):
        if len(path) == 2 * n:
            result.append("".join(path))
            return

        if open_count < n:
            path.append("(")
            backtrack(open_count + 1, close_count)
            path.pop()

        if close_count < open_count:
            path.append(")")
            backtrack(open_count, close_count + 1)
            path.pop()

    backtrack(0, 0)
    return result
```

Invariant:

```text
0 ≤ close_count ≤ open_count ≤ n
```

### Day 29 — Word Search

```python
def word_exists(board, word):
    if word == "":
        return True

    if not board or not board[0]:
        return False

    rows = len(board)
    columns = len(board[0])

    def search(row, column, index):
        if index == len(word):
            return True

        if (
            row < 0
            or row >= rows
            or column < 0
            or column >= columns
            or board[row][column] != word[index]
        ):
            return False

        original = board[row][column]
        board[row][column] = "#"

        found = (
            search(row - 1, column, index + 1)
            or search(row + 1, column, index + 1)
            or search(row, column - 1, index + 1)
            or search(row, column + 1, index + 1)
        )

        board[row][column] = original
        return found

    return any(
        search(row, column, 0)
        for row in range(rows)
        for column in range(columns)
    )
```

---

# Phase 7 — Binary Trees

| Day | Concept | Problem | Core formula | Complexity |
|---|---|---|---|---|
| 30 | Tree DFS | Maximum Depth | `1 + max(left, right)` | `O(n)` time, `O(h)` space |

### Day 30 — Maximum Depth

```python
def max_depth(root):
    if root is None:
        return 0

    left_depth = max_depth(root.left)
    right_depth = max_depth(root.right)

    return 1 + max(left_depth, right_depth)
```

Remember:

```text
Empty node → 0
Leaf node → 1
Current node → 1 + max(left depth, right depth)
```

---

# Pattern Recognition Map

Use this during interviews and assignments.

| If the question says… | Think about… |
|---|---|
| Duplicate, previously seen | Set |
| Pair with target | Hash map or two pointers |
| Character counts | Frequency map |
| Sorted pair problem | Two pointers |
| Fixed contiguous length | Fixed sliding window |
| Longest/shortest valid window | Variable sliding window |
| Many range sums | Prefix sum |
| Exact subarray sum count | Prefix sum + hash map |
| Recent unmatched item | Stack |
| Next greater/smaller value | Monotonic stack |
| Expiring events | Queue |
| Reverse node connections | Linked-list pointers |
| Sorted search | Binary search |
| Minimum feasible value | Binary search on answer |
| Smaller version of same problem | Recursion |
| Repeated recursive subproblems | Memoization |
| Generate every possibility | Backtracking |
| Choose or skip | Subsets |
| Arrange every element | Permutations |
| Reusable choices total a target | Combination Sum |
| Use each duplicate item once | Combination Sum II |
| Generate only valid sequences | Constrained backtracking |
| Grid path without cell reuse | DFS + backtracking |
| Combine left and right results | Tree DFS |

---

# Real-Time Problem-Solving Framework

Before coding, write:

```text
1. Input
2. Output
3. Constraints
4. Edge cases
5. Does order matter?
6. Are duplicates allowed?
7. Can values be reused?
8. Do we need one answer or every answer?
9. Can the input be modified?
10. What size can the input reach?
```

During implementation:

```text
Choose the pattern.
Define the function promise.
Write the base case.
Identify state variables.
Dry-run a small example.
Implement.
Test edge cases.
Measure time and space complexity.
```

Before delivery:

```text
Validate inputs.
Preserve caller data.
Add tests.
Document assumptions.
Add safe limits.
Write acceptance criteria.
Commit clearly.
Explain client-impacting decisions.
```

---

# Institute-Level Readiness

You should now be able to:

- Explain all 30 patterns in simple language.
- Dry-run each algorithm.
- Write code without copying.
- Calculate basic time and space complexity.
- Answer why each data structure was selected.
- Write unit tests and README documentation.
- Explain edge cases during viva.

---

# Corporate-Level Readiness

For every solution, add:

- Type hints
- Docstrings
- Input validation
- Clear exceptions
- Boundary limits
- Unit tests
- No accidental input mutation
- Logging at service boundaries
- Performance awareness
- Meaningful Git commits
- Code-review checklist
- Production failure handling

A correct algorithm is only one part of production-quality software.

---

# Client-Level Readiness

Always clarify:

- Exact expected result
- Maximum input size
- Duplicate behavior
- Reuse rules
- Ordering requirements
- Response-time expectations
- Whether all results are required
- Error behavior
- Pagination or streaming needs
- Security and permission requirements
- Requirement changes and acceptance criteria

Never assume that an interview-style problem statement is complete enough for a real client system.

---

# Day 1–30 Master Assignment

Build a repository:

```text
dsa-days-1-30/
├── arrays_hashing/
├── sliding_window/
├── stack_queue/
├── linked_list/
├── binary_search/
├── recursion/
├── backtracking/
├── trees/
├── tests/
└── README.md
```

For every problem include:

```text
problem.py
test_problem.py
README section
```

Each README section must contain:

```text
Problem statement
Pattern recognition
Approach
Dry run
Complexity
Edge cases
Production concerns
Client questions
```

## Revision Targets

You are ready to continue when you can implement these without notes:

1. Two Sum  
2. Longest Unique Substring  
3. Subarray Sum Equals K  
4. Valid Parentheses  
5. Daily Temperatures  
6. Reverse Linked List  
7. Binary Search  
8. Koko Eating Bananas  
9. Recursive Palindrome  
10. Subsets  
11. Permutations  
12. Combination Sum  
13. Combination Sum II  
14. Generate Parentheses  
15. Word Search  
16. Maximum Tree Depth  

The next logical topic is **Day 31: Binary Tree Traversals—Preorder, Inorder and Postorder**.
