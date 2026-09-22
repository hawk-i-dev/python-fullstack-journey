# Day 6 AI — Classes, Dataclasses, and AI Agent State

Today you will model the internal state of an AI agent cleanly.

An AI agent is not merely a Python class. A real agent usually combines:

```text
LLM + instructions + conversation memory + tools + control loop
```

A class helps us organize the **memory and behavior** of that system.

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

All Python code below is the same on Mac and Windows.

---

## 1. Class vs object

A class is a blueprint; an object is one created instance.

```python
class AIModel:
    pass


tutor_model = AIModel()
research_model = AIModel()
```

```text
AIModel       → blueprint
tutor_model   → one object made from the blueprint
research_model → another object
```

---

## 2. Attributes and methods

```python
class AIModel:
    def __init__(self, name: str, temperature: float) -> None:
        self.name = name
        self.temperature = temperature

    def describe(self) -> str:
        return f"{self.name} uses temperature {self.temperature}."
```

```python
model = AIModel("gpt-5", 0.7)

print(model.name)
print(model.describe())
```

- `self` means “this specific object.”
- Attributes store data.
- Methods define behavior.

---

## 3. Why `@dataclass` is useful

For classes mostly used to hold data, Python’s `@dataclass` removes repeated boilerplate.

Instead of writing `__init__` manually:

```python
from dataclasses import dataclass


@dataclass
class ChatMessage:
    role: str
    content: str
```

Now Python automatically creates an initializer:

```python
message = ChatMessage(
    role="user",
    content="What is RAG?",
)
```

Dataclasses are excellent for internal Python models. Later, we will use **Pydantic** for API request/response validation.

---

## 4. Avoid shared mutable defaults

Wrong:

```python
@dataclass
class AgentState:
    messages: list[ChatMessage] = []  # Wrong
```

Every `AgentState` could accidentally share the same list.

Correct:

```python
from dataclasses import field


@dataclass
class AgentState:
    messages: list[ChatMessage] = field(default_factory=list)
```

`default_factory=list` creates a new empty list for every new agent.

---

# Day 6 AI Program — Model an Agent’s State

Create `ai_core/agent_state.py`:

```python
from dataclasses import dataclass, field
from datetime import datetime, timezone
from typing import Literal


MessageRole = Literal["system", "user", "assistant"]


@dataclass
class ChatMessage:
    role: MessageRole
    content: str
    created_at: datetime = field(
        default_factory=lambda: datetime.now(timezone.utc)
    )

    def __post_init__(self) -> None:
        self.content = self.content.strip()

        if not self.content:
            raise ValueError("Message content cannot be empty.")


@dataclass
class AgentState:
    name: str
    messages: list[ChatMessage] = field(default_factory=list)
    tools_used: list[str] = field(default_factory=list)

    def add_message(self, role: MessageRole, content: str) -> ChatMessage:
        message = ChatMessage(role=role, content=content)
        self.messages.append(message)
        return message

    def record_tool_use(self, tool_name: str) -> None:
        cleaned_tool_name = tool_name.strip()

        if not cleaned_tool_name:
            raise ValueError("Tool name cannot be empty.")

        self.tools_used.append(cleaned_tool_name)

    def latest_user_message(self) -> ChatMessage | None:
        for message in reversed(self.messages):
            if message.role == "user":
                return message

        return None

    def summary(self) -> str:
        latest_message = self.latest_user_message()

        if latest_message is None:
            latest_question = "No user question yet."
        else:
            latest_question = latest_message.content

        return (
            f"Agent: {self.name}\n"
            f"Messages: {len(self.messages)}\n"
            f"Tools used: {self.tools_used or ['None']}\n"
            f"Latest user message: {latest_question}"
        )


agent = AgentState(name="Python AI Tutor")

agent.add_message(
    "system",
    "You are a helpful Python and AI tutor.",
)
agent.add_message(
    "user",
    "Explain the difference between RAG and fine-tuning.",
)
agent.record_tool_use("knowledge_search")

print(agent.summary())
```

Run:

```bash
python ai_core/agent_state.py
ruff check .
```

## What this models

```text
ChatMessage → one message in agent memory
AgentState  → current agent memory and tool history
add_message → stores a new message
record_tool_use → records an external action
latest_user_message → finds current user intent
summary → gives a readable state overview
```

## Day 6 challenge

Add a method inside `AgentState`:

```python
def clear_conversation(self) -> None:
```

It must remove every message but keep `name` and `tools_used`.

Then test it manually:

```python
agent.clear_conversation()
print(agent.summary())
```

## Senior-level takeaway

Use classes when **data and related behavior belong together**. Do not create a class merely because OOP exists; simple functions and dictionaries are often better for small tasks.

Day 7 AI will build on this with **files, JSON, persistence, and saving/loading an AI agent’s conversation memory.**
