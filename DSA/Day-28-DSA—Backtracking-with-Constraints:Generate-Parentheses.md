# Day 28 DSA — Backtracking with Constraints: Generate Parentheses

## 1. Problem

Given `n` pairs of parentheses, generate every valid arrangement.

For:

```python
n = 3
```

Expected output:

```python
[
    "((()))",
    "(()())",
    "(())()",
    "()(())",
    "()()()",
]
```

A valid arrangement must satisfy:

- Every opening parenthesis has a closing parenthesis.
- A closing parenthesis cannot appear before its matching opening parenthesis.
- Exactly `n` opening and `n` closing parentheses are used.

---

# 2. Feynman Explanation

Imagine opening and closing toy boxes.

```text
( means open a box
) means close a box
```

You may open a new box if you still have boxes available.

You may close a box only if there is already an open box waiting to be closed.

Valid:

```text
( ( ) )
```

Invalid:

```text
) (
```

You cannot close a box before opening it.

---

# 3. The Important 20%

Only two rules are needed:

```text
Add "(" when open_count < n
Add ")" when close_count < open_count
```

A complete answer contains:

```text
2 × n characters
```

Core backtracking decisions:

```python
if open_count < n:
    choose "("

if close_count < open_count:
    choose ")"
```

These conditions prevent invalid paths before they are created.

---

# 4. How to Recognize This Pattern

Look for requirements such as:

```text
Generate all valid sequences
Choices have ordering rules
Some choices become invalid based on earlier choices
Return every valid arrangement
```

This suggests:

```text
Backtracking + state counters + pruning
```

Unlike basic permutations, not every choice is always allowed.

---

# 5. How to Think Before Coding

Ask these questions.

### What are the choices?

```text
Add "("
Add ")"
```

### When can we add `"("`?

When fewer than `n` opening parentheses have been used:

```python
open_count < n
```

### When can we add `")"`?

Only when an unmatched opening parenthesis exists:

```python
close_count < open_count
```

### When is the answer complete?

```python
len(path) == 2 * n
```

### What must be undone?

After exploring a choice:

```python
path.pop()
```

---

# 6. State Variables

```python
path         # characters in the current sequence
open_count   # number of "(" characters used
close_count  # number of ")" characters used
result       # completed valid sequences
```

Example state:

```text
path = ["(", "(", ")"]
open_count = 2
close_count = 1
```

One box remains open, so `")"` is allowed.

---

# 7. Validity Invariant

During every recursive call:

```text
0 ≤ close_count ≤ open_count ≤ n
```

This is called an invariant: a rule that remains true throughout the algorithm.

It guarantees:

- Closings never exceed openings.
- Openings never exceed `n`.
- Invalid prefixes are never explored.

---

# 8. Decision Tree for `n = 2`

Start:

```text
""
```

A closing parenthesis is not allowed first.

```text
""
 └── "("
      ├── "(("
      │     └── "(()"
      │           └── "(())" ✅
      │
      └── "()"
            └── "()("
                  └── "()()" ✅
```

Answers:

```python
["(())", "()()"]
```

The algorithm never creates:

```text
")("
"())("
```

Those branches are prevented by the rules.

---

# 9. Dry Run for `n = 2`

Initial state:

```text
path = []
open_count = 0
close_count = 0
```

Only `"("` is allowed:

```text
path = ["("]
open_count = 1
close_count = 0
```

## First branch: add another `"("`

```text
path = ["(", "("]
open_count = 2
close_count = 0
```

No more openings are allowed.

Add `")"` twice:

```text
"(("
"(()"
"(())"  → save
```

Backtrack to:

```text
"("
```

## Second branch: close the first pair

```text
"()"
```

Add the remaining pair:

```text
"()("
"()()"  → save
```

Final result:

```python
["(())", "()()"]
```

---

# 10. Python Implementation

```python
def generate_parentheses(n):
    result = []
    path = []

    def backtrack(open_count, close_count):
        # A complete valid sequence
        if len(path) == 2 * n:
            result.append("".join(path))
            return

        # Choice 1: add an opening parenthesis
        if open_count < n:
            path.append("(")
            backtrack(open_count + 1, close_count)
            path.pop()

        # Choice 2: close an existing open parenthesis
        if close_count < open_count:
            path.append(")")
            backtrack(open_count, close_count + 1)
            path.pop()

    backtrack(0, 0)
    return result


print(generate_parentheses(3))
```

