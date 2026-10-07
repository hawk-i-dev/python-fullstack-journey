# Day 12 AI — Extra Programs + Quiz

**Python code and commands are the same on Mac and Windows** after activating `.venv`.

Run from your Day 12 project folder:

```bash
python main.py
pytest -q
ruff check .
```

## Program 1 — Filter short chunks

Add this function to `document_processor.py`:

```python
def filter_short_chunks(
    chunks: list[DocumentChunk],
    minimum_words: int,
) -> list[DocumentChunk]:
    if minimum_words < 1:
        raise ValueError("minimum_words must be at least 1.")

    return [
        chunk
        for chunk in chunks
        if chunk.word_count >= minimum_words
    ]
```

Add to `main.py` after creating `chunks`:

```python
from document_processor import filter_short_chunks
```

Then add:

```python
chunks = filter_short_chunks(chunks, minimum_words=10)
```

## Program 2 — Chunk review report

Create `chunk_report.py`:

```python
import json
from pathlib import Path


chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"

with chunks_file.open(encoding="utf-8") as file:
    chunks = json.load(file)

word_counts = [chunk["word_count"] for chunk in chunks]

print(f"Total chunks: {len(chunks)}")
print(f"Smallest chunk: {min(word_counts)} words")
print(f"Largest chunk: {max(word_counts)} words")
print(f"Average chunk size: {sum(word_counts) / len(word_counts):.2f} words")

print("\nChunk IDs:")
for chunk in chunks:
    print(chunk["chunk_id"])
```

Run:

```bash
python chunk_report.py
```

## Program 3 — Search chunks using normal Python text matching

Create `keyword_search.py`:

```python
import json
from pathlib import Path


def search_chunks(
    chunks: list[dict[str, object]],
    query: str,
) -> list[dict[str, object]]:
    query_words = set(query.lower().split())

    matches: list[dict[str, object]] = []

    for chunk in chunks:
        content = str(chunk["content"]).lower()
        matching_words = sum(
            word in content
            for word in query_words
        )

        if matching_words > 0:
            matches.append(
                {
                    "chunk_id": chunk["chunk_id"],
                    "score": matching_words,
                    "content": chunk["content"],
                }
            )

    return sorted(
        matches,
        key=lambda item: int(item["score"]),
        reverse=True,
    )


chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"

with chunks_file.open(encoding="utf-8") as file:
    chunks = json.load(file)

results = search_chunks(chunks, "RAG document chunks")

for result in results:
    print(f"\n{result['chunk_id']} | Score: {result['score']}")
    print(result["content"])
```

This is a simple keyword search. Day 13 will replace it with embedding/vector similarity search.

## Program 4 — Test the short-chunk filter

Add this to `tests/test_document_processor.py`:

```python
from document_processor import DocumentChunk, filter_short_chunks


def test_filter_short_chunks_returns_only_large_enough_chunks() -> None:
    chunks = [
        DocumentChunk(
            chunk_id="one",
            source="demo.txt",
            index=0,
            content="one two",
            word_count=2,
        ),
        DocumentChunk(
            chunk_id="two",
            source="demo.txt",
            index=1,
            content="one two three four",
            word_count=4,
        ),
    ]

    filtered_chunks = filter_short_chunks(chunks, minimum_words=3)

    assert len(filtered_chunks) == 1
    assert filtered_chunks[0].chunk_id == "two"
```

# Day 12 AI MCQ Quiz

Reply like:

```text
1.B 2.A 3.C ...
```

1. What does RAG stand for?

   A. Random AI Generation  
   B. Retrieval-Augmented Generation  
   C. Rapid Agent Graph  
   D. Response Analysis Gateway  

2. What is the purpose of document chunking in RAG?

   A. Split large documents into searchable, manageable pieces  
   B. Delete unnecessary documents  
   C. Encrypt every document  
   D. Replace embeddings  

3. Why clean text before chunking?

   A. To make files larger  
   B. To remove every punctuation mark  
   C. To normalize formatting and reduce noisy whitespace  
   D. To convert text directly into an LLM answer  

4. Why use overlapping chunks?

   A. To preserve context across chunk boundaries  
   B. To guarantee factual AI answers  
   C. To remove chunk metadata  
   D. To avoid testing  

5. In today’s project, what does `chunk_size_words` control?

   A. Number of files  
   B. Maximum number of words in a chunk  
   C. Number of vector databases  
   D. Number of users  

6. What is useful chunk metadata?

   A. Source file name and chunk ID  
   B. Developer password  
   C. Random color values  
   D. Git branch password  

7. Why save chunks as JSON?

   A. JSON can represent structured chunk content and metadata  
   B. JSON automatically creates embeddings  
   C. JSON replaces Python code  
   D. JSON guarantees privacy  

8. What should happen if document text is empty?

   A. Create an empty embedding  
   B. Raise a clear validation error  
   C. Upload it to Git  
   D. Create infinite chunks  

9. Why must `overlap_words` be smaller than `chunk_size_words`?

   A. Otherwise chunking can fail to progress correctly  
   B. To make JSON valid  
   C. To improve Git history  
   D. To avoid using lists  

10. What does a chunk ID help with?

   A. Source tracking, debugging, and citations  
   B. API-key generation  
   C. Increasing model temperature  
   D. Creating a virtual environment  

11. What is the main difference between today’s keyword search and future vector search?

   A. Keyword search only compares text terms; vector search compares semantic meaning  
   B. Vector search uses no data  
   C. Keyword search requires an LLM  
   D. They are exactly the same  

12. Before processing client documents, what must be checked?

   A. Only document color  
   B. Permissions, privacy, ownership, and sensitivity  
   C. Only file size  
   D. Only Python version  

13. Why should a RAG system show source citations?

   A. To help users verify where retrieved information came from  
   B. To hide uncertainty  
   C. To remove all documents  
   D. To avoid logging  

14. What should happen when no relevant chunk is found?

   A. The system should invent a confident answer  
   B. The system should clearly say it lacks relevant context  
   C. The system should delete the documents  
   D. The system should stop permanently  

15. What will Day 13 add to this project?

   A. CSS styling  
   B. A relational database only  
   C. Vector/embedding similarity search  
   D. Git installation
