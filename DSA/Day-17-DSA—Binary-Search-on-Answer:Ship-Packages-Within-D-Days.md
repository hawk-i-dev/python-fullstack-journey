Day 17 DSA — Binary Search on Answer: Ship Packages Within D Days

Today’s concept is the same pattern as Day 16, but with a different real interview problem.

## 1. Problem

You are given package weights and a number of days.

You need to find the minimum ship capacity so that all packages can be shipped within `D` days.

Example:

```python
weights = [1,2,3,4,5,6,7,8,9,10]
days = 5
```

Answer:

```python
15
```

Why? With capacity `15`, we can ship like this:

```text
Day 1: 1 + 2 + 3 + 4 + 5 = 15
Day 2: 6 + 7 = 13
Day 3: 8
Day 4: 9
Day 5: 10
```

Minimum capacity required is `15`.

## 2. Core idea

We are not searching package indexes.

We are searching the possible ship capacity.

Minimum possible capacity:

```python
max(weights)
```

Because the ship must carry at least the heaviest package.

Maximum possible capacity:

```python
sum(weights)
```

Because one ship can carry everything in one day.

So answer space is:

```python
max(weights) to sum(weights)
```

## 3. Why binary search works

If capacity is too small, shipping fails.

If capacity is large enough, shipping works.

Pattern:

```text
Fail Fail Fail Pass Pass Pass
```

We need the first `Pass`.

## 4. How to check one capacity

For a guessed capacity:

- Start with `days_used = 1`
- Keep adding packages to current day
- If adding next package crosses capacity:
  - Start a new day
  - Add that package there
- At the end, check if `days_used <= days`

Example with capacity `15`:

```text
weights = [1,2,3,4,5,6,7,8,9,10]

Day 1: 1+2+3+4+5 = 15
Day 2: 6+7 = 13
Day 3: 8
Day 4: 9
Day 5: 10
```

Works.

## 5. Python solution

```python
def ship_within_days(weights, days):
    def can_ship(capacity):
        days_used = 1
        current_load = 0

        for weight in weights:
            if current_load + weight > capacity:
                days_used += 1
                current_load = weight
            else:
                current_load += weight

        return days_used <= days

    left = max(weights)
    right = sum(weights)
    answer = right

    while left <= right:
        mid = (left + right) // 2

        if can_ship(mid):
            answer = mid
            right = mid - 1
        else:
            left = mid + 1

    return answer
```

## 6. Dry run

```python
weights = [1,2,3,4,5,6,7,8,9,10]
days = 5
```

Search range:

```text
left = 10
right = 55
```

Try capacity `32`:

```text
Can ship within 5 days? Yes
Try smaller
```

Try capacity `20`:

```text
Can ship within 5 days? Yes
Try smaller
```

Try capacity `14`:

```text
Can ship within 5 days? No
Try bigger
```

Try capacity `17`:

```text
Can ship within 5 days? Yes
Try smaller
```

Try capacity `15`:

```text
Can ship within 5 days? Yes
Try smaller
```

Final answer:

```python
15
```

## 7. Complexity

Let:

```text
n = number of packages
m = sum(weights) - max(weights)
```

For every capacity guess, we scan all packages once.

Time:

```text
O(n log(sum(weights)))
```

Space:

```text
O(1)
```

## 8. Common mistakes

1. Starting `left = 1`

Wrong because capacity must be at least the heaviest package.

Correct:

```python
left = max(weights)
```

2. Sorting the weights

Wrong. Package order must be preserved.

3. Starting `days_used = 0`

Better to start with:

```python
days_used = 1
```

Because shipping starts on day 1.

4. Forgetting to reset current load

When capacity crosses, reset:

```python
current_load = weight
```

5. Returning the first working `mid` immediately

Wrong. You need the minimum working capacity, so continue searching left.

## 9. Practice problem

Implement this:

```python
def ship_within_days(weights, days):
    pass
```

Test with:

```python
print(ship_within_days([1,2,3,4,5,6,7,8,9,10], 5))
# 15

print(ship_within_days([3,2,2,4,1,4], 3))
# 6

print(ship_within_days([1,2,3,1,1], 4))
# 3
```

## 10. Interview sentence

Say this clearly:

“We binary search the minimum possible ship capacity. For each capacity, we greedily simulate shipping in order and check whether the packages can be shipped within the allowed days.”
