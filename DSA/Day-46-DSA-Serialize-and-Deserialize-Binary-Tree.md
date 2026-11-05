Day 46 DSA: **Serialize and Deserialize a Binary Tree**

Yesterday, you reconstructed a tree from two traversals. Today, you’ll convert a tree into a single string and rebuild it later.

This is useful whenever a structure must be saved, transmitted, or restored.

```text
Tree → serialize → string → deserialize → new equivalent tree
```

Imagine packing a model made from building blocks. Recording the block labels is not enough—you must also record where pieces are missing so you can rebuild the same shape.

That is today’s key idea:

> **Store node values and missing-child markers.**

We’ll use preorder traversal and `#` to represent a missing node.

Consider:

```text
        1
       / \
      2   3
         / \
        4   5
```

Serialized representation:

```text
1,2,#,#,3,4,#,#,5,#,#
```

The reconstructed tree should have the same values and connections, but it will contain newly created node objects.

**Why values alone are insufficient**

These trees have the same preorder values:

```text
Tree A:       Tree B:

    1             1
   /               \
  2                 2
```

Without markers:

```text
Both → 1,2
```

With missing-child markers:

```text
Tree A → 1,2,#,#,#
Tree B → 1,#,2,#,#
```

Now their shapes are distinguishable.

Unlike Day 45’s value-to-index approach, this format can also handle duplicate values. The markers tell us the structure.

**The 80/20 rules**

For serialization:

1. Missing node → write `#`.
2. Existing node → write its value.
3. Serialize the left subtree.
4. Serialize the right subtree.

For deserialization:

1. Read the next token.
2. `#` → return `None`.
3. Otherwise, create a node from the value.
4. Rebuild its left subtree.
5. Rebuild its right subtree.
6. Return the node.

Both sides follow the same order:

```text
Root → Left → Right
```

**Agree on the format before coding**

Our format will use:

- Integer node values, including negative and repeated values.
- Commas between tokens.
- `#` for a missing child.
- `"#"` for an empty tree.
- No trailing comma.

The deserializer will raise `ValueError` for missing tokens, invalid integer tokens, or extra tokens.

It assumes its input is a string. Serialization assumes valid tree structure: no cycles or shared child nodes.

**Think through serialization**

For this small tree:

```text
      1
     /
    2
```

The traversal writes:

| Visit | Token written |
|---|---|
| Node `1` | `"1"` |
| Node `2` | `"2"` |
| Missing left child of `2` | `"#"` |
| Missing right child of `2` | `"#"` |
| Missing right child of `1` | `"#"` |

Result:

```text
1,2,#,#,#
```

We append tokens to a list and join them once. Repeatedly rebuilding a growing string can introduce unnecessary copying.

**Complete runnable Python**

```python
class TreeNode:
    def __init__(
        self,
        value: int,
        left: "TreeNode | None" = None,
        right: "TreeNode | None" = None,
    ):
        self.value = value
        self.left = left
        self.right = right


def serialize(root: TreeNode | None) -> str:
    tokens = []

    def visit(node: TreeNode | None) -> None:
        if node is None:
            tokens.append("#")
            return

        tokens.append(str(node.value))
        visit(node.left)
        visit(node.right)

    visit(root)
    return ",".join(tokens)


def deserialize(data: str) -> TreeNode | None:
    tokens = data.split(",")
    index = 0

    def build() -> TreeNode | None:
        nonlocal index

        if index >= len(tokens):
            raise ValueError("Incomplete tree data")

        token = tokens[index]
        index += 1

        if token == "#":
            return None

        try:
            value = int(token)
        except ValueError as exc:
            raise ValueError(
                f"Invalid integer token: {token!r}"
            ) from exc

        node = TreeNode(value)
        node.left = build()
        node.right = build()

        return node

    root = build()

    if index != len(tokens):
        raise ValueError("Extra tokens after the tree")

    return root


root = TreeNode(
    1,
    TreeNode(2),
    TreeNode(3, TreeNode(4), TreeNode(5)),
)

data = serialize(root)
restored = deserialize(data)

print(data)
# 1,2,#,#,3,4,#,#,5,#,#

print(serialize(restored))
# 1,2,#,#,3,4,#,#,5,#,#

print(restored is root)
# False: different objects, equivalent tree
```

`int(token)` accepts integer spellings supported by Python, such as surrounding whitespace or a leading `+`. The serializer itself produces the consistent form shown above.

For example:

```python
serialize(deserialize("+1,#,#"))
# "1,#,#"
```

The tree is preserved even though the spelling is normalized.

**How deserialization knows when to stop**

Each `build()` call reconstructs **exactly one subtree**.

If it reads `#`, that subtree is empty and the call finishes immediately.

If it reads a value, it must consume two more complete subtrees: one left and one right.

For:

```text
1,2,#,#,#
```

The calls behave like this:

```text
build reads 1
│
├── build left: reads 2
│   ├── build left: reads # → None
│   └── build right: reads # → None
│   returns node 2
│
└── build right: reads # → None

returns node 1
```

