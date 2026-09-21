# Day 4 AI — Errors, Debugging, and Logging

AI systems receive unpredictable input: empty prompts, invalid settings, failed APIs, unavailable databases, and malformed documents. Today you learn how to handle problems safely instead of letting your app crash.

## Start your environment

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

All Python code is the same on Mac and Windows.

---

## 1. Three kinds of errors

```text
Syntax error  → Python cannot understand the code.
Runtime error → Code starts but fails while running.
Logic error   → Code runs but gives the wrong result.
```

Example runtime error:

```python
amount = int("hello")  # ValueError
```

Example logic error:

```python
total = 100
discount = 10
final_price = total + discount  # Runs, but wrong business logic.
```

---

## 2. `try`, `except`, `else`, and `finally`

```python
raw_temperature = "0.7"

try:
    temperature = float(raw_temperature)
except ValueError:
    print("Temperature must be a number.")
else:
    print(f"Valid temperature: {temperature}")
finally:
    print("Validation finished.")
```

- `try`: code that might fail
- `except`: handle a known failure
- `else`: runs only when no error occurs
- `finally`: always runs; useful for cleanup

Do not hide all errors like this:

```python
except Exception:
    pass
```

It hides the real problem and makes debugging difficult.

---

## 3. Raise clear, useful errors

```python
def validate_temperature(temperature: float) -> None:
    if not 0 <= temperature <= 2:
        raise ValueError(
            f"Temperature must be between 0 and 2; received {temperature}."
        )
```

A useful error tells you:

```text
What failed?
What value caused it?
What is the valid rule?
```

---

## 4. Custom errors

Use custom errors when your app needs to distinguish domain problems.

```python
class InvalidAIRequestError(ValueError):
    """Raised when an AI request has invalid data."""
```

```python
raise InvalidAIRequestError("User prompt cannot be empty.")
```

Later, FastAPI will turn these into clean API responses such as `400 Bad Request`.

---

## 5. Logging instead of only `print()`

`print()` is useful while learning. Real applications use logging because logs can include timestamps, severity levels, and production records.

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Preparing AI request.")
logger.warning("Prompt is unusually long.")
logger.error("Failed to validate request.")
```

Common levels:

```text
DEBUG    → detailed developer information
INFO     → normal application events
WARNING  → unexpected but recoverable situation
ERROR    → operation failed
```

Never log API keys, passwords, tokens, or private user data.

---

# Day 4 AI Program — Resilient Request Validator

Create `lessons/day_04_error_handling.py`:

```python
import logging
from typing import Any


logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s: %(message)s",
)

logger = logging.getLogger(__name__)


class InvalidAIRequestError(ValueError):
    """Raised when an AI request has invalid data."""


def parse_temperature(raw_temperature: str) -> float:
    try:
        temperature = float(raw_temperature)
    except ValueError as error:
        raise InvalidAIRequestError(
            f"Temperature must be a number; received {raw_temperature!r}."
        ) from error

    if not 0 <= temperature <= 2:
        raise InvalidAIRequestError(
            f"Temperature must be between 0 and 2; received {temperature}."
        )

    return temperature


def build_ai_request(
    prompt: str,
    raw_temperature: str,
) -> dict[str, Any]:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        raise InvalidAIRequestError("Prompt cannot be empty.")

    temperature = parse_temperature(raw_temperature)

    logger.info("AI request validated successfully.")

    return {
        "model": "gpt-5",
        "messages": [
            {
                "role": "user",
                "content": cleaned_prompt,
            }
        ],
        "temperature": temperature,
    }


test_inputs = [
    ("Explain RAG simply.", "0.5"),
    ("", "0.7"),
    ("Explain AI agents.", "creative"),
    ("Explain embeddings.", "3"),
]

for prompt, raw_temperature in test_inputs:
    try:
        request = build_ai_request(prompt, raw_temperature)
        print(f"\nCreated request: {request}")
    except InvalidAIRequestError as error:
        logger.error("Could not create AI request: %s", error)
```

Run it:

```bash
python lessons/day_04_error_handling.py
ruff check .
```

The program should create one valid request and safely log errors for the other three inputs.

## Debugging checklist

When code fails:

1. Read the final line of the traceback first.
2. Find the file name and line number.
3. Read the error type: `ValueError`, `KeyError`, `TypeError`, etc.
4. Print or inspect the actual input value.
5. Fix the cause, not only the visible symptom.
6. Add validation or a test so it does not return.

## Day 4 challenge

Add a `max_tokens` parameter to `build_ai_request()`.

Rules:

```text
- It must be an integer.
- It must be greater than 0.
- It must not exceed 4,000.
- Invalid values must raise InvalidAIRequestError.
```

Day 5 AI will cover **testing with pytest**: how to prove that your AI functions work and stay correct as your code grows.
