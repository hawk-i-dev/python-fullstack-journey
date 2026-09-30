# Day 8 AI — Extra Programs + Quiz

**Python code and commands are the same on Mac and Windows** after activating `.venv`.

Run from the project root:

```bash
python main.py
pytest -q
ruff check .
```

## Program 1 — Reusable topic utilities module

Create `ai_core/topic_tools.py`:

```python
def normalize_topic(topic: str) -> str:
    cleaned_topic = " ".join(topic.split())

    if not cleaned_topic:
        raise ValueError("Topic cannot be empty.")

    return cleaned_topic


def create_topic_slug(topic: str) -> str:
    normalized_topic = normalize_topic(topic)

    return normalized_topic.lower().replace(" ", "-")
```

Create `ai_core/day_08_program_01.py`:

```python
from ai_core.topic_tools import create_topic_slug, normalize_topic


topic = "  Retrieval   Augmented   Generation  "

print(normalize_topic(topic))
print(create_topic_slug(topic))
```

Run:

```bash
python -m ai_core.day_08_program_01
```

Expected slug:

```text
retrieval-augmented-generation
```

## Program 2 — Request summary module

Create `ai_core/request_summary.py`:

```python
from typing import Any


def summarize_request(request: dict[str, Any]) -> str:
    messages = request.get("messages", [])
    model = request.get("model", "unknown")
    temperature = request.get("temperature", "unknown")
    max_tokens = request.get("max_tokens", "unknown")

    return (
        f"Model: {model}\n"
        f"Temperature: {temperature}\n"
        f"Max tokens: {max_tokens}\n"
        f"Message count: {len(messages)}"
    )
```

Add this to `main.py`:

```python
from ai_core.request_summary import summarize_request
```

Then replace the final `print(...)` with:

```python
print(summarize_request(request))
```

This demonstrates that `main.py` coordinates modules but does not contain all business logic.

## Program 3 — Test a module independently

Create `tests/test_topic_tools.py`:

```python
import pytest

from ai_core.topic_tools import create_topic_slug, normalize_topic


def test_normalize_topic_removes_extra_whitespace() -> None:
    assert normalize_topic("  AI   agents  ") == "AI agents"


def test_create_topic_slug() -> None:
    assert create_topic_slug("Vector Database") == "vector-database"


def test_normalize_topic_rejects_empty_value() -> None:
    with pytest.raises(ValueError, match="Topic cannot be empty"):
        normalize_topic("   ")
```

Run:

```bash
pytest -q
```

## Program 4 — Keep executable code out of imports

Create `ai_core/day_08_program_04.py`:

```python
def calculate_message_count(messages: list[dict[str, str]]) -> int:
    return len(messages)


def main() -> None:
    messages = [
        {"role": "system", "content": "You are helpful."},
        {"role": "user", "content": "Explain RAG."},
    ]

    print(f"Message count: {calculate_message_count(messages)}")


if __name__ == "__main__":
    main()
```

The `if __name__ == "__main__":` block means `main()` runs only when this file is executed directly—not when another module imports it.

# Day 8 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. What is a Python module?

   A. A database  
   B. A Python file containing reusable code  
   C. A Git branch  
   D. A test result  

2. What is a Python package?

   A. A folder that groups related Python modules  
   B. Only a single function  
   C. A JSON file  
   D. A virtual environment  

3. What is the main purpose of `__init__.py` in this course structure?

   A. Start every program automatically  
   B. Store API keys  
   C. Mark `ai_core` as a Python package  
   D. Replace `main.py`  

4. Which import is an absolute project import?

   A. `from ai_core.config import ModelConfig`  
   B. `import ../config`  
   C. `from main import everything`  
   D. `include config.py`  

5. Which file should contain model configuration validation?

   A. `main.py` only  
   B. `config.py`  
   C. `README.md`  
   D. `.gitignore`  

6. Which file should create prompts?

   A. `prompt_builder.py`  
   B. `agent_memory.json`  
   C. `requirements.txt`  
   D. `__init__.py` only  

7. What is a circular import?

   A. Importing a module once  
   B. Two or more modules importing each other in a loop  
   C. Running `python main.py`  
   D. Importing a standard-library module  

8. Why are circular imports harmful?

   A. They make folders smaller  
   B. They can cause confusing errors and tightly coupled code  
   C. They improve test coverage  
   D. They encrypt the code  

9. What does this block do?

```python
if __name__ == "__main__":
    main()
```

   A. Runs `main()` only when the file is executed directly  
   B. Runs `main()` every time the module is imported  
   C. Deletes the module after running  
   D. Runs only during pytest  

10. Which is the best module design?

   A. One 2,000-line `main.py` file  
   B. Small modules with one clear responsibility  
   C. Every module imports every other module  
   D. Store code inside JSON files  

11. Why test modules independently?

   A. To verify one responsibility without depending on the whole application  
   B. To avoid using functions  
   C. To make imports fail  
   D. To remove project structure  

12. What is an acceptance criterion?

   A. A clear condition that proves a requested feature works  
   B. A Python error  
   C. A type of model  
   D. A terminal command  

13. Which is a valid acceptance criterion for the Day 8 assignment?

   A. “Make it good”  
   B. “The project should feel advanced”  
   C. “An empty topic raises ValueError”  
   D. “Use as many files as possible”  

14. What should a corporate code review check?

   A. Only file colors in VS Code  
   B. Code quality, tests, security, and maintainability  
   C. Only the number of lines  
   D. The developer’s personal password  

15. Before building an AI feature for a client, what should you do first?

   A. Add random libraries  
   B. Clarify requirements, scope, privacy, and success criteria  
   C. Deploy immediately  
   D. Put API keys into source code