The shared cursor ensures that every call starts where the previous call finished.

This is the same cursor idea as Day 45, but missing-child markers now define the subtree boundaries.

**Why leftover tokens must be rejected**

Consider:

```text
1,#,#,9
```

The first three tokens already describe a complete one-node tree:

```text
1,#,#
```

The trailing `9` has no valid position.

If we ignore leftover tokens, we silently accept malformed data. That is why we check:

```python
if index != len(tokens):
    raise ValueError(...)
```

Similarly:

```text
1,#
```

is incomplete. The root has a left-child marker, but its right subtree is missing from the encoding.

**Why the algorithm works**

The first token identifies whether the subtree is empty.

For a nonempty subtree, the next token gives the root value. The following tokens recursively describe exactly one left subtree and then exactly one right subtree.

Because the boundaries are determined by the same rules at every level, the original structure can be rebuilt without guessing.

Duplicate values do not cause ambiguity:

```text
      5
     / \
    5   5
```

Encoding:

```text
5,5,#,#,5,#,#
```

Each occurrence gets its own position in the structure.

**Complexity**

Let:

- `n` be the node count.
- `h` be the height.
- `L` be the serialized string length.

Under the usual assumption of bounded-size integer values:

- Serialization takes **O(n)** time.
- Deserialization takes **O(n)** time.
- Each uses **O(h)** recursive stack space.
- The token list uses **O(n)** space.
- The serialized output and reconstructed tree each require **O(n)** space.

More generally, string storage depends on `L`: values with many digits require more space and conversion work.

An `n`-node binary tree has `n + 1` missing-child pointers. Therefore, this format contains:

```text
n value tokens + (n + 1) markers = 2n + 1 tokens
```

That includes the empty tree:

```text
n = 0 → one token: #
```

Token count is useful as a sanity check, but does not by itself prove the sequence is valid.

**Common mistakes**

| Mistake | Why it fails |
|---|---|
| Store only preorder values | Different shapes can produce the same values |
| Omit missing-child markers inconsistently | Decoder loses structural boundaries |
| Decode right before left | Reconstructs a different structure |
| Treat `0` as missing | Zero is a valid node value |
| Ignore remaining tokens | Accepts malformed input |
| Compare restored root with original using `is` | Reconstruction creates new objects |
| Assume ordinary tree encoding preserves shared references | It represents a tree, not arbitrary object graphs |

**Tests to run**

```python
assert serialize(None) == "#"
assert deserialize("#") is None

single = TreeNode(0)
assert serialize(single) == "0,#,#"

left_only = TreeNode(1, left=TreeNode(2))
right_only = TreeNode(1, right=TreeNode(2))

assert serialize(left_only) == "1,2,#,#,#"
assert serialize(right_only) == "1,#,2,#,#"

duplicates = TreeNode(-1, TreeNode(-1), TreeNode(0))
encoded = serialize(duplicates)
assert serialize(deserialize(encoded)) == encoded

# Main example round-trip.
encoded = serialize(root)
restored = deserialize(encoded)
assert serialize(restored) == encoded
assert restored is not root

# Invalid input must be rejected.
for invalid in ["", "1,#", "1,#,#,9", "x,#,#", "#,#"]:
    try:
        deserialize(invalid)
    except ValueError:
        pass
    else:
        raise AssertionError(f"Expected failure for {invalid!r}")
```

For additional confidence, compare the original and reconstructed trees recursively by values and left/right structure. This checks equivalence independently of the serializer.

**Your assignment**

Build this tree:

```text
          10
         /  \
       -2    10
         \   /
          4 0
```

Expected serialization:

```text
10,-2,#,4,#,#,10,0,#,#,#
```

Complete these tasks:

1. Implement `serialize()` and `deserialize()` independently.
2. Trace the tokens written during serialization.
3. Trace the decoder’s cursor movement.
4. Confirm that reserializing the restored tree produces the same string.
5. Test empty trees, duplicate values, zero, negative values, and malformed input.

This tree contains five nodes, so expect:

```text
2 × 5 + 1 = 11 tokens
```

As an extension, design a level-order format using BFS and missing-child markers. Specify whether trailing markers may be omitted before implementing the decoder; both sides must agree on the format.

At **institute level**, explain why markers preserve shape and why equal values do not prevent reconstruction. Be able to decode a small string by hand.

At **corporate level**, define format versions and validate data before treating it as a restored structure. For external inputs, choose practical size and depth limits. An iterative implementation avoids recursive stack limits for deep trees.

At **client level**, clarify whether “restore” means equivalent values and structure or preserving business identifiers and relationships. New Python objects do not automatically restore database records, permissions, or external references.

For GitHub:

```text
Day-46-DSA-Serialize-and-Deserialize-Binary-Tree.md
```

Before coding, draw the tree represented by:

```text
8,#,4,2,#,#,#
```

**Which child of `8` exists, and where does node `2` belong?**
