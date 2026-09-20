# Day 3 AI — Functions as Reliable Building Blocks

Today you will learn how to write functions like an AI/backend engineer: clear inputs, validation, predictable outputs, and no hidden side effects.

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

All Python code is the same on both systems.

---

## 1. A function is a contract

```python
def build_prompt(topic: str, level: str) -> str:
    return f"Explain {topic} for a {level} learner."
```

Contract:

```text
Input: topic and level must be strings
Output: one string prompt
Job: create a prompt only
```

Good functions are small, clear, and do one job.

---

## 2. Positional and keyword arguments

```python
def create_model_config(model: str, temperature: float) -> dict[str, object]:
    return {
        "model": model,
        "temperature": temperature,
    }
```

Positional call:

```python
config = create_model_config("gpt-5", 0.7)
```

Keyword call — clearer in professional code:

```python
config = create_model_config(
    model="gpt-5",
    temperature=0.7,
)
```

---

## 3. Default values

```python
def build_prompt(
    topic: str,
    level: str = "beginner",
) -> str:
    return f"Explain {topic} for a {level} learner."
```

```python
print(build_prompt("RAG"))
print(build_prompt("Embeddings", level="advanced"))
```

Defaults are useful when a value has a safe, common choice.

---

## 4. Keyword-only arguments

Use `*` to force important optional settings to be named:

```python
def create_llm_config(
    model: str,
    *,
    temperature: float = 0.7,
    max_tokens: int = 500,
) -> dict[str, object]:
    return {
        "model": model,
        "temperature": temperature,
        "max_tokens": max_tokens,
    }
```

Correct:

```python
config = create_llm_config(
    "gpt-5",
    temperature=0.3,
    max_tokens=300,
)
```

Wrong:

```python
# create_llm_config("gpt-5", 0.3, 300)
```

This avoids mistakes such as accidentally passing `300` as `temperature`.

---

## 5. Validation and raising errors

A function should reject invalid input early.

```python
def validate_temperature(temperature: float) -> None:
    if not 0 <= temperature <= 2:
        raise ValueError("Temperature must be between 0 and 2.")
```

`raise` stops normal execution and clearly explains what is wrong. Later, APIs and AI agents will catch and handle these errors safely.

---

## 6. Local scope

Variables created inside a function belong only to that function.

```python
def make_message() -> str:
    message = "Hello from inside the function."
    return message


print(make_message())

# print(message)  # NameError: message exists only inside the function.
```

Avoid `global` variables in serious applications. Pass data into functions and return data out.

---

# Day 3 AI Program — Build a Safe LLM Request

Create `lessons/day_03_llm_request_builder.py`:

```python
from typing import Any


VALID_ROLES = {"system", "user", "assistant"}


def validate_message(role: str, content: str) -> dict[str, str]:
    cleaned_content = content.strip()

    if role not in VALID_ROLES:
        raise ValueError(f"Invalid role: {role!r}")

    if not cleaned_content:
        raise ValueError("Message content cannot be empty.")

    return {
        "role": role,
        "content": cleaned_content,
    }


def create_llm_request(
    system_instruction: str,
    user_prompt: str,
    *,
    model: str = "gpt-5",
    temperature: float = 0.7,
    max_tokens: int = 500,
) -> dict[str, Any]:
    if not 0 <= temperature <= 2:
        raise ValueError("Temperature must be between 0 and 2.")

    if max_tokens <= 0:
        raise ValueError("max_tokens must be greater than 0.")

    system_message = validate_message("system", system_instruction)
    user_message = validate_message("user", user_prompt)

    return {
        "model": model,
        "messages": [system_message, user_message],
        "temperature": temperature,
        "max_tokens": max_tokens,
    }


request = create_llm_request(
    system_instruction="You are a helpful Python and AI tutor.",
    user_prompt="Explain RAG with a simple example.",
    model="gpt-5",
    temperature=0.4,
    max_tokens=300,
)

print("LLM request created successfully:\n")

for key, value in request.items():
    print(f"{key}: {value}")
```

Run:

```bash
python lessons/day_03_llm_request_builder.py
ruff check .
```

## Day 3 challenge

Add this function to the same file:

```python
def estimate_request_size(request: dict[str, Any]) -> int:
```

It should return the total number of characters across every message’s `content`.

For example:

```python
request = {
    "messages": [
        {"role": "system", "content": "You are helpful."},
        {"role": "user", "content": "What is RAG?"},
    ]
}
```

It should calculate:

```python
len("You are helpful.") + len("What is RAG?")
```

## Key takeaway

```text
Functions = reusable contracts
Type hints = communicate expected data
Validation = reject bad input early
Keyword-only arguments = prevent configuration mistakes
Local scope = safer, predictable code
```

Day 4 AI will cover **error handling, debugging, logging, and writing tests**—the skills that make AI applications reliable.
