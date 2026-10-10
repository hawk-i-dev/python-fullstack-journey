# Day 15 AI — Extra Programs + Quiz

The official OpenAI Python documentation identifies the Responses API as the primary API, recommends loading `OPENAI_API_KEY` through environment configuration, and supports `python-dotenv` for local `.env` files. [OpenAI Python API reference](https://developers.openai.com/api/reference/python)

## Program 1 — Validate prompt size before any API call

Create `prompt_validator.py`:

```python
def validate_prompt_size(
    prompt: str,
    maximum_characters: int = 12_000,
) -> str:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        raise ValueError("Prompt cannot be empty.")

    if maximum_characters < 1:
        raise ValueError(
            "maximum_characters must be at least 1."
        )

    if len(cleaned_prompt) > maximum_characters:
        raise ValueError(
            "Prompt exceeds the allowed character limit."
        )

    return cleaned_prompt
```

Create `tests/test_prompt_validator.py`:

```python
import pytest

from prompt_validator import validate_prompt_size


def test_validate_prompt_size_returns_clean_prompt() -> None:
    assert validate_prompt_size("  Explain RAG.  ") == "Explain RAG."


def test_validate_prompt_size_rejects_empty_prompt() -> None:
    with pytest.raises(ValueError, match="cannot be empty"):
        validate_prompt_size("   ")


def test_validate_prompt_size_rejects_large_prompt() -> None:
    with pytest.raises(ValueError, match="exceeds"):
        validate_prompt_size("a" * 11, maximum_characters=10)
```

## Program 2 — Safe provider configuration

Create `provider_config.py`:

```python
import os
from dataclasses import dataclass

from dotenv import load_dotenv


load_dotenv()


@dataclass(frozen=True)
class ProviderConfig:
    model: str
    has_api_key: bool


def load_provider_config() -> ProviderConfig:
    api_key = os.getenv("OPENAI_API_KEY", "")
    model = os.getenv("OPENAI_MODEL", "gpt-5.5").strip()

    if not model:
        raise ValueError("OPENAI_MODEL cannot be empty.")

    return ProviderConfig(
        model=model,
        has_api_key=bool(api_key),
    )


if __name__ == "__main__":
    config = load_provider_config()

    print(f"Model: {config.model}")
    print(f"API key configured: {config.has_api_key}")
```

Notice: it prints only whether a key exists, never the key itself.

## Program 3 — Add timeout and retry configuration

Update the real client creation inside `llm_client.py`:

```python
client = OpenAI(
    api_key=api_key,
    timeout=30.0,
    max_retries=2,
)
```

A timeout prevents the application from waiting forever. Limited retries help handle temporary connection or rate-limit failures. The SDK documents timeout settings, retries, request IDs, and API error handling. [OpenAI Python API reference](https://developers.openai.com/api/reference/python)

## Program 4 — Test missing API key without a real call

Add this to `tests/test_llm_client.py`:

```python
import pytest

from llm_client import LLMProviderError, generate_grounded_answer


def test_generate_grounded_answer_rejects_empty_prompt() -> None:
    with pytest.raises(ValueError, match="Prompt cannot be empty"):
        generate_grounded_answer(
            "   ",
            client=object(),
            model="test-model",
        )


def test_generate_grounded_answer_requires_key_without_client(
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    monkeypatch.delenv("OPENAI_API_KEY", raising=False)

    with pytest.raises(LLMProviderError, match="OPENAI_API_KEY is missing"):
        generate_grounded_answer(
            "Use only retrieved context.",
            model="test-model",
        )
```

Run:

```bash
pytest -q
ruff check .
```

# Day 15 AI MCQ Quiz

Reply like:

```text
1.B 2.A 3.C ...
```

1. Why should an API key be stored in an environment variable or `.env` file?

   A. It makes the key public  
   B. It avoids hard-coding secrets in source code  
   C. It removes the need for `.gitignore`  
   D. It guarantees free API use  

2. Which file must normally be ignored by Git?

   A. `.env`  
   B. `README.md`  
   C. `main.py`  
   D. `tests/`  

3. Which API is the primary API in the current OpenAI Python documentation?

   A. Files API  
   B. Chat Completions API only  
   C. Responses API  
   D. Images API only  

4. Which code sends a request through the Python Responses API?

   A. `client.responses.create(...)`  
   B. `client.start_model(...)`  
   C. `OpenAI.generate(...)`  
   D. `response.ask(...)`  

5. Which response property provides generated text in the Day 15 code?

   A. `response.output_text`  
   B. `response.json_file`  
   C. `response.embedding`  
   D. `response.commit_message`  

6. Why use a fake client in automated tests?

   A. To make a real paid API call  
   B. To test application logic without network access or API cost  
   C. To expose the API key  
   D. To avoid writing assertions  

7. What should happen when `OPENAI_API_KEY` is missing?

   A. The app should silently use a random key  
   B. The app should raise a clear configuration error  
   C. The key should be written into source code  
   D. The app should ignore all validation  

8. Why should the RAG app avoid calling the LLM when no relevant context is found?

   A. To reduce unsupported or hallucinated answers  
   B. To make the application slower  
   C. To delete document chunks  
   D. To avoid source citations  

9. What can cause a `RateLimitError`?

   A. Using a Python list  
   B. Request or account limits being reached  
   C. An empty Git repository  
   D. A Pandas DataFrame  

10. Why configure an API timeout?

   A. To prevent the app from waiting indefinitely for a response  
   B. To expose credentials  
   C. To remove tests  
   D. To force every request to fail  

11. Why configure limited retries?

   A. To handle temporary connection or service failures  
   B. To guarantee the LLM is always correct  
   C. To bypass access controls  
   D. To replace validation  

12. What must never be printed in logs?

   A. Error category  
   B. API key, password, or access token  
   C. Request ID  
   D. Model name  

13. What should be shown to a user alongside a RAG answer?

   A. The approved source/chunk information used as context  
   B. The API key  
   C. Internal server passwords  
   D. Only a similarity score  

14. Why keep `llm_client.py` separate from `main_live_rag.py`?

   A. It makes provider calls easier to test, replace, and maintain  
   B. It prevents all errors  
   C. It removes the need for retrieval  
   D. It makes `.env` public  

15. What is the safest way to test RAG application behavior?

   A. Use fake clients and controlled test chunks  
   B. Use production client data without permission  
   C. Commit the `.env` file  
   D. Remove error handling
