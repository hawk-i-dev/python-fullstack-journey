# Day 14 AI — Extra Programs + Quiz

**Python code and commands are the same on Mac and Windows** after activating `.venv`.

Run from the Day 12–14 project folder:

```bash
python main_rag.py
pytest -q
ruff check .
```

## Program 1 — Retrieval confidence checker

Add this to `rag_answer.py`:

```python
from vector_search import SearchResult


def classify_retrieval_confidence(
    results: list[SearchResult],
) -> str:
    if not results:
        return "no_context"

    best_score = results[0].score

    if best_score >= 0.30:
        return "high"
    elif best_score >= 0.05:
        return "low"

    return "no_context"
```

Use it in `main_rag.py`:

```python
from rag_answer import classify_retrieval_confidence
```

Then add after `results` is created:

```python
confidence = classify_retrieval_confidence(results)

print(f"\nRetrieval confidence: {confidence}")
```

Important: these score thresholds are for learning only. Real systems evaluate thresholds using real user questions and expected answers.

## Program 2 — Markdown citations

Add this function to `rag_answer.py`:

```python
def format_citations_markdown(
    citations: list[Citation],
) -> str:
    if not citations:
        return "No sources were retrieved."

    lines = ["## Sources"]

    for citation in citations:
        lines.append(
            f"- `{citation.source}` — "
            f"`{citation.chunk_id}` "
            f"(similarity: {citation.score})"
        )

    return "\n".join(lines)
```

Test it in a new file named `citation_demo.py`:

```python
from rag_answer import Citation, format_citations_markdown


citations = [
    Citation(
        source="ai_rag_notes.txt",
        chunk_id="ai_rag_notes.txt-chunk-001",
        score=0.82,
    )
]

print(format_citations_markdown(citations))
```

## Program 3 — Save a RAG audit record

Create `rag_audit.py`:

```python
import json
from dataclasses import asdict
from pathlib import Path

from rag_answer import RAGAnswer


def save_rag_audit_record(
    question: str,
    rag_answer: RAGAnswer,
    output_file: Path,
) -> None:
    output_file.parent.mkdir(parents=True, exist_ok=True)

    record = {
        "question": question,
        "answer": rag_answer.answer,
        "has_context": rag_answer.has_context,
        "citations": [
            asdict(citation)
            for citation in rag_answer.citations
        ],
    }

    with output_file.open("w", encoding="utf-8") as file:
        json.dump(record, file, indent=2)
```

In a real company, do not store sensitive user questions or private document content unless there is clear permission, retention policy, and access control.

## Program 4 — Test confidence classification

Add this to `tests/test_rag_answer.py`:

```python
from rag_answer import classify_retrieval_confidence
from vector_search import SearchResult


def test_classify_retrieval_confidence_returns_high() -> None:
    results = [
        SearchResult(
            chunk_id="rag-001",
            source="rag.txt",
            score=0.8,
            content="RAG retrieves context.",
        )
    ]

    assert classify_retrieval_confidence(results) == "high"


def test_classify_retrieval_confidence_handles_no_results() -> None:
    assert classify_retrieval_confidence([]) == "no_context"
```

# Day 14 AI MCQ Quiz

Reply like:

```text
1.B 2.A 3.C ...
```

1. What is the main purpose of the Day 14 RAG console?

   A. Train a new foundation model  
   B. Search retrieved chunks and return a source-backed answer  
   C. Delete document chunks  
   D. Replace Python with JSON  

2. What does “grounded answer” mean?

   A. An answer based only on retrieved/approved context  
   B. An answer that uses random internet facts  
   C. An answer without sources  
   D. An answer that always agrees with the user  

3. Why include citations in a RAG answer?

   A. To make responses longer only  
   B. To help users verify the source of the answer  
   C. To remove document metadata  
   D. To avoid testing  

4. What should happen when no relevant context is retrieved?

   A. Invent an answer confidently  
   B. Return the first available chunk  
   C. Clearly state that relevant information was not found  
   D. Delete the query  

5. What does `SearchResult` represent?

   A. A retrieved chunk with source, score, and content  
   B. An API key  
   C. A Git branch  
   D. A Pandas DataFrame only  

6. What does `RAGAnswer.has_context` communicate?

   A. Whether usable retrieved context exists  
   B. Whether Python is installed  
   C. Whether a document is private  
   D. Whether Git has commits  

7. Why build the RAG prompt separately from the answer formatter?

   A. To keep responsibilities modular and testable  
   B. To make imports circular  
   C. To avoid validation  
   D. To remove citations  

8. What is an extractive answer?

   A. An answer derived directly from retrieved evidence  
   B. An answer trained from scratch  
   C. An answer with no context  
   D. A type of API key  

9. What does a low retrieval-confidence result mean?

   A. The result may not fully answer the question  
   B. The answer is guaranteed correct  
   C. The source does not exist  
   D. The program must crash  

10. Does a high similarity score guarantee that an answer is factually correct?

   A. Yes, always  
   B. No; it only indicates retrieval relevance, not truth  
   C. Yes, if JSON is used  
   D. Yes, if the document is long  

11. Why should a production RAG system keep retrieval and generation separate?

   A. To make evaluation, debugging, and safety easier  
   B. To remove all tests  
   C. To avoid using source documents  
   D. To hide errors  

12. What should a real system consider before saving RAG audit records?

   A. Privacy, permission, retention policy, and access control  
   B. Only file color  
   C. Only Python version  
   D. Only model name  

13. Why test the no-context situation?

   A. To ensure the system abstains safely instead of hallucinating  
   B. To force every query to return an answer  
   C. To remove validation  
   D. To increase token usage  

14. What is the role of a confidence threshold?

   A. Separate stronger matches from weak/irrelevant matches  
   B. Guarantee model truthfulness  
   C. Replace user permissions  
   D. Create embeddings automatically  

15. What will Day 15 add to the RAG project?

   A. A real LLM provider adapter using environment variables  
   B. A CSS-only dashboard  
   C. A spreadsheet report  
   D. A Git reinstallation