Output:

```python
[
    "((()))",
    "(()())",
    "(())()",
    "()(())",
    "()()()",
]
```

---

# 11. Build the Code Step by Step

Create storage:

```python
result = []
path = []
```

Track both counts:

```python
def backtrack(open_count, close_count):
```

Save a complete sequence:

```python
if len(path) == 2 * n:
    result.append("".join(path))
    return
```

Add an opening parenthesis when available:

```python
if open_count < n:
    path.append("(")
    backtrack(open_count + 1, close_count)
    path.pop()
```

Add a closing parenthesis only when valid:

```python
if close_count < open_count:
    path.append(")")
    backtrack(open_count, close_count + 1)
    path.pop()
```

Start with zero characters:

```python
backtrack(0, 0)
```

---

# 12. Why Not Generate Everything and Filter Later?

A weaker approach generates every string of length `2n`:

```text
Each position has two choices.
Total raw strings = 2²ⁿ
```

It then checks which strings are valid.

Most generated strings are useless.

The backtracking solution only follows valid prefixes:

```python
if open_count < n:
    ...

if close_count < open_count:
    ...
```

This is a key production lesson:

> Prevent invalid work early instead of generating it and repairing it later.

---

# 13. Why the Algorithm Works

The algorithm creates sequences using only valid choices.

It never:

- Uses more than `n` openings.
- Uses more closings than openings.
- Saves a sequence before it has `2n` characters.

When the sequence reaches length `2n`, it must contain:

```text
n opening parentheses
n closing parentheses
```

Therefore, every saved sequence is valid.

Because both legal choices are explored at every state, every valid sequence is eventually generated.

---

# 14. Edge Cases

## Zero pairs

```python
generate_parentheses(0)
```

Output:

```python
[""]
```

There is one valid arrangement using zero pairs: the empty string.

## One pair

```python
generate_parentheses(1)
```

Output:

```python
["()"]
```

## Two pairs

```python
generate_parentheses(2)
```

Output:

```python
["(())", "()()"]
```

## Negative input

```python
generate_parentheses(-1)
```

Production code should reject it:

```python
raise ValueError("n must be non-negative")
```

---

# 15. Number of Valid Results

The number of valid parenthesis strings is a Catalan number.

```text
n = 0 → 1
n = 1 → 1
n = 2 → 2
n = 3 → 5
n = 4 → 14
n = 5 → 42
n = 10 → 16,796
n = 15 → 9,694,845
```

The output grows quickly even though invalid branches are pruned.

---

# 16. Complexity

Let `Cₙ` be the `n`th Catalan number.

There are `Cₙ` valid outputs, and constructing each output requires `O(n)` work.

```text
Time: O(Cₙ × n)
Auxiliary recursion space: O(n)
Output space: O(Cₙ × n)
```

The recursion depth is at most:

```text
2n
```

which simplifies to:

```text
O(n)
```

---

# 17. Common Mistakes

## Allowing any closing parenthesis

Incorrect:

```python
if close_count < n:
```

This may create invalid prefixes such as:

```text
")"
"())"
```

Correct:

```python
if close_count < open_count:
```

## Using `<=`

Incorrect:

```python
if open_count <= n:
```

This allows `n + 1` opening parentheses.

Correct:

```python
if open_count < n:
```

## Saving `path` without joining

For a string result, save:

```python
result.append("".join(path))
```

## Forgetting to undo

Every append needs a matching pop:

```python
path.append("(")
backtrack(...)
path.pop()
```

## Updating the wrong counter

Adding `"("` increments `open_count`.

Adding `")"` increments `close_count`.

---

# 18. Real-World Assignment — Parser Test-Pattern Generator

A team is developing a mathematical expression editor.

They need every valid nested-parenthesis pattern for testing.

Examples for two pairs:

```python
[
    "(())",
    "()()",
]
```

Implement:

```python
def generate_test_patterns(pair_count):
    pass
```

The patterns will be used to test:

- Expression parsing
- Syntax highlighting
- Auto-formatting
- Cursor navigation
- Parenthesis matching

---

# 19. Institute-Level Assignment

Implement `generate_test_patterns()` using backtracking.

Requirements:

- Do not generate invalid strings and filter afterward.
- Maintain opening and closing counters.
- Use a list for `path`.
- Return all valid patterns.
- Explain the decision tree for `n = 3`.
- Explain the invariant.
- Include tests and a README.

Submission:

```text
day-28-generate-parentheses/
├── assignment.py
├── test_assignment.py
└── README.md
```

Viva questions:

1. Why can’t `")"` be the first character?
2. Why must `close_count < open_count`?
3. Why is the total length `2n`?
4. What is the invariant?
5. Why is generate-and-filter inefficient?

---

# 20. Corporate-Level Requirements

Add:

- Type hints
- Docstring
- Input validation
- Unit tests
- Maximum `n` limit
- Maximum-result protection
- Optional lazy generation
- Clear exceptions
- Metrics for request size and duration

Suggested signature:

```python
def generate_test_patterns(
    pair_count: int,
    max_pairs: int = 10,
) -> list[str]:
    """Return all valid parenthesis patterns."""
    pass
```

Validate:

```text
pair_count must be an integer
pair_count must be non-negative
pair_count must not exceed max_pairs
```

Do not accept Boolean values accidentally:

```python
isinstance(True, int)  # True
```

A strict check can use:

```python
type(pair_count) is int
```

---

# 21. Client-Level Questions

Before implementation, ask:

1. Do you need all valid patterns or only the count?
2. What is the maximum number of pairs?
3. Should results be returned at once or streamed?
4. Are multiple bracket types required?
5. Are empty results allowed for zero pairs?
6. Is result ordering important?
7. Should duplicate patterns be impossible by design?
8. Is there a response-time limit?
9. Is pagination required?
10. Will users provide the value directly?
11. Should an oversized request be rejected or processed asynchronously?
12. Are partially valid patterns also needed for negative testing?

---

# 22. Requirement-Change Examples

## Client needs only the number of patterns

Generating every string wastes memory.

Use a Catalan-number calculation or dynamic programming.

## Client needs multiple bracket types

The state must track bracket types, not only two counters.

A stack may also become necessary.

## Client needs invalid examples

Create a separate negative-test generator. Do not mix invalid patterns into the valid generator.

## Client needs millions of patterns

Return an iterator or process them as a stream instead of storing everything.

---

# 23. Acceptance Criteria

```text
pair_count=0 returns [""].
pair_count=1 returns ["()"].
pair_count=2 returns exactly "(())" and "()()".
pair_count=3 returns exactly 5 unique patterns.

Every result has length 2 × pair_count.
Every prefix has closing_count ≤ opening_count.
Every result contains pair_count opens and closes.
Negative input is rejected.
Input above the configured limit is rejected.
```

---

# 24. Testing Checklist

Test:

```python
generate_test_patterns(0)
generate_test_patterns(1)
generate_test_patterns(2)
generate_test_patterns(3)
generate_test_patterns(-1)
generate_test_patterns(11)
generate_test_patterns("3")
generate_test_patterns(True)
```

Validation helper:

```python
def is_valid(pattern):
    balance = 0

    for character in pattern:
        if character == "(":
            balance += 1
        else:
            balance -= 1

        if balance < 0:
            return False

    return balance == 0
```

Test every returned pattern:

```python
for pattern in result:
    assert is_valid(pattern)
```

Test uniqueness:

```python
assert len(result) == len(set(result))
```

---

# 25. Git and Review Expectations

Suggested commits:

```text
feat: implement valid parenthesis generator
test: cover Catalan counts and invalid inputs
refactor: add request limits and validation
docs: explain invariants and client considerations
```

During review, verify:

- Are invalid prefixes prevented?
- Does every append have a matching pop?
- Are both counters updated correctly?
- Is the input limit appropriate?
- Are large-output risks documented?
- Are tests verifying properties, not only fixed examples?

---

# 26. Your Day 28 Assignment

Complete three levels.

### Level 1 — Algorithm

Implement `generate_test_patterns()`.

### Level 2 — Production quality

Add validation, tests, type hints, limits and property-based checks.

### Level 3 — Requirement analysis

Document:

- Client questions
- Acceptance criteria
- Large-output risks
- How the design changes if only the count is required

## Day 28 takeaway

```text
Add "(" when open_count < n.

Add ")" when close_count < open_count.

Save when length == 2 × n.

Prevent invalid paths before exploring them.
```
