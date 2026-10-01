# Day 10 AI — NumPy and Vector Mathematics for AI

NumPy is the foundation for numerical Python. Machine learning, embeddings, neural networks, images, and similarity search all work with numerical arrays.

## Start your environment

**Windows PowerShell**

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
```

**Mac Terminal**

```zsh
cd ~/Documents/python-ai-mastery
source .venv/bin/activate
```

The remaining commands and Python code are the same on both systems.

Check NumPy:

```bash
python -c "import numpy as np; print(np.__version__)"
```

---

## 1. Python list vs NumPy array

```python
import numpy as np

scores_list = [10, 20, 30]
scores_array = np.array([10, 20, 30])
```

A Python list is flexible. A NumPy array is built for fast numerical work.

```python
print(scores_list * 2)
# [10, 20, 30, 10, 20, 30]

print(scores_array * 2)
# [20, 40, 60]
```

For AI, use arrays when processing numerical vectors, matrices, images, model outputs, or embeddings.

---

## 2. Dimensions and `shape`

```python
vector = np.array([0.2, 0.8, 0.1])
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

print(vector.shape)  # (3,)
print(matrix.shape)  # (2, 3)
```

```text
(3,)    → one-dimensional vector with 3 values
(2, 3) → 2 rows, each with 3 columns
```

In AI:

```text
One embedding      → vector: (embedding_size,)
Many embeddings    → matrix: (document_count, embedding_size)
```

---

## 3. Vectorized operations

Avoid manual loops when NumPy can process an entire array:

```python
expenses = np.array([120, 250, 80, 150])

print(expenses + 10)
print(expenses * 1.18)
print(expenses.mean())
print(expenses.max())
```

This is called **vectorization**. It is clearer and often much faster than processing values one by one with Python loops.

---

## 4. Axes

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

print(matrix.sum(axis=0))  # [5, 7, 9]
print(matrix.sum(axis=1))  # [6, 15]
```

```text
axis=0 → reduce rows; result is calculated per column
axis=1 → reduce columns; result is calculated per row
```

For document embeddings:

```text
shape: (100 documents, 1536 embedding values)

axis=0 → calculate across all documents
axis=1 → calculate inside each document vector
```

---

## 5. Element-wise multiplication vs matrix multiplication

```python
first = np.array([1, 2, 3])
second = np.array([4, 5, 6])

print(first * second)  # [4, 10, 18]
print(first @ second)  # 32
```

- `*` multiplies each matching position.
- `@` calculates matrix multiplication / dot product.

The dot product is central to similarity search.

---

## 6. Embedding similarity idea

An embedding is a list of numbers representing meaning.

```text
"What is RAG?"          → [0.1, 0.8, 0.2, ...]
"Explain vector search" → [0.2, 0.7, 0.3, ...]
```

A query and a relevant document often have vectors pointing in similar directions.

For now, we use dot product:

```python
score = query_embedding @ document_embedding
```

Later, we will use cosine similarity and real embeddings from an embedding model.

---

# Day 10 Program — Mini Document Similarity Search

Create `lessons/day_10_numpy_similarity.py`:

```python
import numpy as np


document_names = np.array([
    "rag_basics.txt",
    "python_lists.txt",
    "ai_agents.txt",
])

# Demo vectors only. Real embedding models generate these values.
document_embeddings = np.array([
    [0.9, 0.1, 0.2],
    [0.1, 0.9, 0.1],
    [0.7, 0.2, 0.8],
])

query_embedding = np.array([0.8, 0.1, 0.7])

similarity_scores = document_embeddings @ query_embedding

best_index = int(np.argmax(similarity_scores))
best_document = document_names[best_index]
best_score = similarity_scores[best_index]

print("Document matrix shape:", document_embeddings.shape)
print("\nSimilarity scores:")

for name, score in zip(document_names, similarity_scores, strict=True):
    print(f"{name}: {score:.3f}")

print(f"\nBest matching document: {best_document}")
print(f"Score: {best_score:.3f}")
```

Run:

```bash
python lessons/day_10_numpy_similarity.py
ruff check .
```

Expected best match: `ai_agents.txt`.

---

# Assignment — Embedding Similarity Explorer

Create `ai_core/vector_search.py`.

Requirements:

```text
Input:
- document names
- document embedding matrix
- query embedding vector
- number of results to return

Output:
- top matching document names and scores
```

## Acceptance criteria

- Reject an empty document list.
- Reject embedding dimensions that do not match.
- Reject `top_k` less than 1.
- Return results in descending score order.
- Use NumPy vectorized operations.
- Add at least three pytest tests.
- Pass:

```bash
pytest -q
ruff check .
```

Example expected output:

```python
[
    {"document": "ai_agents.txt", "score": 1.14},
    {"document": "rag_basics.txt", "score": 0.93},
]
```

---

# Real-world approach

## Institute level

Explain:

```text
What is an array?
What is a vector?
What is a matrix?
What does shape mean?
What is a dot product?
How does similarity search connect to RAG?
```

Submit:

```text
- NumPy program
- Assignment code
- Test output
- Short diagram of query vector → scores → top documents
```

## Corporate level

For a real feature, confirm:

```text
- Are all embeddings the same dimension?
- What happens with malformed data?
- How many documents must be searched?
- Is NumPy fast enough, or do we need a vector database?
- How do we evaluate retrieval accuracy and latency?
```

NumPy is excellent for learning and small prototypes. Large production RAG systems usually use a vector database or specialized search engine.

## Client level

Ask:

```text
Which documents can be searched?
Who is allowed to access each document?
Should answers show source citations?
What happens when no relevant document exists?
How accurate must search results be?
```

Never say “the AI found the truth.” Say: “The system found the most similar available documents,” then show sources.

## Key takeaway

```text
NumPy array  → fast numerical data structure
Vector       → one numerical representation
Matrix       → many vectors together
shape        → dimensions of an array
@            → dot product / matrix multiplication
argmax       → position of highest score
```

Day 11 AI will cover **Pandas and real-world data analysis for AI datasets.**
