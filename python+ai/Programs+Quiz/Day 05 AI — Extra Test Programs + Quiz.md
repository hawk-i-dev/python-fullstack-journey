# Day 5 AI — Extra Test Programs + Quiz

**Python code and test commands are the same on Mac and Windows** after activating `.venv`.

```bash
pytest -q
ruff check .
```

## Program 1 — Test default LLM settings

Add this to `tests/test_request_tools.py`:

```python
def test_create_ai_request_uses_default_values() -> None:
    request = create_ai_request("What is a vector database?")

    assert request["model"] == "gpt-5"
    assert request["temperature"] == 0.7
    assert request["max_tokens"] == 500
```

## Program 2 — Test that prompt whitespace is removed

Add this to `tests/test_request_tools.py`:

```python
def test_create_ai_request_trims_prompt_whitespace() -> None:
    request = create_ai_request("   Explain AI agents.   ")

    assert request["messages"][0]["content"] == "Explain AI agents."
```

## Program 3 — Test a simple token validator

Create `ai_core/token_tools.py`:

```python
class InvalidTokenLimitError(ValueError):
    """Raised when a token limit is outside the allowed range."""


def validate_token_limit(max_tokens: int) -> int:
    if not 1 <= max_tokens <= 4_000:
        raise InvalidTokenLimitError(
            "max_tokens must be between 1 and 4000."
        )

    return max_tokens
```

Create `tests/test_token_tools.py`:

```python
import pytest

from ai_core.token_tools import InvalidTokenLimitError, validate_token_limit


def test_validate_token_limit_accepts_valid_value() -> None:
    assert validate_token_limit(500) == 500


@pytest.mark.parametrize("max_tokens", [0, -1, 4_001])
def test_validate_token_limit_rejects_invalid_values(
    max_tokens: int,
) -> None:
    with pytest.raises(InvalidTokenLimitError, match="max_tokens"):
        validate_token_limit(max_tokens)
```

## Program 4 — Test conversation-message creation

Create `ai_core/message_tools.py`:

```python
class InvalidMessageError(ValueError):
    """Raised when a message is invalid."""


def create_message(role: str, content: str) -> dict[str, str]:
    valid_roles = {"system", "user", "assistant"}
    cleaned_content = content.strip()

    if role not in valid_roles:
        raise InvalidMessageError("Role is invalid.")

    if not cleaned_content:
        raise InvalidMessageError("Content cannot be empty.")

    return {
        "role": role,
        "content": cleaned_content,
    }
```

Create `tests/test_message_tools.py`:

```python
import pytest

from ai_core.message_tools import InvalidMessageError, create_message


def test_create_message_returns_clean_message() -> None:
    message = create_message("user", "  What is RAG?  ")

    assert message == {
        "role": "user",
        "content": "What is RAG?",
    }


@pytest.mark.parametrize("role", ["admin", "", "bot"])
def test_create_message_rejects_invalid_role(role: str) -> None:
    with pytest.raises(InvalidMessageError, match="Role is invalid"):
        create_message(role, "Hello")
```

Run all tests:

```bash
pytest -q
```

# Day 5 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. What is the main purpose of automated tests?

   A. Make code longer  
   B. Verify code behavior automatically  
   C. Replace Python  
   D. Hide errors  

2. Which command runs pytest tests quietly?

   A. `pytest -q`  
   B. `python -q`  
   C. `ruff test`  
   D. `pip test`  

3. What naming pattern does pytest commonly discover by default?

   A. `check_*.py`  
   B. `program_*.py`  
   C. `test_*.py`  
   D. `run_*.py`  

4. What does `assert` do in a test?

   A. Installs a dependency  
   B. Checks that an expected condition is true  
   C. Prints every variable  
   D. Starts the application server  

5. If an `assert` condition is false, pytest usually:

   A. Marks the test as failed  
   B. Deletes the test  
   C. Ignores it  
   D. Reinstalls Python  

6. What does AAA stand for in a test?

   A. Add, Apply, Abort  
   B. Arrange, Act, Assert  
   C. Agent, API, AI  
   D. Always, Again, After  

7. What is the correct tool for testing an expected error?

   A. `print(error)`  
   B. `pytest.raises(...)`  
   C. `assert False`  
   D. `break`  

8. What does this verify?

```python
with pytest.raises(ValueError):
    validate_token_limit(0)
```

   A. `validate_token_limit(0)` returns zero  
   B. The function raises `ValueError`  
   C. The test is skipped  
   D. The token limit is fixed automatically  

9. What is `@pytest.mark.parametrize` useful for?

   A. Installing parameters  
   B. Running one test with multiple input values  
   C. Converting all values to strings  
   D. Creating a database  

10. Why should tests include invalid input?

   A. To prove validation rejects bad data safely  
   B. To make every test fail  
   C. To avoid writing functions  
   D. To remove error handling  

11. What should a good test normally verify?

   A. Only that code has many lines  
   B. One clear behavior  
   C. Every project feature in one test  
   D. The Python version only  

12. What is a regression?

   A. A new bug in behavior that previously worked  
   B. A successful test  
   C. A Python keyword  
   D. A type hint  

13. Why are tests important in AI applications?

   A. AI code never changes  
   B. Prompt, validation, and tool logic must remain reliable  
   C. LLMs automatically write all tests  
   D. Tests remove the need for input validation  

14. What is the expected result of this?

```python
assert create_message("user", " Hi ")["content"] == "Hi"
```

   A. It verifies that whitespace is removed  
   B. It creates a database  
   C. It rejects every user message  
   D. It checks only the role  

15. When should you run tests?

   A. Only after deployment  
   B. Only when an error happens  
   C. Whenever you add or change behavior  
   D. Never if the code looks correct
