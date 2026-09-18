# Day 1 AI — How Python Really Works

Today’s goal: understand variables, objects, types, references, mutability, and functions. These concepts are behind nearly every Python, backend, data, and AI bug.

Before starting, activate your environment:

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
```

Create `lessons/day_01_python_objects.py`.

## 1. Variables are labels, not boxes

Many beginners think a variable “contains” a value. In Python, a variable is better understood as a **label pointing to an object**.

```python
model_name = "gpt-5"
temperature = 0.7
is_active = True
```

```text
model_name ──→ "gpt-5"     (string object)
temperature ─→ 0.7         (float object)
is_active ───→ True        (boolean object)
```

Check the object’s type and identity:

```python
model_name = "gpt-5"

print(model_name)
print(type(model_name))
print(id(model_name))
```

`id()` identifies an object during the current program run. Do not rely on its number in application logic; use it only to learn/debug.

## 2. Python is dynamically typed

You do not declare types manually:

```python
message = "Hello"
message = 100
```

This is allowed, but avoid it in professional code because it reduces clarity.

Prefer clear, stable types:

```python
model_name: str = "gpt-5"
max_tokens: int = 500
temperature: float = 0.7
use_memory: bool = True
```

Type hints do not force Python at runtime, but they help VS Code, linters, tests, and teammates understand your code.

## 3. Mutable vs immutable objects

This is one of Python’s most important concepts.

| Immutable — cannot change in place | Mutable — can change in place |
|---|---|
| `str` | `list` |
| `int` | `dict` |
| `float` | `set` |
| `bool` | custom class instances, usually |
| `tuple` | |

### Immutable example

```python
prompt = "Explain Python"
print(id(prompt))

prompt = prompt + " simply"
print(id(prompt))
```

The second line creates a **new string object** and moves `prompt` to it.

### Mutable example

```python
messages = ["Hello"]
print(id(messages))

messages.append("How can I help?")
print(messages)
print(id(messages))
```

The list changes, but it remains the same object.

## 4. References: the source of many bugs

```python
first_messages = ["Hello", "What is AI?"]
second_messages = first_messages

second_messages.append("Explain embeddings")

print(first_messages)
```

Output:

```python
['Hello', 'What is AI?', 'Explain embeddings']
```

Why? Both variables point to the same list.

```text
first_messages ──┐
                 ├──→ ["Hello", "What is AI?"]
second_messages ─┘
```

### Fix: create a copy

```python
first_messages = ["Hello", "What is AI?"]
second_messages = first_messages.copy()

second_messages.append("Explain embeddings")

print(first_messages)
print(second_messages)
```

For nested lists/dictionaries, use `copy.deepcopy()` later. A normal `.copy()` is a shallow copy.

## 5. Lists vs tuples vs dictionaries

### List: ordered, mutable collection

```python
skills = ["Python", "SQL", "FastAPI"]
skills.append("Machine Learning")

print(skills)
```

Use a list when order matters and items may change.

### Tuple: ordered, immutable collection

```python
supported_languages = ("Python", "JavaScript", "SQL")
```

Use a tuple for values that should not change.

### Dictionary: named data

```python
ai_request = {
    "model": "gpt-5",
    "prompt": "Explain vector databases",
    "temperature": 0.7,
    "max_tokens": 300,
}

print(ai_request["model"])
```

AI applications constantly use dictionaries because API requests and JSON data have named fields.

## 6. Functions: reusable behavior

A function should do one clear job.

```python
def build_prompt(topic: str, level: str) -> str:
    return f"Explain {topic} for a {level} learner with one example."
```

Use it:

```python
prompt = build_prompt("Python dictionaries", "beginner")
print(prompt)
```

Read this signature like English:

```python
def build_prompt(topic: str, level: str) -> str:
```

- Function name: `build_prompt`
- Inputs: `topic`, `level`
- Both inputs should be strings
- Output should be a string

## 7. Important senior-level rule: never use mutable default values

Wrong:

```python
def add_message(message: str, history: list[str] = []):
    history.append(message)
    return history
```

That same list can be reused between calls.

Correct:

```python
def add_message(message: str, history: list[str] | None = None) -> list[str]:
    if history is None:
        history = []

    history.append(message)
    return history
```

This pattern matters in chatbot conversation history, API services, and AI agents.

## 8. Day 1 practical task — AI request builder

Add this complete code to `lessons/day_01_python_objects.py`:

```python
from typing import Any


def build_ai_request(
    prompt: str,
    model: str = "gpt-5",
    temperature: float = 0.7,
    max_tokens: int = 300,
) -> dict[str, Any]:
    cleaned_prompt = prompt.strip()

    if not cleaned_prompt:
        raise ValueError("Prompt cannot be empty.")

    if not 0 <= temperature <= 2:
        raise ValueError("Temperature must be between 0 and 2.")

    if max_tokens <= 0:
        raise ValueError("max_tokens must be greater than 0.")

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


request = build_ai_request(
    prompt=" Explain Python lists with an AI example. ",
    temperature=0.5,
    max_tokens=250,
)

print(request)
print(request["messages"][0]["content"])
```

Run it:

```powershell
python lessons/day_01_python_objects.py
ruff check .
```

## Day 1 challenge

Add a function named `add_conversation_message()` that accepts:

- `role`: `"user"` or `"assistant"`
- `content`: a non-empty string
- `history`: an optional list of messages

It must return an updated conversation history without using a mutable default argument.

## Quick self-check

1. Is a Python variable a box containing a value, or a label referencing an object?
2. Is a list mutable or immutable?
3. Why do `second = first` and `second = first.copy()` behave differently?
4. Why is `items: list = []` dangerous as a function default?
5. Which structure is best for an AI API request: list, tuple, or dictionary?

Send your challenge code and answers when done.
