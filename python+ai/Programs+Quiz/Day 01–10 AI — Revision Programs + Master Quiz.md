# Day 1–10 AI — Revision Programs + Master Quiz

**Python code is the same on Mac and Windows** after activating `.venv`.

Run programs with:

```bash
python lessons/file_name.py
pytest -q
ruff check .
```

# Revision Programs

## Program 1 — Validate and clean AI messages

Create `lessons/revision_program_01.py`:

```python
VALID_ROLES = {"system", "user", "assistant"}


def clean_messages(
    messages: list[dict[str, str]],
) -> list[dict[str, str]]:
    cleaned_messages: list[dict[str, str]] = []

    for message in messages:
        role = message.get("role", "")
        content = message.get("content", "").strip()

        if role not in VALID_ROLES:
            continue

        if not content:
            continue

        cleaned_messages.append(
            {
                "role": role,
                "content": content,
            }
        )

    return cleaned_messages


messages = [
    {"role": "system", "content": " You are a helpful tutor. "},
    {"role": "user", "content": " What is RAG? "},
    {"role": "invalid", "content": "Ignore this."},
    {"role": "assistant", "content": "   "},
]

print(clean_messages(messages))
```

## Program 2 — Agent state with dataclass

Create `lessons/revision_program_02.py`:

```python
from dataclasses import dataclass, field


@dataclass
class LearningAgent:
    name: str
    messages: list[dict[str, str]] = field(default_factory=list)
    tools_used: list[str] = field(default_factory=list)

    def add_message(self, role: str, content: str) -> None:
        cleaned_content = content.strip()

        if not cleaned_content:
            raise ValueError("Message cannot be empty.")

        self.messages.append(
            {
                "role": role,
                "content": cleaned_content,
            }
        )

    def use_tool(self, tool_name: str) -> None:
        self.tools_used.append(tool_name)


agent = LearningAgent("AI Learning Assistant")

agent.add_message("user", "Explain embeddings.")
agent.use_tool("vector_search")

print(agent)
```

## Program 3 — Save agent memory as JSON

Create `lessons/revision_program_03.py`:

```python
import json
from pathlib import Path


memory_file = Path("data") / "revision_agent_memory.json"

memory_file.parent.mkdir(parents=True, exist_ok=True)

agent_memory = {
    "name": "AI Learning Assistant",
    "messages": [
        {"role": "user", "content": "What is RAG?"},
        {
            "role": "assistant",
            "content": "RAG retrieves relevant data before generating an answer.",
        },
    ],
}

with memory_file.open("w", encoding="utf-8") as file:
    json.dump(agent_memory, file, indent=2)

with memory_file.open(encoding="utf-8") as file:
    restored_memory = json.load(file)

print(restored_memory)
```

## Program 4 — NumPy document ranking

Create `lessons/revision_program_04.py`:

```python
import numpy as np


document_names = np.array([
    "python_basics.txt",
    "rag_basics.txt",
    "ai_agents.txt",
])

document_embeddings = np.array([
    [0.1, 0.9, 0.1],
    [0.8, 0.2, 0.7],
    [0.7, 0.1, 0.9],
])

query_embedding = np.array([0.8, 0.1, 0.8])

scores = document_embeddings @ query_embedding
ranked_indexes = np.argsort(scores)[::-1]

for index in ranked_indexes:
    print(f"{document_names[index]}: {scores[index]:.3f}")
```

## Program 5 — Test a reusable function

Create `ai_core/text_tools.py`:

```python
def normalize_text(value: str) -> str:
    normalized_value = " ".join(value.split())

    if not normalized_value:
        raise ValueError("Text cannot be empty.")

    return normalized_value
```

Create `tests/test_text_tools.py`:

```python
import pytest

from ai_core.text_tools import normalize_text


def test_normalize_text_removes_extra_spaces() -> None:
    assert normalize_text("  AI   agents  ") == "AI agents"


def test_normalize_text_rejects_empty_text() -> None:
    with pytest.raises(ValueError, match="Text cannot be empty"):
        normalize_text("   ")
```

Run:

```bash
pytest -q
ruff check .
```

---

# Day 1–10 AI Master Quiz

Reply like:

```text
1.B 2.C 3.A ...
```

1. A Python variable is best described as:

   A. A fixed box containing a value  
   B. A label/reference to an object  
   C. A database table  
   D. A required function  

2. Which is mutable?

   A. `tuple`  
   B. `str`  
   C. `list`  
   D. `int`  

3. Why can this change both variables?

```python
first = [1, 2]
second = first
second.append(3)
```

   A. Both variables reference the same list  
   B. Lists are immutable  
   C. `append()` creates a tuple  
   D. Python copied the list automatically  

