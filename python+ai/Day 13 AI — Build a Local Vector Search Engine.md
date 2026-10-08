# Day 13 AI — Build a Local Vector Search Engine

Today’s EOD project adds the **retrieval** step of RAG.

```text
Day 12: document → clean → chunks → JSON
Day 13: chunks → vectors → similarity search → top matching chunks
```

We will use **TF-IDF + cosine similarity** locally. It is not a modern semantic embedding model, but it teaches the complete vector-search workflow without API keys or downloads.

## Start your environment

Continue inside your Day 12 project.

**Windows PowerShell**

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
cd projects\day_12_document_chunker
```

**Mac Terminal**

```zsh
cd ~/Documents/python-ai-mastery
source .venv/bin/activate
cd projects/day_12_document_chunker
```

Check the library:

```bash
python -c "import sklearn; print(sklearn.__version__)"
```

If it is missing:

```bash
pip install scikit-learn
```

---

# Core concepts

## Vectorization

A computer cannot directly compare the meaning of two text chunks. First, text becomes numerical vectors.

```text
"RAG retrieves document chunks"
              ↓
[0.0, 0.3, 0.8, 0.1, ...]
```

## TF-IDF

TF-IDF gives higher importance to words that are useful in one document but not common everywhere.

```text
TF  → term frequency inside a document
IDF → importance based on rarity across documents
```

## Cosine similarity

Cosine similarity compares vector direction.

```text
Query vector      → "How does RAG retrieve chunks?"
Chunk vector      → "RAG retrieves relevant document chunks."

Higher similarity score → more relevant chunk
```

For TF-IDF vectors, scores normally range from `0` to `1`.

---

# Day 13 Project — Local RAG Retriever

Create `vector_search.py`:

```python
import json
from dataclasses import dataclass
from pathlib import Path

from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity


@dataclass(frozen=True)
class SearchResult:
    chunk_id: str
    source: str
    score: float
    content: str


def load_chunks(file_path: Path) -> list[dict[str, str]]:
    with file_path.open(encoding="utf-8") as file:
        chunks = json.load(file)

    if not isinstance(chunks, list) or not chunks:
        raise ValueError("Chunk file must contain a non-empty list.")

    validated_chunks: list[dict[str, str]] = []

    for chunk in chunks:
        if not isinstance(chunk, dict):
            raise ValueError("Every chunk must be a JSON object.")

        chunk_id = chunk.get("chunk_id")
        source = chunk.get("source")
        content = chunk.get("content")

        if not all(
            isinstance(value, str) and value.strip()
            for value in [chunk_id, source, content]
        ):
            raise ValueError(
                "Every chunk requires non-empty chunk_id, source, and content."
            )

        validated_chunks.append(
            {
                "chunk_id": chunk_id,
                "source": source,
                "content": content,
            }
        )

    return validated_chunks


def search_chunks(
    chunks: list[dict[str, str]],
    query: str,
    *,
    top_k: int = 3,
    minimum_score: float = 0.05,
) -> list[SearchResult]:
    cleaned_query = query.strip()

    if not cleaned_query:
        raise ValueError("Search query cannot be empty.")

    if top_k < 1:
        raise ValueError("top_k must be at least 1.")

    if not 0 <= minimum_score <= 1:
        raise ValueError("minimum_score must be between 0 and 1.")

    contents = [chunk["content"] for chunk in chunks]

    vectorizer = TfidfVectorizer(
        stop_words="english",
        ngram_range=(1, 2),
    )

    chunk_vectors = vectorizer.fit_transform(contents)
    query_vector = vectorizer.transform([cleaned_query])

    scores = cosine_similarity(
        query_vector,
        chunk_vectors,
    ).flatten()

    ranked_indexes = scores.argsort()[::-1]

    results: list[SearchResult] = []

    for index in ranked_indexes:
        score = float(scores[index])

        if score < minimum_score:
            continue

        chunk = chunks[index]

        results.append(
            SearchResult(
                chunk_id=chunk["chunk_id"],
                source=chunk["source"],
                score=round(score, 4),
                content=chunk["content"],
            )
        )

        if len(results) == top_k:
            break

    return results
