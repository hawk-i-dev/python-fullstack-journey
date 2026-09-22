# Day 6 AI — Extra Programs + Quiz

**Python code and commands are the same on Mac and Windows** after activating `.venv`.

```bash
python ai_core/file_name.py
ruff check .
```

## Program 1 — Basic class and method

Create `ai_core/day_06_program_01.py`:

```python
class AIModel:
    def __init__(self, name: str, temperature: float) -> None:
        self.name = name
        self.temperature = temperature

    def describe(self) -> str:
        return (
            f"Model: {self.name}, "
            f"temperature: {self.temperature}"
        )


model = AIModel("gpt-5", 0.7)

print(model.name)
print(model.describe())
```

## Program 2 — Dataclass for an AI document

Create `ai_core/day_06_program_02.py`:

```python
from dataclasses import dataclass


@dataclass
class DocumentChunk:
    document_name: str
    chunk_number: int
    content: str

    def preview(self, length: int = 50) -> str:
        return self.content[:length]


chunk = DocumentChunk(
    document_name="rag_notes.txt",
    chunk_number=1,
    content="RAG retrieves relevant document chunks before generating an answer.",
)

print(chunk)
print(chunk.preview())
```

## Program 3 — Agent state with safe list defaults

Create `ai_core/day_06_program_03.py`:

```python
from dataclasses import dataclass, field


@dataclass
class SimpleAgent:
    name: str
    tools: list[str] = field(default_factory=list)

    def add_tool(self, tool_name: str) -> None:
        cleaned_tool_name = tool_name.strip()

        if not cleaned_tool_name:
            raise ValueError("Tool name cannot be empty.")

        self.tools.append(cleaned_tool_name)


research_agent = SimpleAgent("Research Agent")
support_agent = SimpleAgent("Support Agent")

research_agent.add_tool("web_search")

print(research_agent)
print(support_agent)
```

Observe: `support_agent.tools` remains an empty list.

## Program 4 — Find the latest user message

Create `ai_core/day_06_program_04.py`:

```python
from dataclasses import dataclass
from typing import Literal


MessageRole = Literal["system", "user", "assistant"]


@dataclass
class Message:
    role: MessageRole
    content: str


@dataclass
class Conversation:
    messages: list[Message]

    def latest_user_message(self) -> Message | None:
        for message in reversed(self.messages):
            if message.role == "user":
                return message

        return None


conversation = Conversation(
    messages=[
        Message("system", "You are helpful."),
        Message("user", "What is an AI agent?"),
        Message("assistant", "An agent can use tools."),
        Message("user", "What is RAG?"),
    ]
)

latest = conversation.latest_user_message()

if latest is None:
    print("No user message found.")
else:
    print(f"Latest question: {latest.content}")
```

# Day 6 AI MCQ Quiz

Reply like: `1.B 2.C 3.A ...`

1. What is a class?

   A. A specific object already created  
   B. A blueprint for creating objects  
   C. A Python error  
   D. A database table only  

2. What is an object?

   A. An instance created from a class  
   B. A Python keyword only  
   C. A required function argument  
   D. A file extension  

3. What does `self` normally refer to inside an instance method?

   A. The current object instance  
   B. Every object in the program  
   C. The Python interpreter  
   D. A global variable  

4. What is an attribute?

   A. A class/object’s stored data  
   B. A Python loop  
   C. A test assertion  
   D. An exception type  

5. What is a method?

   A. A function defined on a class  
   B. A separate Python file  
   C. A database query  
   D. A type of list only  

6. What does `@dataclass` help generate automatically?

   A. A web server  
   B. Common class boilerplate such as `__init__`  
   C. An LLM response  
   D. A SQL migration  

7. Why is this unsafe?

```python
@dataclass
class AgentState:
    messages: list[str] = []
```

   A. Lists cannot store strings  
   B. Every instance may share the same list  
   C. Dataclasses cannot have attributes  
   D. `messages` must be a tuple  

8. What is the safe pattern?

   A. `messages: list[str] = []`  
   B. `messages: list[str] = None`  
   C. `messages: list[str] = field(default_factory=list)`  
   D. `messages = set()`  

9. What does `__post_init__()` run after?

   A. A class is imported  
   B. A dataclass instance is initialized  
   C. Every `for` loop  
   D. A test fails  

10. Why might an AI agent need an `AgentState` object?

   A. To store conversation, tool usage, and current state  
   B. To replace an LLM  
   C. To avoid using functions  
   D. To install packages  

11. What does `reversed(messages)` allow you to do?

   A. Delete messages  
   B. Iterate from the final message backward  
   C. Convert messages to a dictionary  
   D. Sort messages alphabetically  

12. What does `Message | None` mean in a return type?

   A. It returns only a string  
   B. It returns a `Message` or `None`  
   C. It returns every message  
   D. It always raises an error  

13. Which is the best explanation of an AI agent?

   A. Only a Python class  
   B. Only an LLM API call  
   C. A system combining an LLM, instructions, memory, tools, and control flow  
   D. A database table  

14. What should `clear_conversation()` do in the Day 6 challenge?

   A. Delete the whole agent object  
   B. Remove messages while retaining the agent’s name and tools  
   C. Remove only the latest user message  
   D. Add an empty assistant message  

15. When is a class most useful?

   A. When related data and behavior belong together  
   B. For every one-line calculation  
   C. Only for database access  
   D. Never; functions are always better
