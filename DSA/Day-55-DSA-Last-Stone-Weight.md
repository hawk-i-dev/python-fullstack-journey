# Day 55 DSA — Last Stone Weight

On Day 54, the heap root was the **worst candidate to keep**. Today, it’s the **next item to act on**. You’ll use a **max-heap** to repeatedly find the two heaviest stones, even as their weights change.

## 1. Think: grab the two heaviest stones

Imagine a pile of stones. In each round, take the two heaviest. If they weigh the same, both disappear. If they weigh `8` and `7`, put one stone of weight `1` back into the pile.

Now ask: **What do I need to find quickly in every round?** Not the whole sorted pile. Just the two largest weights. That’s what the heap gives you.

> **80/20 rule:** When you repeatedly need the largest item and the collection changes after each step, use a max-heap.

**Contract:** Given a list of positive integer weights, return the last remaining weight, or `0` if no stones remain. An empty input returns `0`. Equal weights represent separate stones. Leave the original list unchanged.

## 2. Turn the rule into code

We’ll use Python’s min-heap operations with **negative weights**. Weight `8` is stored as `-8`, which comes out before `-7` because `-8 < -7`. Negate each popped value to recover the original weight.

Each round:

1. Pop the heaviest stone, `heaviest`.
2. Pop the second heaviest, `second`.
3. If `heaviest > second`, push the **negative** of their difference back into the heap. If they’re equal, push nothing.

**Invariant:** At the start of every round, the heap contains exactly the stones still in play, with each weight stored as its negative. Because the first two pops recover the two largest weights, each round follows the problem’s rule.

```python
import heapq


def last_stone_weight(stones):
    heap = [-weight for weight in stones]
    heapq.heapify(heap)

    while len(heap) >= 2:
        heaviest = -heapq.heappop(heap)
        second = -heapq.heappop(heap)

        if heaviest > second:
            remaining = heaviest - second
            heapq.heappush(heap, -remaining)

    return -heap[0] if heap else 0


print(last_stone_weight([2, 7, 4, 1, 8, 1]))  # 1
```

## 3. Dry run: `[2, 7, 4, 1, 8, 1]`

| Round | Two heaviest | Put back | Stones still in play |
|---|---|---:|---|
| 1 | `8`, `7` | `1` | `[4, 2, 1, 1, 1]` |
| 2 | `4`, `2` | `2` | `[2, 1, 1, 1]` |
| 3 | `2`, `1` | `1` | `[1, 1, 1]` |
| 4 | `1`, `1` | Nothing | `[1]` |

The last column is sorted for readability; the heap itself isn’t sorted. Notice that the `2` created in round 2 becomes the heaviest stone in round 3. **That’s why sorting once and walking through the original list won’t work:** a new stone can change the next pair.

## 4. Complexity and mistakes to watch for

Building the heap with `heapify()` takes `O(n)` time. Each round removes two stones and puts back at most one, so the pile shrinks and there are at most `n - 1` rounds for nonempty input. Each round does at most three heap operations, giving **`O(n log n)` worst-case time** and **`O(n)` auxiliary space**.

The two most useful bugs to test for:

- **Pushing the difference as a positive number.** The heap must contain only negated weights. Push `-remaining`, not `remaining`.
- **Pushing a zero after a tie.** Equal stones both disappear. Zero is the return value for an empty pile, not a stone to keep in the heap.

## 5. Practice without copying

Implement `last_stone_weight(stones)` from memory. Before coding, trace `[10, 4, 2, 10, 3]` on paper. Write down both popped weights and what goes back each round. **Expected answer: `1`.**

Then run these checks:

```python
assert last_stone_weight([]) == 0
assert last_stone_weight([6]) == 6
assert last_stone_weight([5, 5]) == 0
assert last_stone_weight([9, 4]) == 5
assert last_stone_weight([10, 4, 2, 10, 3]) == 1

stones = [2, 7, 4, 1, 8, 1]
assert last_stone_weight(stones) == 1
assert stones == [2, 7, 4, 1, 8, 1]
```

**Stretch task:** Write a slower reference solution that sorts the remaining stones each round. Compare it with your heap solution on small inputs. If the two disagree, compare the first round where their piles differ.

## 6. Carry it into real work

- **Institute/interview:** Explain why reinsertion matters and why the loop must terminate. Be able to trace the negative values as well as the real weights.
- **Corporate:** Think of a priority queue that pulls the most urgent work next. If a partially completed job goes back with a new priority, reinsert it with the updated key rather than changing a key in place without restoring heap order.
- **Client:** Clarify what “highest priority” means and what happens on ties. Equal stone weights are interchangeable here; two equally urgent jobs may instead need to run in arrival order.

GitHub filename: `Day-55-DSA-Last-Stone-Weight.md`

**Explain-back checkpoint:** With `[9, 7, 1]`, what goes back after the first round, and which two stones must be picked next?