```

Create `main_search.py`:

```python
from pathlib import Path

from vector_search import load_chunks, search_chunks


def main() -> None:
    chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"

    chunks = load_chunks(chunks_file)

    query = input("Ask a question about the document: ")

    results = search_chunks(
        chunks,
        query,
        top_k=3,
        minimum_score=0.05,
    )

    if not results:
        print("\nNo relevant document chunks were found.")
        return

    print("\nRetrieved chunks:")

    for position, result in enumerate(results, start=1):
        print(f"\n{position}. {result.chunk_id}")
        print(f"Source: {result.source}")
        print(f"Similarity score: {result.score}")
        print(f"Content: {result.content}")


if __name__ == "__main__":
    main()
```

Run:

```bash
python main_search.py
ruff check .
```

Try these queries:

```text
How does RAG retrieve information?
Why are embeddings stored in a vector database?
How should private documents be handled?
```

---

# Why this is not yet semantic search

TF-IDF depends mainly on matching words.

```text
Query: “How do I find relevant knowledge?”
Chunk: “RAG retrieves relevant document chunks.”
```

TF-IDF may miss the connection because the exact words differ.

A real embedding model better understands that:

```text
find relevant knowledge ≈ retrieve relevant document chunks
```

We will add proper semantic embeddings later.

---

# 1–3 Hour Delivery Plan

## 1-hour MVP

```text
[ ] Load Day 12 chunks JSON
[ ] Convert chunk text to TF-IDF vectors
[ ] Accept a user query
[ ] Print the best matching chunk
```

## 2-hour version

```text
[ ] Return top-k results
[ ] Add cosine similarity scores
[ ] Validate query, top_k, and chunk data
[ ] Return “no relevant chunk” when all scores are too low
```

## 3-hour corporate version

Create `tests/test_vector_search.py`:

```python
from vector_search import search_chunks


CHUNKS = [
    {
        "chunk_id": "rag-001",
        "source": "rag.txt",
        "content": (
            "RAG retrieves relevant document chunks before "
            "the model generates an answer."
        ),
    },
    {
        "chunk_id": "python-001",
        "source": "python.txt",
        "content": (
            "Python functions organize reusable program logic."
        ),
    },
]


def test_search_chunks_returns_most_relevant_result() -> None:
    results = search_chunks(
        CHUNKS,
        "retrieves relevant document chunks",
        top_k=1,
    )

    assert len(results) == 1
    assert results[0].chunk_id == "rag-001"
    assert results[0].score > 0


def test_search_chunks_returns_no_results_for_unrelated_query() -> None:
    results = search_chunks(
        CHUNKS,
        "quantum particle physics",
        top_k=3,
    )

    assert results == []
```

Run:

```bash
pytest -q
ruff check .
```

---

# Assignment Extension

Add a `source` filter:

```python
def search_chunks(
    chunks: list[dict[str, str]],
    query: str,
    *,
    top_k: int = 3,
    minimum_score: float = 0.05,
    source: str | None = None,
) -> list[SearchResult]:
```

Requirements:

```text
- If source is None, search every chunk.
- If source is supplied, search only chunks from that source.
- Return an empty list if no chunks belong to the requested source.
- Add two tests.
```

---

# Institute, Corporate, and Client Approach

## Institute level

Explain:

```text
What is TF-IDF?
What is cosine similarity?
Why do we rank results?
Why can similarity search return no result?
What is the difference between keyword and semantic search?
```

## Corporate level

Before release, confirm:

```text
[ ] Are chunk IDs and source metadata preserved?
[ ] Do users only search documents they are permitted to access?
[ ] What minimum score avoids irrelevant results?
[ ] How will retrieval accuracy be evaluated?
[ ] Are “no result” answers safe and clear?
[ ] Are latency and search volume monitored?
```

## Client explanation

> “This feature searches your approved knowledge documents and returns the most relevant source passages. It does not generate an answer yet; it first proves that the right information can be retrieved.”

Day 14 AI will use the retrieved chunks to build a **mini RAG answer generator with source citations.**
