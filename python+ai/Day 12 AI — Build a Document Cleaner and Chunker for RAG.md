# Day 12 AI — Build a Document Cleaner and Chunker for RAG

Today’s EOD project prepares text documents for a future RAG system.

```text
Raw document
   ↓
Clean text
   ↓
Split into meaningful chunks
   ↓
Save chunks with metadata
   ↓
Later: embeddings → vector database → retrieval → LLM answer
```

A chunk is the unit your RAG system searches. Poor chunking produces poor retrieval, even if the LLM is strong.

## Start your environment

**Windows PowerShell**

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
cd projects
New-Item -ItemType Directory -Force day_12_document_chunker
cd day_12_document_chunker
New-Item -ItemType Directory -Force data, data\raw, data\processed, tests
```

**Mac Terminal**

```zsh
cd ~/Documents/python-ai-mastery
source .venv/bin/activate
cd projects
mkdir -p day_12_document_chunker/data/raw day_12_document_chunker/data/processed day_12_document_chunker/tests
cd day_12_document_chunker
```

All Python code below is the same on Mac and Windows.

---

## 1. Create a sample document

Create `data/raw/ai_rag_notes.txt`:

```text
Retrieval-Augmented Generation, or RAG, improves an LLM application by
retrieving relevant information before generating an answer.

A RAG system usually starts with documents such as PDFs, text files, web
pages, support articles, or database records. The documents are cleaned and
split into smaller chunks.

Each chunk is converted into an embedding, which is a numerical vector that
represents meaning. These embeddings are stored in a vector database.

When a user asks a question, the application converts the question into an
embedding. It searches for document chunks with similar embeddings and sends
the best chunks to the LLM as context.

RAG does not guarantee that every answer is correct. A reliable application
shows source citations, handles missing context, protects private documents,
and evaluates retrieval quality.
```

---

## 2. Create the document-processing module

Create `document_processor.py`:

```python
import json
from dataclasses import asdict, dataclass
from pathlib import Path


@dataclass(frozen=True)
class DocumentChunk:
    chunk_id: str
    source: str
    index: int
    content: str
    word_count: int


def clean_text(text: str) -> str:
    if not text.strip():
        raise ValueError("Document text cannot be empty.")

    paragraphs: list[str] = []

    for paragraph in text.split("\n\n"):
        cleaned_paragraph = " ".join(paragraph.split())

        if cleaned_paragraph:
            paragraphs.append(cleaned_paragraph)

    return "\n\n".join(paragraphs)


def chunk_text(
    text: str,
    *,
    source: str,
    chunk_size_words: int = 50,
    overlap_words: int = 10,
) -> list[DocumentChunk]:
    if chunk_size_words <= 0:
        raise ValueError("chunk_size_words must be greater than zero.")

    if overlap_words < 0:
        raise ValueError("overlap_words cannot be negative.")

    if overlap_words >= chunk_size_words:
        raise ValueError(
            "overlap_words must be smaller than chunk_size_words."
        )

    cleaned_text = clean_text(text)
    words = cleaned_text.split()

    chunks: list[DocumentChunk] = []
    step_size = chunk_size_words - overlap_words

    for index, start in enumerate(range(0, len(words), step_size)):
        chunk_words = words[start : start + chunk_size_words]

        if not chunk_words:
            break

        chunk_content = " ".join(chunk_words)

        chunks.append(
            DocumentChunk(
                chunk_id=f"{source}-chunk-{index + 1:03d}",
                source=source,
                index=index,
                content=chunk_content,
                word_count=len(chunk_words),
            )
        )

        if start + chunk_size_words >= len(words):
            break

    return chunks


def save_chunks(
    chunks: list[DocumentChunk],
    output_file: Path,
) -> None:
    output_file.parent.mkdir(parents=True, exist_ok=True)

    chunk_data = [asdict(chunk) for chunk in chunks]

    with output_file.open("w", encoding="utf-8") as file:
        json.dump(chunk_data, file, indent=2)
```

---

## 3. Create the program entry point

Create `main.py`:

```python
from pathlib import Path

from document_processor import chunk_text, save_chunks


