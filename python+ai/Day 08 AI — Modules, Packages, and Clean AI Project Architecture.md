# Day 8 AI — Modules, Packages, and Clean AI Project Architecture

Today you learn how to organize AI code so it stays understandable when it grows from one file into an LLM, RAG, or agent application.

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

All code and remaining commands are the same on both systems.

---

## Core concept

A single-file project becomes difficult to manage:

```text
main.py
├── prompts
├── validation
├── model configuration
├── agent memory
├── API calls
├── file handling
└── logging
```

A professional project separates responsibilities:

```text
ai_core/
├── config.py          → model configuration and validation
├── prompt_builder.py  → prompt-related logic
├── request_service.py → combines modules into an LLM request
├── agent_state.py     → agent memory and state
└── conversation_store.py → JSON persistence
```

Rule:

```text
One module should have one clear responsibility.
```

Avoid putting everything in `main.py`.

---

## Important import rule

Use absolute imports inside your project:

```python
from ai_core.config import ModelConfig
from ai_core.prompt_builder import build_tutor_instruction
```

Avoid circular imports:

```text
config.py imports request_service.py
request_service.py imports config.py
```

That creates confusion and can break the application.

Dependency direction should be:

```text
config.py + prompt_builder.py
            ↓
     request_service.py
            ↓
          main.py
```

Lower-level modules must not import higher-level modules.

---

# Day 8 Program — Modular AI Learning Request Builder

Create or confirm this structure:

```text
python-ai-mastery/
├── ai_core/
│   ├── __init__.py
│   ├── config.py
│   ├── prompt_builder.py
│   └── request_service.py
├── tests/
└── main.py
```

## 1. `ai_core/config.py`

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ModelConfig:
    model: str = "demo-llm"
    temperature: float = 0.5
    max_tokens: int = 500

    def __post_init__(self) -> None:
        if not self.model.strip():
            raise ValueError("Model name cannot be empty.")

        if not 0 <= self.temperature <= 2:
            raise ValueError("Temperature must be between 0 and 2.")

        if not 1 <= self.max_tokens <= 4_000:
            raise ValueError("max_tokens must be between 1 and 4000.")
```

`frozen=True` means the configuration cannot accidentally change after creation.

## 2. `ai_core/prompt_builder.py`

```python
def build_tutor_instruction(topic: str, level: str) -> str:
    cleaned_topic = topic.strip()
    cleaned_level = level.strip()

    if not cleaned_topic:
        raise ValueError("Topic cannot be empty.")

    if not cleaned_level:
        raise ValueError("Level cannot be empty.")

    return (
        "You are a helpful Python and AI tutor. "
        f"Explain {cleaned_topic} for a {cleaned_level} learner. "
        "Use plain language and one practical example."
    )
```

## 3. `ai_core/request_service.py`

```python
from typing import Any

from ai_core.config import ModelConfig
from ai_core.prompt_builder import build_tutor_instruction


def create_learning_request(
    topic: str,
    level: str,
    config: ModelConfig,
) -> dict[str, Any]:
    instruction = build_tutor_instruction(topic, level)

    return {
        "model": config.model,
        "temperature": config.temperature,
        "max_tokens": config.max_tokens,
        "messages": [
            {
                "role": "system",
                "content": instruction,
            },
            {
                "role": "user",
                "content": f"Teach me about {topic.strip()}.",
            },
        ],
    }
```

## 4. `main.py`

```python
import json

from ai_core.config import ModelConfig
from ai_core.request_service import create_learning_request


def main() -> None:
    config = ModelConfig(
        model="demo-llm",
        temperature=0.4,
        max_tokens=350,
    )

    request = create_learning_request(
        topic="Retrieval-Augmented Generation",
        level="beginner",
        config=config,
    )

    print(json.dumps(request, indent=2))


if __name__ == "__main__":
    main()
```

Run:

```bash
python main.py
ruff check .
```

---

# Assignment — AI Request Builder v1

## Requirement

A learning-platform client wants a service that generates well-structured AI learning requests.

### Scope

Input:

```text
topic
level
model configuration
```

Output:

```text
Validated AI request dictionary
```

### Acceptance criteria

Your project must:

- Reject an empty topic.
- Reject an empty level.
- Reject invalid `temperature` values.
- Reject invalid `max_tokens` values.
- Return a dictionary containing `model`, `temperature`, `max_tokens`, and `messages`.
- Keep configuration, prompt logic, and request-building logic in separate modules.
- Pass:

```bash
pytest -q
ruff check .
```

## Your task

Create `tests/test_request_service.py` and write at least these tests:

```text
1. A valid request has the expected model configuration.
2. A valid request has two messages.
3. An empty topic raises ValueError.
4. An invalid temperature raises ValueError.
```

---

# How to handle this at different levels

## Institute level

Submit:

```text
- Source code
- Test screenshot/output
- README.md
- Architecture diagram
- Explanation of every module
```

For viva, be ready to explain:

```text
Why did you separate config.py and request_service.py?
What is a package?
Why avoid circular imports?
Why use tests?
```

## Corporate level

Treat it like a small ticket:

```text
Ticket: Build validated learning-request builder
Branch: feature/learning-request-builder
Definition of done:
- Requirements implemented
- Tests passing
- Linter passing
- No secrets in code
- README updated
- Code review ready
```

A reviewer will check:

- Is each module focused?
- Are errors understandable?
- Are tests meaningful?
- Is the code easy to extend for a real LLM API later?

## Client level

Do not begin coding immediately. Clarify:

```text
Which learners: beginner, intermediate, or advanced?
Which topics are allowed?
What response format is needed?
Should users choose model settings?
What is the maximum answer size?
Must user inputs and chats remain private?
```

Then show a small demo, collect feedback, and confirm the client accepts the output before expanding scope.

---

## Real-world approach to remember

```text
Understand requirement
→ define acceptance criteria
→ design small modules
→ build and test
→ explain/demo to stakeholder
→ improve using feedback
```

Day 9 AI will cover **project documentation, README files, Git workflow, and how to present an AI project professionally.**
