# Day 4 AI — Extra Programs + Quiz

**Python code is the same on Mac and Windows.**

Run:

```bash
python lessons/file_name.py
ruff check .
```

## Program 1 — Safe integer parser

Create `lessons/day_04_program_01.py`:

```python
def parse_max_tokens(raw_value: str) -> int:
    try:
        max_tokens = int(raw_value)
    except ValueError as error:
        raise ValueError(
            f"max_tokens must be a whole number; received {raw_value!r}."
        ) from error

    if not 1 <= max_tokens <= 4_000:
        raise ValueError("max_tokens must be between 1 and 4000.")

    return max_tokens


for raw_value in ["500", "0", "5000", "abc"]:
    try:
        print(f"{raw_value!r} -> {parse_max_tokens(raw_value)}")
    except ValueError as error:
        print(f"{raw_value!r} -> Error: {error}")
```

## Program 2 — Handle missing dictionary data

Create `lessons/day_04_program_02.py`:

```python
def get_user_prompt(request: dict[str, str]) -> str:
    try:
        prompt = request["prompt"].strip()
    except KeyError as error:
        raise ValueError("Request must include a 'prompt' field.") from error

    if not prompt:
        raise ValueError("Prompt cannot be empty.")

    return prompt


requests = [
    {"prompt": "Explain RAG."},
    {},
    {"prompt": "   "},
]

for request in requests:
    try:
        print(get_user_prompt(request))
    except ValueError as error:
        print(f"Invalid request: {error}")
```

## Program 3 — Logging an AI workflow

Create `lessons/day_04_program_03.py`:

```python
import logging


logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
)

logger = logging.getLogger(__name__)


def process_prompt(prompt: str) -> str:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        logger.warning("Received an empty prompt.")
        raise ValueError("Prompt cannot be empty.")

    logger.info("Processing prompt with %s characters.", len(cleaned_prompt))

    return f"AI response placeholder for: {cleaned_prompt}"


for prompt in ["What is an AI agent?", "   ", "Explain embeddings."]:
    try:
        print(process_prompt(prompt))
    except ValueError as error:
        logger.error("Prompt processing failed: %s", error)
```

## Program 4 — Custom error class

Create `lessons/day_04_program_04.py`:

```python
class InvalidAIRequestError(ValueError):
    """Raised when an AI request does not meet application rules."""


def validate_model(model: str) -> str:
    allowed_models = {"gpt-5", "gpt-4.1", "local-model"}

    if model not in allowed_models:
        raise InvalidAIRequestError(
            f"Unsupported model: {model}. Choose one of {sorted(allowed_models)}."
        )

    return model


for model in ["gpt-5", "unknown-model"]:
    try:
        print(f"Using model: {validate_model(model)}")
    except InvalidAIRequestError as error:
        print(f"Request rejected: {error}")
```

# Day 4 AI MCQ Quiz

Reply like: `1.B 2.C 3.A ...`

1. Which error happens when Python cannot understand code structure?

   A. Logic error  
   B. Syntax error  
   C. Runtime error  
   D. Validation error  

2. Which error means code runs but produces an incorrect result?

   A. Syntax error  
   B. Import error  
   C. Logic error  
   D. Indentation error  

3. Which block contains code that may fail?

   A. `try`  
   B. `except`  
   C. `finally`  
   D. `return`  

4. When does an `else` block attached to `try/except` run?

   A. Always  
   B. Only when an exception occurs  
   C. Only when no exception occurs  
   D. Never  

5. When does `finally` run?

   A. Only after a successful `try`  
   B. Only after an exception  
   C. Always  
   D. Only during testing  

6. Which error commonly occurs for `int("hello")`?

   A. `KeyError`  
   B. `ValueError`  
   C. `TypeError`  
   D. `SyntaxError`  

7. Which error commonly occurs for `data["missing_key"]`?

   A. `KeyError`  
   B. `ValueError`  
   C. `RuntimeError`  
   D. `NameError`  

8. What does `raise ValueError("Invalid input")` do?

   A. Silently ignores invalid input  
   B. Signals a clear error and stops normal function flow  
   C. Deletes invalid data  
   D. Restarts the program  

9. Why is this dangerous?

```python
except Exception:
    pass
```

   A. It makes Python faster  
   B. It hides real errors and makes debugging difficult  
   C. It only handles `ValueError`  
   D. It creates a syntax error  

10. Which logging level is most suitable for a normal successful event?

   A. `DEBUG`  
   B. `INFO`  
   C. `WARNING`  
   D. `ERROR`  

11. Which logging level is suitable for a recoverable unexpected situation?

   A. `WARNING`  
   B. `INFO`  
   C. `DEBUG`  
   D. `RETURN`  

12. Which logging level is suitable when an operation fails?

   A. `INFO`  
   B. `DEBUG`  
   C. `ERROR`  
   D. `PASS`  

13. What should never be written to logs?

   A. Timestamp  
   B. Error message  
   C. API key or password  
   D. Log level  

14. Why create `InvalidAIRequestError(ValueError)`?

   A. To make errors impossible  
   B. To represent a clear application-specific validation problem  
   C. To replace all Python errors  
   D. To install an AI model  

15. What should you read first in a Python traceback?

   A. The final line showing the error type and message  
   B. The first import statement  
   C. Only the file name  
   D. Nothing; rerun immediately
