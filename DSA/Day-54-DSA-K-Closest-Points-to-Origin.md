# Day 54 DSA: K Closest Points to the Origin

Yesterday, you learned that a heap root is the **smallest key**, not necessarily the smallest original value. Today, we’ll use that idea to keep the `k` closest points while making the **farthest retained point** easy to remove.

## 1. Think: a delivery van with two seats

Imagine a delivery van can stop at only two locations, and you want the two closest to your warehouse. You keep a shortlist. Every time you find a new location, compare it with the **farthest location on that shortlist**. If the new one is closer, swap it in.

That’s the 80/20 idea:

> **To keep the closest `k`, make the farthest retained point easiest to remove.**

For this exercise, the warehouse is the origin `(0, 0)`, and closeness means straight-line distance.

## 2. Define “closest” and handle ties

For a point `(x, y)`, its distance from the origin is `sqrt(x*x + y*y)`. We don’t need the square root: it preserves the ordering of nonnegative numbers.

```text
Point (1, 2) → squared distance 1² + 2² = 5
Point (3, 3) → squared distance 3² + 3² = 18

5 < 18, so (1, 2) is closer.
```

**Our contract:** Given a list of integer coordinate pairs and an integer `k` with `0 <= k <= len(points)`, return exactly `k` points as tuples. Rank them by squared distance, then by smaller `x`, then by smaller `y`. Duplicate points count separately. Return `[]` for `k = 0`, raise `ValueError` for an out-of-range `k`, and leave the input unchanged.

A straightforward baseline is to sort by `(x*x + y*y, x, y)` and take the first `k`. But when `k` is small compared with the number of points, we can keep a much smaller shortlist.

## 3. Derive the heap key

Python’s min-heap puts the **smallest key** at the root. We want the **worst retained point** there. So we reverse all three parts of the ranking:

```python
key = (-distance_squared, -x, -y)
```

Here’s why both distance and tie-breaker signs matter:

| Point | Squared distance | Heap key | Which is worse? |
|---|---:|---|---|
| `(3, 3)` | 18 | `(-18, -3, -3)` | Worse than either point at distance squared `5` |
| `(1, 2)` | 5 | `(-5, -1, -2)` | Worse than `(-2, 1)` because its `x` is larger |
| `(-2, 1)` | 5 | `(-5, 2, -1)` | Better of the two tied points |

The heap compares tuples left to right. **A larger key means a better candidate**, so replace the root only when `candidate > heap[0]`.

**Invariant:** After processing each point, the heap contains the best `min(k, points_seen)` points under our ranking rule.

## 4. Runnable Python

```python
import heapq


def k_closest(points, k):
    if not 0 <= k <= len(points):
        raise ValueError("k must be between 0 and len(points)")
    if k == 0:
        return []

    heap = []

    for x, y in points:
        distance_squared = x * x + y * y
        candidate = (-distance_squared, -x, -y)

        if len(heap) < k:
            heapq.heappush(heap, candidate)
        elif candidate > heap[0]:
            heapq.heapreplace(heap, candidate)

    # Larger encoded keys are better. Decode the retained points.
    return [(-neg_x, -neg_y)
            for _, neg_x, neg_y in sorted(heap, reverse=True)]


points = [(3, 3), (1, 2), (-2, 1), (0, 1)]
print(k_closest(points, 2))  # [(0, 1), (-2, 1)]
```

### Dry run: `k = 2`

| Incoming point | Distance² | Decision | Retained points, best first |
|---|---:|---|---|
| `(3, 3)` | 18 | Push: room available | `[(3, 3)]` |
| `(1, 2)` | 5 | Push: room available | `[(1, 2), (3, 3)]` |
| `(-2, 1)` | 5 | Replace `(3, 3)` | `[(-2, 1), (1, 2)]` |
| `(0, 1)` | 1 | Replace `(1, 2)`, the worse of the tied points | `[(0, 1), (-2, 1)]` |

The last column shows ranking order for explanation. **The heap itself is not sorted.**

### Why it works

While the heap has room, every point belongs on the shortlist. Once it’s full, a candidate no better than the root cannot improve the shortlist: all `k` retained points already rank at least as well. A better candidate takes the root’s place. That preserves the invariant at every step.

For `n` points and `k >= 1`, worst-case time is **`O(n log(k + 1))`**, including sorting the final `k` results. Auxiliary space is **`O(k)`**. For `k = 0`, the function returns immediately.

## 5. Tests that catch the common mistakes

```python
# Same distance: smaller x wins.
assert k_closest([(1, 2), (-2, 1)], 1) == [(-2, 1)]

# Same distance and x: smaller y wins.
assert k_closest([(1, 2), (1, -2)], 1) == [(1, -2)]

# Duplicate points count separately.
assert k_closest([(2, 0), (0, 0), (0, 0)], 2) == [(0, 0), (0, 0)]

# k=0 and k=n.
assert k_closest([], 0) == []
assert k_closest([(2, 0), (0, 0)], 2) == [(0, 0), (2, 0)]
```

A common bug is storing `(distance_squared, x, y)` directly in a bounded min-heap. That puts the **closest** point at the root—the one you most want to keep. Another is negating the distance but not the tie-breakers, which fails when two points are equally close.

## 6. Your assignment

Implement `k_closest()` without copying. First write the key for each point and trace every time the root changes:

```python
points = [(2, 0), (-2, 0), (0, 2), (1, 1), (0, 0)]
k = 3

# Expected: [(0, 0), (1, 1), (-2, 0)]
```

Then add `k_closest_to(points, k, center)` so the reference point can be anywhere. Change the distance calculation, but keep the same tie-breaking rule on the original coordinates.

```python
k_closest_to([(0, 0), (3, 1), (2, 2)], 2, center=(2, 1))
# Expected: [(2, 2), (3, 1)]
```

- **Institute/interview:** Explain why squared distance preserves the ranking and why the farthest retained point sits at the root.
- **Corporate:** Compare your heap implementation against the sorting baseline. Include cases with ties, duplicates, invalid ranks, and unchanged input.
- **Client:** Ask what “nearest” means. The nearest shop by straight-line distance might not be the fastest to reach by road. Agree on the distance or travel-time metric before choosing the ranking key.

GitHub filename: `Day-54-DSA-K-Closest-Points-to-Origin.md`

**Explain-back checkpoint:** For `k = 1`, points `(1, 2)` and `(1, -2)` are equally close and have the same `x`. Which point wins, and how do the two heap keys prove it?
