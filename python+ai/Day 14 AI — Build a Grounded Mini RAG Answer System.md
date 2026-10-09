# Day 14 AI — Build a Grounded Mini RAG Answer System

Today you complete the first end-to-end local RAG pipeline:

```text
Document
→ chunks
→ vector search
→ top relevant context
→ grounded answer
→ source citation
```

Today’s system will use an **extractive local answer**—it only returns retrieved evidence and never invents information. This is the safety-critical layer before we connect a real LLM in a later project.

## Start your environment

Continue in the Day 12/13 project.

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

---

# Core RAG rule

A grounded answer system must follow this rule:

```text
Use retrieved context only.
If no relevant context exists, say so.
Show the source used.
Do not invent unsupported facts.
```

This is far safer than sending a user question directly to an LLM.

---

# Day 14 Project — Grounded RAG Console

Create `rag_answer.py`:

```python
from dataclasses import dataclass

from vector_search import SearchResult


@dataclass(frozen=True)
class Citation:
    source: str
    chunk_id: str
    score: float


@dataclass(frozen=True)
class RAGAnswer:
    answer: str
    citations: list[Citation]
    has_context: bool


def build_rag_prompt(
    question: str,
    results: list[SearchResult],
) -> str:
    cleaned_question = question.strip()

    if not cleaned_question:
        raise ValueError("Question cannot be empty.")

    if not results:
        context = "No relevant context was retrieved."
    else:
        context_sections = []

        for result in results:
            context_sections.append(
                f"[Source: {result.source} | Chunk: {result.chunk_id}]\n"
                f"{result.content}"
            )

        context = "\n\n".join(context_sections)

    return (
        "You are a careful RAG assistant.\n"
        "Answer only using the retrieved context.\n"
        "If the context does not answer the question, say that clearly.\n"
        "Include source citations in the final answer.\n\n"
        f"Question:\n{cleaned_question}\n\n"
        f"Retrieved context:\n{context}"
    )


def create_extractive_answer(
    results: list[SearchResult],
) -> RAGAnswer:
    if not results:
        return RAGAnswer(
            answer=(
                "I could not find relevant information in the "
                "available documents."
            ),
            citations=[],
            has_context=False,
        )

    best_result = results[0]

    citation = Citation(
        source=best_result.source,
        chunk_id=best_result.chunk_id,
        score=best_result.score,
    )

    return RAGAnswer(
        answer=(
            "Based on the retrieved document context: "
            f"{best_result.content}"
        ),
        citations=[citation],
        has_context=True,
    )


def format_rag_answer(rag_answer: RAGAnswer) -> str:
    lines = [
        "Answer:",
        rag_answer.answer,
    ]

    if rag_answer.citations:
        lines.append("\nSources:")

        for citation in rag_answer.citations:
            lines.append(
                f"- {citation.source} "
                f"({citation.chunk_id}, score: {citation.score})"
            )

    return "\n".join(lines)
```

Create `main_rag.py`:

```python
from pathlib import Path

from rag_answer import (
    build_rag_prompt,
    create_extractive_answer,
    format_rag_answer,
)
from vector_search import load_chunks, search_chunks


def main() -> None:
    chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"

    chunks = load_chunks(chunks_file)

    question = input("Ask a question about your document: ")

    results = search_chunks(
        chunks,
        question,
        top_k=3,
        minimum_score=0.05,
    )

    rag_prompt = build_rag_prompt(question, results)
    rag_answer = create_extractive_answer(results)

    print("\n" + "=" * 60)
    print(format_rag_answer(rag_answer))

    print("\n" + "=" * 60)
    print("Prompt prepared for a future LLM:")
    print(rag_prompt)


if __name__ == "__main__":
    main()
```

Run:

```bash
python main_rag.py
ruff check .
```

Try:

```text
How does RAG use documents?
Why are embeddings needed?
What should happen with private documents?
What is quantum physics?
```

The final question should return a safe “no relevant information” result.

---

# What you built

```text
vector_search.py
    ↓
retrieved SearchResult objects
    ↓
rag_answer.py
    ├── prompt builder
    ├── grounded answer creator
    └── citation formatter
    ↓
main_rag.py
    ↓
working RAG console application
```

---

# 1–3 Hour Delivery Plan

## 1-hour MVP

```text
[ ] Search chunks for a question
[ ] Return the best matching chunk
[ ] Show source and chunk ID
[ ] Return a clear message when nothing matches
```

## 2-hour version

```text
[ ] Build a strict RAG prompt
[ ] Add answer and citation dataclasses
[ ] Format source-backed answers
[ ] Test multiple relevant and irrelevant questions
```

## 3-hour corporate version

Create `tests/test_rag_answer.py`:

```python
from rag_answer import (
    build_rag_prompt,
    create_extractive_answer,
)
from vector_search import SearchResult


RESULTS = [
    SearchResult(
        chunk_id="rag-001",
        source="rag.txt",
        score=0.82,
        content=(
            "RAG retrieves relevant document chunks before "
            "generating an answer."
        ),
    )
]


def test_build_rag_prompt_includes_question_and_context() -> None:
    prompt = build_rag_prompt(
        "How does RAG work?",
        RESULTS,
    )

    assert "How does RAG work?" in prompt
    assert "rag-001" in prompt
    assert "RAG retrieves relevant document chunks" in prompt


def test_create_extractive_answer_includes_citation() -> None:
    answer = create_extractive_answer(RESULTS)

    assert answer.has_context is True
    assert len(answer.citations) == 1
    assert answer.citations[0].source == "rag.txt"


def test_create_extractive_answer_handles_missing_context() -> None:
    answer = create_extractive_answer([])

    assert answer.has_context is False
    assert answer.citations == []
    assert "could not find relevant information" in answer.answer
```

Run:

```bash
pytest -q
ruff check .
```

---

# Assignment Extension

Add a confidence rule.

Update `create_extractive_answer()` so that:

```text
- Score >= 0.30 → return normal source-backed answer.
- Score below 0.30 → return a low-confidence message.
- Low-confidence answers must still show their source.
```

Example:

```text
“The available context may not fully answer this question.
Please verify the cited source.”
```

---

# Institute, Corporate, and Client Approach

## Institute level

Explain:

```text
What is grounded generation?
Why must RAG show citations?
Why does “no result” matter?
Why is retrieval separate from generation?
What is the role of a confidence threshold?
```

## Corporate level

Before production release, check:

```text
[ ] Is retrieval quality evaluated with real user questions?
[ ] Are sources shown clearly?
[ ] Does the system abstain when context is weak?
[ ] Are document permissions applied before retrieval?
[ ] Are user questions and retrieved documents logged safely?
[ ] Is there monitoring for wrong/irrelevant retrieval?
```

## Client explanation

> “The assistant searches only approved documents before answering. Every answer is tied to a visible source, and when the system lacks relevant information, it says so instead of guessing.”

Day 15 AI will connect this RAG pipeline to a real LLM provider safely using environment variables and an API adapter.
