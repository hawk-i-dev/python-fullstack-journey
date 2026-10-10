# Day 15 AI — Connect RAG to a Real LLM Safely

Today you add a real LLM provider adapter to your RAG pipeline.

```text
Question
→ vector search
→ relevant chunks
→ grounded prompt
→ LLM API
→ answer + visible source details
```

We’ll use the OpenAI Python SDK and the Responses API. The official SDK reads `OPENAI_API_KEY` from the environment, and the official quickstart shows the `client.responses.create(...)` pattern. [OpenAI Developer Quickstart](https://developers.openai.com/api/docs/quickstart)

> Running live API calls may use credits on your API account. The project still works offline for tests because tests use a fake client.

## Start your environment

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

Install or update dependencies:

```bash
pip install --upgrade openai python-dotenv
```

---

## 1. Secure your API key

Create `.env` in the project root:

```env
OPENAI_API_KEY=your_real_api_key_here
OPENAI_MODEL=gpt-5.5
```

Do not share, commit, screenshot, or paste the key into code/chat.

Confirm `.gitignore` contains:

```gitignore
.env
```

Also create `.env.example`:

```env
OPENAI_API_KEY=
OPENAI_MODEL=gpt-5.5
```

The empty example file is safe to commit; the real `.env` file is not.

---

# Day 15 Project — Live RAG LLM Adapter

Create `llm_client.py`:

```python
import os
from typing import Any

from dotenv import load_dotenv
from openai import (
    APIConnectionError,
    APIError,
    AuthenticationError,
    OpenAI,
    RateLimitError,
)


load_dotenv()


class LLMProviderError(RuntimeError):
    """Raised when the LLM provider cannot generate an answer."""


def generate_grounded_answer(
    prompt: str,
    *,
    client: Any | None = None,
    model: str | None = None,
) -> str:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        raise ValueError("Prompt cannot be empty.")

    selected_model = model or os.getenv(
        "OPENAI_MODEL",
        "gpt-5.5",
    )

    if client is None:
        api_key = os.getenv("OPENAI_API_KEY")

        if not api_key:
            raise LLMProviderError(
                "OPENAI_API_KEY is missing. Add it to your .env file."
            )

        client = OpenAI(api_key=api_key)

    try:
        response = client.responses.create(
            model=selected_model,
            input=cleaned_prompt,
        )
    except AuthenticationError as error:
        raise LLMProviderError(
            "Authentication failed. Check your API key."
        ) from error
    except RateLimitError as error:
        raise LLMProviderError(
            "Rate limit or account limit reached. Try again later."
        ) from error
    except APIConnectionError as error:
        raise LLMProviderError(
            "Could not connect to the LLM provider."
        ) from error
    except APIError as error:
        raise LLMProviderError(
            "The LLM provider returned an API error."
        ) from error

    answer = response.output_text.strip()

    if not answer:
        raise LLMProviderError(
            "The LLM provider returned an empty answer."
        )

    return answer
```

Create `main_live_rag.py`:

```python
from pathlib import Path

from llm_client import LLMProviderError, generate_grounded_answer
from rag_answer import build_rag_prompt
from vector_search import load_chunks, search_chunks


def main() -> None:
    chunks_file = Path("data") / "processed" / "ai_rag_chunks.json"

    chunks = load_chunks(chunks_file)

    question = input("Ask a question about your documents: ")

    results = search_chunks(
        chunks,
        question,
        top_k=3,
        minimum_score=0.05,
    )

    if not results:
        print("\nNo relevant document context was found.")
        print("No LLM request was made.")
        return

    prompt = build_rag_prompt(question, results)

    try:
        answer = generate_grounded_answer(prompt)
    except LLMProviderError as error:
        print(f"\nCould not generate an LLM answer: {error}")
        return

    print("\nAnswer:")
    print(answer)

    print("\nRetrieved sources given to the model:")

    for result in results:
        print(
            f"- {result.source} | {result.chunk_id} "
            f"| score: {result.score}"
        )


if __name__ == "__main__":
    main()
```

Run:

```bash
python main_live_rag.py
ruff check .
```

Ask:

```text
How does RAG use document chunks?
Why are embeddings useful?
What should a RAG application do with private documents?
```

---

# Why the adapter is separate

```text
main_live_rag.py → application flow
rag_answer.py    → grounded prompt rules
vector_search.py → retrieval
llm_client.py    → external LLM API communication
```

This makes the project easier to test, swap providers, debug, and maintain.

---

# 1–3 Hour Delivery Plan

## 1-hour MVP

```text
[ ] Create .env and .env.example
[ ] Add llm_client.py
[ ] Send a grounded prompt to the LLM
[ ] Display answer and retrieved sources
```

## 2-hour version

```text
[ ] Handle missing API key
[ ] Handle authentication, rate-limit, connection, and API errors
[ ] Do not call the LLM when retrieval finds no context
[ ] Confirm .env is ignored by Git
```

## 3-hour corporate version

Create `tests/test_llm_client.py`:

```python
from llm_client import generate_grounded_answer


class FakeResponse:
    output_text = "RAG retrieves relevant context before answering."


class FakeResponses:
    def __init__(self) -> None:
        self.last_model: str | None = None
        self.last_input: str | None = None

    def create(self, *, model: str, input: str) -> FakeResponse:
        self.last_model = model
        self.last_input = input
        return FakeResponse()


class FakeClient:
    def __init__(self) -> None:
        self.responses = FakeResponses()


def test_generate_grounded_answer_uses_supplied_client() -> None:
    client = FakeClient()

    answer = generate_grounded_answer(
        "Use only retrieved context.",
        client=client,
        model="test-model",
    )

    assert answer == "RAG retrieves relevant context before answering."
    assert client.responses.last_model == "test-model"
    assert client.responses.last_input == "Use only retrieved context."
```

Run:

```bash
pytest -q
ruff check .
```

This test makes no paid API call.

---

# Assignment Extension

Add a function to `llm_client.py`:

```python
def validate_prompt_size(
    prompt: str,
    maximum_characters: int = 12_000,
) -> str:
```

Requirements:

```text
- Reject empty prompts.
- Reject prompts longer than maximum_characters.
- Return the cleaned valid prompt.
- Add pytest tests.
```

Why? Large retrieved context can increase latency and cost. Real RAG systems must manage prompt size.

---

# Institute, Corporate, and Client Approach

## Institute level

Explain:

```text
Why environment variables are safer than hard-coded keys.
Why API code belongs in its own adapter.
Why tests use a fake client.
Why a RAG system should not call an LLM without relevant context.
```

## Corporate level

```text
[ ] Secrets are in a secret manager or environment variables.
[ ] .env is ignored by Git.
[ ] API calls have error handling and timeouts.
[ ] Logs never include API keys or sensitive prompt content.
[ ] Retrieval quality is evaluated separately from LLM quality.
[ ] Usage, latency, failures, and cost are monitored.
[ ] Tests never require a real API key.
```

## Client explanation

> “The assistant first retrieves information from your approved documents. Only then does it ask the language model to produce an answer. The user can see which document chunks were supplied, and the system does not call the model when no relevant internal information is found.”

Day 16 AI will add **conversation memory and multi-turn RAG chat**, so the assistant can understand follow-up questions safely.
