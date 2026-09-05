## Day 14 DSA — Linked List + Reverse Linked List

Title:

```text
Linked List + Reverse Linked List
```

## Feynman Idea

Imagine a treasure hunt.

Each clue tells you where the next clue is.

That is a linked list.

Each node stores:

```text
value
pointer to next node
```

Unlike an array, linked list items are not stored side-by-side in memory.

You move by following `next`.

## 80/20 Core

Remember this:

```text
Linked list problems are pointer rewiring problems.
```

For reverse linked list:

```text
Make each node point backward instead of forward.
```

## What Is a Linked List?

A linked list is a chain of nodes.

Example:

```text
1 → 2 → 3 → 4 → None
```

Each node has:

```python
value
next
```

Node structure:

```python
class ListNode:
    def __init__(self, value=0, next=None):
        self.value = value
        self.next = next
```

## Array vs Linked List

| Operation | Array/List | Linked List |
|---|---|---|
| Access by index | Fast O(1) | Slow O(n) |
| Insert/delete after known node | Costly shift | Fast O(1) |
| Memory layout | Continuous | Scattered |
| Traversal | Direct index | Follow `next` |

## Problem: Reverse Linked List

Given:

```text
1 → 2 → 3 → 4 → None
```

Return:

```text
4 → 3 → 2 → 1 → None
```

You must reverse the links.

Not the values.

## Mental Model

At each node, ask:

```text
Where should current.next point?
```

Originally:

```text
current → next
```

After reversal:

```text
current → previous
```

Use three pointers:

```text
previous
current
next_node
```

## Why Three Pointers?

Before changing `current.next`, save the original next node.

Otherwise, you lose the rest of the list.

```text
next_node = current.next
current.next = previous
previous = current
current = next_node
```

## Code

Create file:

```text
day_14_reverse_linked_list.py
```

Write:

```python
class ListNode:
    def __init__(self, value=0, next=None):
        self.value = value
        self.next = next


def reverse_list(head):
    previous = None
    current = head

    while current:
        next_node = current.next
        current.next = previous
        previous = current
        current = next_node

    return previous


def print_list(head):
    current = head

    while current:
        print(current.value, end=" -> ")
        current = current.next

    print("None")


node4 = ListNode(4)
node3 = ListNode(3, node4)
node2 = ListNode(2, node3)
node1 = ListNode(1, node2)

print_list(node1)

reversed_head = reverse_list(node1)

print_list(reversed_head)
```

Expected output:

```text
1 -> 2 -> 3 -> 4 -> None
4 -> 3 -> 2 -> 1 -> None
```

## Dry Run

Start:

```text
previous = None
current = 1
```

Step 1:

```text
next_node = 2
1.next = None
previous = 1
current = 2
```

Now reversed part:

```text
1 → None
```

Remaining:

```text
2 → 3 → 4 → None
```

Step 2:

```text
next_node = 3
2.next = 1
previous = 2
current = 3
```

Reversed part:

```text
2 → 1 → None
```

Remaining:

```text
3 → 4 → None
```

Continue until:

```text
previous = 4
current = None
```

Return:

```python
previous
```

Because `previous` is now the new head.

## Complexity

```text
Time Complexity: O(n)
```

We visit each node once.

```text
Space Complexity: O(1)
```

We only use pointer variables.

## Common Mistakes

Avoid these:

```text
1. Not saving next_node before changing current.next
2. Returning old head instead of previous
3. Reversing values instead of links
4. Forgetting current = next_node
5. Losing the rest of the list
```

## Practice Task

Solve:

```python
def reverse_list(head):
    pass
```

Test cases:

```text
Input:  1 → 2 → 3 → 4 → None
Output: 4 → 3 → 2 → 1 → None

Input:  1 → None
Output: 1 → None

Input:  None
Output: None
```

Key rule:

```text
Reverse linked list = save next, reverse pointer, move forward.
```