4. Which is falsy in Python?

   A. `"False"`  
   B. `[0]`  
   C. `" "`  
   D. `{}`  

5. What does `continue` do inside a loop?

   A. Stops the whole loop  
   B. Skips the current iteration  
   C. Returns from a function  
   D. Deletes the current item  

6. Which is the correct way to compare a value with `None`?

   A. `value == None`  
   B. `value is None`  
   C. `value = None`  
   D. `value in None`  

7. What does `-> str` communicate here?

```python
def build_prompt(topic: str) -> str:
```

   A. The expected return type is a string  
   B. The function returns nothing  
   C. `topic` is a list  
   D. Python converts every value automatically  

8. Why are keyword-only arguments useful?

   A. They remove function validation  
   B. They prevent accidentally mixing up settings  
   C. They make every input optional  
   D. They stop functions from returning values  

9. What is wrong with this default parameter?

```python
def add_item(item: str, items: list[str] = []):
```

   A. Lists cannot be arguments  
   B. The same list can be reused between calls  
   C. Functions cannot return lists  
   D. Strings cannot be added to lists  

10. What does `raise ValueError(...)` do?

   A. Signals invalid input and stops normal function flow  
   B. Silently ignores an error  
   C. Only prints a message  
   D. Restarts Python  

11. Which error commonly occurs with `int("hello")`?

   A. `KeyError`  
   B. `ValueError`  
   C. `SyntaxError`  
   D. `ImportError`  

12. Why is this dangerous?

```python
except Exception:
    pass
```

   A. It makes logging too detailed  
   B. It hides the actual error  
   C. It creates a virtual environment  
   D. It converts errors to strings  

13. What does `assert` do in pytest?

   A. Checks an expected condition  
   B. Installs a package  
   C. Starts a server  
   D. Creates Git commits  

14. What does `pytest.raises(ValueError)` test?

   A. A function returns `ValueError`  
   B. A function raises `ValueError`  
   C. Python installs `ValueError`  
   D. The test is skipped  

15. Why use `@pytest.mark.parametrize`?

   A. To run one test against multiple inputs  
   B. To create a database  
   C. To remove test cases  
   D. To make values immutable  

16. What is a class?

   A. A specific object instance  
   B. A blueprint for objects  
   C. A test file  
   D. A Git branch  

17. Why use `field(default_factory=list)` in a dataclass?

   A. To make lists immutable  
   B. To create a separate list for each instance  
   C. To delete a list after use  
   D. To convert a list to JSON  

18. What does `Message | None` mean?

   A. It returns a message or `None`  
   B. It returns only a string  
   C. It always raises an error  
   D. It returns every message  

19. Why use `Path("data") / "memory.json"`?

   A. It encrypts the file  
   B. It creates a portable path for Mac, Windows, and Linux  
   C. It automatically uploads the file  
   D. It replaces JSON  

20. What does `json.dump(data, file)` do?

   A. Reads JSON from a file  
   B. Writes Python data as JSON to a file  
   C. Deletes the JSON file  
   D. Creates a Git commit  

21. Why use `.isoformat()` on a datetime before saving JSON?

   A. JSON cannot directly store Python datetime objects  
   B. It turns the date into a password  
   C. It deletes timezone information  
   D. It creates a NumPy array  

22. What is a Python module?

   A. A Python file containing reusable code  
   B. A database backup  
   C. A terminal command  
   D. A virtual environment  

23. What is a circular import?

   A. A module importing itself once  
   B. Modules importing each other in a loop  
   C. A Git branch with a circle icon  
   D. A JSON formatting style  

24. What does this do?

```python
if __name__ == "__main__":
    main()
```

   A. Runs `main()` only when the file is executed directly  
   B. Runs `main()` every time the module is imported  
   C. Creates a class  
   D. Starts pytest  

25. What should never be committed to Git?

   A. `README.md`  
   B. Tests  
   C. API keys and passwords  
   D. `.py` files  

26. What is an acceptance criterion?

   A. A testable condition proving a requirement is complete  
   B. A Git command  
   C. An AI model name  
   D. A file extension  

27. What is NumPy mainly used for?

   A. Fast numerical computing with arrays  
   B. Git management  
   C. CSS styling  
   D. Writing Markdown only  

28. What does this shape mean?

```python
(4, 3)
```

   A. Four rows and three columns  
   B. Three rows and four columns  
   C. A vector with seven values  
   D. Four separate files  

29. What does `@` represent between compatible NumPy vectors?

   A. Dot product / matrix multiplication  
   B. Element-wise multiplication  
   C. List append  
   D. JSON conversion  

30. Why are embedding vectors important for RAG?

   A. They represent meaning numerically for similarity search  
   B. They guarantee every answer is correct  
   C. They replace client requirements  
   D. They remove the need for documents
