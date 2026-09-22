# Day 5 AI — Testing with Pytest

Today you learn how professional developers prove their code works. This is vital for AI systems because validation, prompt building, tool calls, and RAG pipelines must stay reliable as code changes.

## Start environment

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

---

## 1. Create this structure

```text
python-ai-mastery/
├── ai_core/
│   ├── __init__.py
│   └── request_tools.py
└── tests/
    └── test_request_tools.py
```

`__init__.py` can be empty. It tells Python that `ai_core` is a package.

---

## 2. Create the code to test

Put this in `ai_core/request_tools.py`:

```python
from typing import Any


class InvalidAIRequestError(ValueError):
    """Raised when an AI request has invalid data."""


def create_ai_request(
    prompt: str,
    *,
    model: str = "gpt-5",
    temperature: float = 0.7,
    max_tokens: int = 500,
) -> dict[str, Any]:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        raise InvalidAIRequestError("Prompt cannot be empty.")

    if not 0 <= temperature <= 2:
        raise InvalidAIRequestError("Temperature must be between 0 and 2.")

    if not 1 <= max_tokens <= 4_000:
        raise InvalidAIRequestError("max_tokens must be between 1 and 4000.")

    return {
        "model": model,
        "messages": [
            {
                "role": "user",
                "content": cleaned_prompt,
            }
        ],
        "temperature": temperature,
        "max_tokens": max_tokens,
    }
```

---

## 3. Write automated tests

Put this in `tests/test_request_tools.py`:

```python
import pytest

from ai_core.request_tools import InvalidAIRequestError, create_ai_request


def test_create_ai_request_returns_expected_data() -> None:
    request = create_ai_request(
        " Explain RAG simply. ",
        model="gpt-5",
        temperature=0.5,
        max_tokens=300,
    )

    assert request["model"] == "gpt-5"
    assert request["temperature"] == 0.5
    assert request["max_tokens"] == 300
    assert request["messages"][0]["role"] == "user"
    assert request["messages"][0]["content"] == "Explain RAG simply."


def test_create_ai_request_rejects_empty_prompt() -> None:
    with pytest.raises(InvalidAIRequestError, match="Prompt cannot be empty"):
        create_ai_request("   ")


@pytest.mark.parametrize("temperature", [-0.1, 2.1, 5])
def test_create_ai_request_rejects_invalid_temperature(
    temperature: float,
) -> None:
    with pytest.raises(InvalidAIRequestError, match="Temperature"):
        create_ai_request("Explain embeddings.", temperature=temperature)


@pytest.mark.parametrize("max_tokens", [0, -1, 4_001])
def test_create_ai_request_rejects_invalid_max_tokens(
    max_tokens: int,
) -> None:
    with pytest.raises(InvalidAIRequestError, match="max_tokens"):
        create_ai_request("Explain AI agents.", max_tokens=max_tokens)
```

---

## 4. Run tests

```bash
pytest -q
```

Expected:

```text
6 passed
```

Then run:

```bash
ruff check .
```

## Core testing idea: AAA

Every test usually follows:

```text
Arrange → create the input/data
Act    → call the function
Assert → verify the output or error
```

Example:

```python
def test_prompt_is_trimmed() -> None:
    request = create_ai_request("  Hello  ")

    assert request["messages"][0]["content"] == "Hello"
```

## Why `pytest.raises()` matters

This test proves invalid input is rejected correctly:

```python
with pytest.raises(InvalidAIRequestError):
    create_ai_request("")
```

A test should verify both:

- valid behavior works
- invalid behavior fails in the intended, safe way

## Day 5 challenge

Add this test:

```python
def test_create_ai_request_uses_default_values() -> None:
```

Check that when only a prompt is supplied:

```python
request = create_ai_request("What is a vector database?")
```

It uses:

```python
model == "gpt-5"
temperature == 0.7
max_tokens == 500
```

## Key takeaway

```text
pytest            → runs tests
assert            → checks an expected result
pytest.raises()   → checks expected errors
parametrize       → runs one test against multiple inputs
```

Day 6 AI will cover **classes, OOP, dataclasses, and modelling an AI agent’s state cleanly.**
