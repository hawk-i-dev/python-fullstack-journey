# Day 7 AI — Files, JSON, and Persistent Agent Memory

Today your AI agent will save conversation memory to a JSON file and load it again later.

Without persistence, agent memory disappears whenever the Python program stops.

```text
Program starts → agent has memory in RAM
Program ends   → memory is lost

JSON file → memory can be saved and restored
```

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

The Python code and run command are the same on both systems.

---

## 1. Use `pathlib`, not hard-coded paths

Avoid this:

```python
file_path = "C:\\Users\\name\\Documents\\memory.json"
```

Use this:

```python
from pathlib import Path

memory_file = Path("data") / "agent_memory.json"
```

`Path` works correctly on Windows, Mac, and Linux.

---

## 2. Why JSON matters for AI

JSON is structured text, commonly used for:

- LLM API requests and responses
- Agent memory
- Configuration
- RAG metadata
- Saving application state

Example:

```json
{
  "role": "user",
  "content": "What is RAG?"
}
```

Python dictionary equivalent:

```python
{
    "role": "user",
    "content": "What is RAG?",
}
```

---

## 3. Important: Python objects are not always JSON-ready

These work directly in JSON:

```python
str
int
float
bool
list
dict
None
```

This does not work directly:

```python
datetime
```

So we convert a datetime to text with:

```python
created_at.isoformat()
```

---

# Day 7 AI Program — Save and Load Agent Memory

Create `ai_core/conversation_store.py`:

```python
import json
from datetime import datetime
from pathlib import Path
from typing import Any

from ai_core.agent_state import AgentState, ChatMessage


VALID_ROLES = {"system", "user", "assistant"}


class ConversationStoreError(ValueError):
    """Raised when conversation memory cannot be saved or loaded."""


def save_agent_state(agent: AgentState, file_path: Path) -> None:
    file_path.parent.mkdir(parents=True, exist_ok=True)

    data = {
        "name": agent.name,
        "tools_used": agent.tools_used,
        "messages": [
            {
                "role": message.role,
                "content": message.content,
                "created_at": message.created_at.isoformat(),
            }
            for message in agent.messages
        ],
    }

    try:
        with file_path.open("w", encoding="utf-8") as file:
            json.dump(data, file, indent=2)
    except OSError as error:
        raise ConversationStoreError(
            f"Could not save conversation to {file_path}."
        ) from error


def load_agent_state(file_path: Path) -> AgentState:
    try:
        with file_path.open(encoding="utf-8") as file:
            data: dict[str, Any] = json.load(file)
    except FileNotFoundError as error:
        raise ConversationStoreError(
            f"Conversation file does not exist: {file_path}."
        ) from error
    except json.JSONDecodeError as error:
        raise ConversationStoreError(
            f"Conversation file contains invalid JSON: {file_path}."
        ) from error
    except OSError as error:
        raise ConversationStoreError(
            f"Could not read conversation file: {file_path}."
        ) from error

    name = data.get("name")

    if not isinstance(name, str) or not name.strip():
        raise ConversationStoreError("Conversation must contain a valid agent name.")

    agent = AgentState(name=name)

    tools_used = data.get("tools_used", [])
    messages = data.get("messages", [])

    if not isinstance(tools_used, list) or not isinstance(messages, list):
        raise ConversationStoreError("Conversation has an invalid structure.")

    for tool_name in tools_used:
        if not isinstance(tool_name, str):
            raise ConversationStoreError("Tool names must be strings.")

        agent.record_tool_use(tool_name)

    for message_data in messages:
        if not isinstance(message_data, dict):
            raise ConversationStoreError("Every message must be an object.")

        role = message_data.get("role")
        content = message_data.get("content")
        created_at = message_data.get("created_at")

        if role not in VALID_ROLES:
            raise ConversationStoreError(f"Invalid message role: {role!r}.")

        if not isinstance(content, str):
            raise ConversationStoreError("Message content must be a string.")

        if not isinstance(created_at, str):
            raise ConversationStoreError("Message timestamp must be a string.")

        try:
            timestamp = datetime.fromisoformat(created_at)
        except ValueError as error:
            raise ConversationStoreError(
                f"Invalid timestamp: {created_at!r}."
            ) from error

        agent.messages.append(
            ChatMessage(
                role=role,
                content=content,
                created_at=timestamp,
            )
        )

    return agent


if __name__ == "__main__":
    memory_file = Path("data") / "agent_memory.json"

    agent = AgentState(name="Python AI Tutor")
    agent.add_message("system", "You are a helpful Python and AI tutor.")
    agent.add_message("user", "Explain how RAG uses document chunks.")
    agent.record_tool_use("knowledge_search")

    save_agent_state(agent, memory_file)
    print(f"Saved agent memory to: {memory_file}")

    restored_agent = load_agent_state(memory_file)

    print("\nRestored agent state:")
    print(restored_agent.summary())
```

Run it from the project root:

```bash
python -m ai_core.conversation_store
ruff check .
```

It creates:

```text
data/
└── agent_memory.json
```

Open the JSON file in VS Code and inspect it.

## Important safety rule

Never save these in JSON files committed to Git:

```text
API keys
Passwords
Access tokens
Private customer data
```

Add this to `.gitignore` if you later store real/private agent memory:

```gitignore
data/private/
```

## Day 7 challenge

Add a function:

```python
def save_recent_messages(
    agent: AgentState,
    file_path: Path,
    limit: int,
) -> None:
```

It should save only the most recent `limit` messages.

Example:

```text
Agent has 10 messages
limit = 3
JSON file saves only the final 3 messages
```

## Key takeaway

```text
Pathlib       → portable file paths
JSON          → structured, persistent text data
json.dump()   → Python data → JSON file
json.load()   → JSON file → Python data
isoformat()   → datetime → JSON-safe text
```

Day 8 AI will cover **Python modules, packages, imports, and clean project architecture**—how to keep an AI/RAG project organized as it grows.