def main() -> None:
    input_file = Path("data") / "raw" / "ai_rag_notes.txt"
    output_file = Path("data") / "processed" / "ai_rag_chunks.json"

    if not input_file.exists():
        raise FileNotFoundError(
            f"Input document was not found: {input_file}"
        )

    raw_text = input_file.read_text(encoding="utf-8")

    chunks = chunk_text(
        raw_text,
        source=input_file.name,
        chunk_size_words=40,
        overlap_words=8,
    )

    save_chunks(chunks, output_file)

    print(f"Created {len(chunks)} chunks.")
    print(f"Saved chunks to: {output_file}")

    for chunk in chunks:
        preview = chunk.content[:80]
        print(
            f"\n{chunk.chunk_id} "
            f"({chunk.word_count} words): {preview}..."
        )


if __name__ == "__main__":
    main()
```

Run:

```bash
python main.py
ruff check .
```

You should find this file:

```text
data/processed/ai_rag_chunks.json
```

---

# Concepts to understand

## Why clean text?

Raw documents may contain:

```text
extra spaces
empty lines
line breaks in the middle of sentences
copied PDF formatting
```

Cleaning creates consistent text before chunking.

## Why chunk documents?

An LLM cannot receive an entire company knowledge base in every request.

```text
10,000-page document library
        ↓
small relevant chunks
        ↓
only best chunks are sent to the LLM
```

## Why overlap chunks?

Without overlap, a useful sentence can be split between two chunks.

```text
Chunk 1: RAG retrieves relevant document ...
Chunk 2: ... chunks before generating an answer.
```

Overlap preserves some context across boundaries.

---

# 1–3 Hour Delivery Plan

## 1-hour MVP

```text
[ ] Read one text file
[ ] Clean text
[ ] Split text into chunks
[ ] Print chunk previews
```

## 2-hour version

```text
[ ] Add chunk IDs and metadata
[ ] Add word overlap
[ ] Save chunks as JSON
[ ] Validate empty input and invalid settings
```

## 3-hour corporate version

Create `tests/test_document_processor.py`:

```python
import pytest

from document_processor import clean_text, chunk_text


def test_clean_text_normalizes_whitespace() -> None:
    text = "  RAG   retrieves \n\n useful   context.  "

    assert clean_text(text) == "RAG retrieves\n\nuseful context."


def test_chunk_text_creates_overlapping_chunks() -> None:
    text = "one two three four five six seven eight"

    chunks = chunk_text(
        text,
        source="demo.txt",
        chunk_size_words=4,
        overlap_words=1,
    )

    assert len(chunks) == 3
    assert chunks[0].content == "one two three four"
    assert chunks[1].content == "four five six seven"


def test_chunk_text_rejects_invalid_overlap() -> None:
    with pytest.raises(ValueError, match="smaller"):
        chunk_text(
            "one two three",
            source="demo.txt",
            chunk_size_words=3,
            overlap_words=3,
        )
```

Run:

```bash
pytest -q
ruff check .
```

---

# Assignment Extension

Add this function to `document_processor.py`:

```python
def filter_short_chunks(
    chunks: list[DocumentChunk],
    minimum_words: int,
) -> list[DocumentChunk]:
```

Requirements:

```text
- Reject a minimum_words value below 1.
- Return only chunks with enough words.
- Add two pytest tests.
```

---

# Institute, Corporate, and Client Approach

## Institute level

Be able to explain:

```text
What is RAG?
What is chunking?
Why is overlap useful?
Why save metadata?
Why do we use JSON?
```

## Corporate level

Before processing documents, confirm:

```text
[ ] Who owns the documents?
[ ] Does the user have permission to access them?
[ ] Does the data contain personal, financial, or confidential content?
[ ] What chunk size works for this document type?
[ ] How will retrieval quality be measured?
[ ] Are source citations required?
```

## Client explanation

> “This tool prepares documents for an AI knowledge-search system. It breaks large documents into traceable pieces so the future assistant can retrieve relevant information and show where the answer came from.”

Day 13 AI will use these chunks to build a **local vector search engine**, the core retrieval step of RAG.
