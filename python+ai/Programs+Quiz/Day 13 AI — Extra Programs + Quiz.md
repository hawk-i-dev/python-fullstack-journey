# Day 13 AI — Extra Programs + Quiz

**Python code and commands are the same on Mac and Windows** after activating `.venv`.

Run from the Day 12/13 project folder:

```bash
python main_search.py
pytest -q
ruff check .
```

## Program 1 — Inspect TF-IDF vocabulary and vector shape

Create `vocabulary_explorer.py`:

```python
from pathlib import Path

from sklearn.feature_extraction.text import TfidfVectorizer

from vector_search import load_chunks


chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"
chunks = load_chunks(chunks_file)

contents = [chunk["content"] for chunk in chunks]

vectorizer = TfidfVectorizer(
    stop_words="english",
    ngram_range=(1, 2),
)

chunk_vectors = vectorizer.fit_transform(contents)
vocabulary = vectorizer.get_feature_names_out()

print(f"Chunk vector matrix shape: {chunk_vectors.shape}")
print(f"Vocabulary size: {len(vocabulary)}")

print("\nFirst 20 vocabulary terms:")
for term in vocabulary[:20]:
    print(term)
```

Run:

```bash
python vocabulary_explorer.py
```

## Program 2 — Build citation-ready retrieved context

Create `context_builder.py`:

```python
from vector_search import SearchResult


def build_retrieved_context(
    results: list[SearchResult],
) -> str:
    if not results:
        return "No relevant context was retrieved."

    sections: list[str] = []

    for result in results:
        sections.append(
            f"[Source: {result.source} | Chunk: {result.chunk_id}]\n"
            f"{result.content}"
        )

    return "\n\n".join(sections)


if __name__ == "__main__":
    sample_results = [
        SearchResult(
            chunk_id="ai_rag_notes.txt-chunk-001",
            source="ai_rag_notes.txt",
            score=0.81,
            content=(
                "RAG retrieves relevant information before "
                "generating an answer."
            ),
        ),
        SearchResult(
            chunk_id="ai_rag_notes.txt-chunk-002",
            source="ai_rag_notes.txt",
            score=0.63,
            content=(
                "Embeddings are numerical vectors that represent meaning."
            ),
        ),
    ]

    print(build_retrieved_context(sample_results))
```

This prepares context that a future LLM will receive in Day 14.

## Program 3 — Source filter

Add this function to `vector_search.py`:

```python
def filter_chunks_by_source(
    chunks: list[dict[str, str]],
    source: str | None,
) -> list[dict[str, str]]:
    if source is None:
        return chunks

    cleaned_source = source.strip()

    if not cleaned_source:
        raise ValueError("Source cannot be empty.")

    return [
        chunk
        for chunk in chunks
        if chunk["source"] == cleaned_source
    ]
```

Test it manually:

```python
from vector_search import filter_chunks_by_source, load_chunks
from pathlib import Path

chunks = load_chunks(
    Path("data") / "processed" / "ai_rag_chunks.json"
)

filtered = filter_chunks_by_source(
    chunks,
    "ai_rag_notes.txt",
)

print(len(filtered))
```

## Program 4 — Test retrieved context

Create `tests/test_context_builder.py`:

```python
from context_builder import build_retrieved_context
from vector_search import SearchResult


def test_build_retrieved_context_includes_source_and_content() -> None:
    results = [
        SearchResult(
            chunk_id="rag-001",
            source="rag.txt",
            score=0.8,
            content="RAG retrieves relevant context.",
        )
    ]

    context = build_retrieved_context(results)

    assert "rag.txt" in context
    assert "rag-001" in context
    assert "RAG retrieves relevant context." in context


def test_build_retrieved_context_handles_empty_results() -> None:
    assert (
        build_retrieved_context([])
        == "No relevant context was retrieved."
    )
```

Run:

```bash
pytest -q
ruff check .
```

# Day 13 AI MCQ Quiz

Reply like:

```text
1.B 2.A 3.C ...
```

1. What is vectorization in text retrieval?

   A. Deleting text files  
   B. Converting text into numerical representations  
   C. Converting vectors into passwords  
   D. Creating Git branches  

2. What does TF-IDF help represent?

   A. Importance of terms in documents  
   B. File permissions only  
   C. API-key validity  
   D. Python package versions  

3. What should be used on the document corpus first?

   A. `vectorizer.transform(documents)`  
   B. `vectorizer.get_feature_names_out()`  
   C. `vectorizer.fit_transform(documents)`  
   D. `vectorizer.delete(documents)`  

4. What should be used on a new user query after the vectorizer is fitted?

   A. `vectorizer.transform([query])`  
   B. `vectorizer.fit_transform([query])` every time  
   C. `json.load(query)`  
   D. `query.argsort()`  

5. What does cosine similarity measure?

   A. File size  
   B. Similarity of vector direction  
   C. Number of words only  
   D. Git commit quality  

6. What does a higher similarity score usually mean?

   A. The chunk is more relevant to the query  
   B. The file is larger  
   C. The document is newer  
   D. The API key is valid  

7. What does `scores.argsort()[::-1]` help create?

   A. A ranking from highest score to lowest  
   B. A random ordering  
   C. A new vector database  
   D. An LLM response  

8. Why use `minimum_score`?

   A. To filter weak or irrelevant matches  
   B. To increase all scores  
   C. To delete documents  
   D. To stop testing  

9. What should a RAG application do when no chunk meets the relevance threshold?

   A. Invent a confident answer  
   B. Clearly state that no relevant context was found  
   C. Delete all chunks  
   D. Always return the first chunk  

10. Why preserve `source` and `chunk_id` in search results?

   A. For traceability, debugging, and citations  
   B. To improve Python syntax  
   C. To avoid vectorization  
   D. To hide document origins  

11. What is a limitation of TF-IDF search?

   A. It often depends on overlapping words rather than deep semantic meaning  
   B. It cannot process text at all  
   C. It requires an LLM API key  
   D. It always returns random chunks  

12. What does a source filter support?

   A. Searching only approved/selected documents  
   B. Making every query match  
   C. Removing chunk metadata  
   D. Replacing tests  

13. What is the purpose of `build_retrieved_context()`?

   A. Prepare retrieved chunks with source information for an LLM prompt  
   B. Train a foundation model  
   C. Delete irrelevant files  
   D. Create CSV reports  

14. Why test unrelated queries?

   A. To verify the system can safely return no results  
   B. To guarantee all queries have answers  
   C. To avoid validation  
   D. To increase token usage  

15. What will Day 14 add?

   A. A mini RAG answer generator using retrieved context and citations  
   B. A CSS-only webpage  
   C. A relational database migration  
   D. Git installation
