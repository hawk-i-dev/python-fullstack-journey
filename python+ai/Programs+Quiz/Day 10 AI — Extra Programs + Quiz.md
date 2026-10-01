# Day 10 AI — Extra Programs + Quiz

**Code and commands are the same on Mac and Windows** after activating `.venv`.

Run:

```bash
python lessons/file_name.py
ruff check .
```

## Program 1 — Vectorized calculations

Create `lessons/day_10_program_01.py`:

```python
import numpy as np


daily_token_usage = np.array([120, 250, 180, 320, 210])

print("Token usage:", daily_token_usage)
print("With 10% increase:", daily_token_usage * 1.10)
print("Average:", daily_token_usage.mean())
print("Maximum:", daily_token_usage.max())
print("Minimum:", daily_token_usage.min())
```

## Program 2 — Matrix shape and axes

Create `lessons/day_10_program_02.py`:

```python
import numpy as np


document_embeddings = np.array([
    [0.2, 0.8, 0.1],
    [0.9, 0.1, 0.3],
    [0.5, 0.4, 0.7],
])

print("Matrix:")
print(document_embeddings)

print("\nShape:", document_embeddings.shape)
print("Column totals, axis=0:", document_embeddings.sum(axis=0))
print("Row totals, axis=1:", document_embeddings.sum(axis=1))
```

## Program 3 — Element-wise multiplication and dot product

Create `lessons/day_10_program_03.py`:

```python
import numpy as np


query_embedding = np.array([0.8, 0.1, 0.7])
document_embedding = np.array([0.7, 0.2, 0.8])

print("Element-wise multiplication:")
print(query_embedding * document_embedding)

print("\nDot-product similarity score:")
print(query_embedding @ document_embedding)
```

## Program 4 — Rank documents by similarity

Create `lessons/day_10_program_04.py`:

```python
import numpy as np


document_names = np.array([
    "rag_basics.txt",
    "python_functions.txt",
    "ai_agents.txt",
    "vector_databases.txt",
])

document_embeddings = np.array([
    [0.9, 0.1, 0.2],
    [0.1, 0.9, 0.1],
    [0.7, 0.2, 0.8],
    [0.8, 0.1, 0.7],
])

query_embedding = np.array([0.8, 0.1, 0.7])

scores = document_embeddings @ query_embedding
ranked_indexes = np.argsort(scores)[::-1]

print("Ranked search results:")

for index in ranked_indexes:
    print(f"{document_names[index]}: {scores[index]:.3f}")
```

`np.argsort(scores)` returns indexes in sorted order. `[::-1]` reverses them so the highest score appears first.

# Day 10 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. What is NumPy mainly used for?

   A. Designing web pages  
   B. Fast numerical computing with arrays  
   C. Managing Git branches  
   D. Writing JSON only  

2. Which imports NumPy using the common alias?

   A. `import numpy as np`  
   B. `import np as numpy`  
   C. `from Python import numpy`  
   D. `install numpy`  

3. What is a NumPy array?

   A. Only a text file  
   B. A Git repository  
   C. A numerical data structure optimized for array operations  
   D. A Python exception  

4. What does this produce?

```python
np.array([1, 2, 3]) * 2
```

   A. `[1, 2, 3, 1, 2, 3]`  
   B. `[2, 4, 6]`  
   C. `6`  
   D. Error  

5. What does this shape mean?

```python
(3, 4)
```

   A. Three rows and four columns  
   B. Four rows and three columns  
   C. A vector with seven values  
   D. Three separate arrays  

6. Which represents one embedding vector with three values?

   A. `np.array([[0.1, 0.2, 0.3]])` only  
   B. `np.array([0.1, 0.2, 0.3])`  
   C. `{"0.1", "0.2", "0.3"}`  
   D. `"0.1, 0.2, 0.3"`  

7. What does `matrix.sum(axis=0)` calculate?

   A. One sum for each row  
   B. One sum for each column  
   C. The largest value only  
   D. The matrix shape  

8. What does `matrix.sum(axis=1)` calculate?

   A. One sum for each row  
   B. One sum for each column  
   C. The matrix data type  
   D. A new empty matrix  

9. What does `*` usually mean between two equal-shaped NumPy arrays?

   A. Dot product  
   B. Element-wise multiplication  
   C. String joining  
   D. Array sorting  

10. What does `@` mean between two compatible vectors?

   A. Element-wise multiplication  
   B. Dot product / matrix multiplication  
   C. List append  
   D. JSON conversion  

11. Why are embeddings useful in AI?

   A. They represent meaning as numerical vectors  
   B. They replace all databases  
   C. They guarantee factual answers  
   D. They are only for CSS styling  

12. What does `np.argmax(scores)` return?

   A. The highest score value  
   B. The index of the highest score  
   C. Every score in reverse  
   D. The array shape  

13. What does `np.argsort(scores)` return?

   A. The original array unchanged  
   B. The average score  
   C. Indexes that would sort the scores  
   D. Only the largest score  

14. Why must query and document embeddings have matching dimensions?

   A. They must be stored in the same file  
   B. Similarity operations such as dot product require compatible shapes  
   C. NumPy works only with three values  
   D. They must have the same document name  

15. For a very large production RAG document collection, what is usually more suitable than plain NumPy search?

   A. A vector database or specialized search system  
   B. A Python list only  
   C. A README file  
   D. A random-number generator
