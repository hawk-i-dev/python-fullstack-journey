# Day 2 AI — Decisions, Loops, and AI Message Processing

Today you learn how Python makes decisions and processes collections. This is essential for validating AI inputs, managing chat history, filtering documents for RAG, and agent workflows.

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

Python code below is the same on both systems.

---

## 1. Boolean logic and comparisons

```python
temperature = 0.7

print(temperature > 0)
print(temperature <= 2)
print(temperature == 0.7)
print(temperature != 1.0)
```

Use `==` to compare values:

```python
role = "user"

if role == "user":
    print("This is a user message.")
```

Use `is` only when checking identity, usually against `None`:

```python
api_key = None

if api_key is None:
    print("API key is missing.")
```

Do not write this:

```python
if role is "user":  # Wrong
    ...
```

---

## 2. Truthy and falsy values

Python treats these as `False` in an `if` condition:

```python
None
False
0
0.0
""
[]
{}
set()
```

Example:

```python
prompt = ""

if not prompt:
    print("Prompt is required.")
```

This is clean validation code used in APIs and AI applications.

---

## 3. `if`, `elif`, and `else`

```python
def describe_temperature(temperature: float) -> str:
    if temperature < 0:
        return "Invalid: temperature cannot be negative."
    elif temperature <= 0.3:
        return "Focused and predictable output."
    elif temperature <= 1:
        return "Balanced output."
    elif temperature <= 2:
        return "More creative output."
    else:
        return "Invalid: temperature must be at most 2."
```

A condition must be checked from the most specific/range-limited rule to the broadest rule.

---

## 4. Loops

### Loop through a list

```python
models = ["gpt-5", "gpt-4.1", "local-model"]

for model in models:
    print(model)
```

### Use `enumerate()` when you need position and value

```python
messages = ["Hello", "What is RAG?", "Give an example."]

for index, message in enumerate(messages, start=1):
    print(f"{index}. {message}")
```

### Loop through a dictionary

```python
request = {
    "model": "gpt-5",
    "temperature": 0.7,
    "max_tokens": 500,
}

for key, value in request.items():
    print(f"{key}: {value}")
```

---

## 5. `break`, `continue`, and `return`

```python
messages = ["Hello", "", "Explain embeddings", "STOP", "More text"]

for message in messages:
    if not message:
        continue  # Skip empty messages.

    if message == "STOP":
        break  # Stop the loop completely.

    print(message)
```

Inside a function, `return` immediately ends the function and optionally gives back a result.

---

## 6. List comprehensions

A list comprehension creates a new list from existing data.

```python
messages = [" hello ", "", " What is RAG? ", "   "]

cleaned_messages = [
    message.strip()
    for message in messages
    if message.strip()
]

print(cleaned_messages)
```

Output:

```python
['hello', 'What is RAG?']
```

Read it as:

> For every `message` in `messages`, keep its stripped version only if it is not empty.

Use comprehensions for simple transformations. Use a normal `for` loop when the logic is complex.

---

# Day 2 AI Program — Conversation Validator

Create `lessons/day_02_conversation_validator.py`:

```python
VALID_ROLES = {"system", "user", "assistant"}


def validate_message(message: dict[str, str]) -> str | None:
    role = message.get("role", "")
    content = message.get("content", "").strip()

    if role not in VALID_ROLES:
        return f"Invalid role: {role!r}"

    if not content:
        return "Message content cannot be empty."

    if len(content) > 500:
        return "Message content is too long for this demo."

    return None


def clean_conversation(
    conversation: list[dict[str, str]],
) -> list[dict[str, str]]:
    valid_messages: list[dict[str, str]] = []

    for message in conversation:
        error = validate_message(message)

        if error:
            print(f"Skipped message: {error}")
            continue

        valid_messages.append(
            {
                "role": message["role"],
                "content": message["content"].strip(),
            }
        )

    return valid_messages


conversation = [
    {"role": "system", "content": " You are a helpful Python tutor. "},
    {"role": "user", "content": " What is an AI agent? "},
    {"role": "unknown", "content": "This should be rejected."},
    {"role": "assistant", "content": "   "},
    {"role": "assistant", "content": "An AI agent can reason and use tools."},
]

cleaned_conversation = clean_conversation(conversation)

print("\nValid conversation:")
for index, message in enumerate(cleaned_conversation, start=1):
    print(f"{index}. {message['role'].title()}: {message['content']}")
```

Run:

```bash
python lessons/day_02_conversation_validator.py
ruff check .
```

Expected result: two invalid messages are skipped, and three clean messages are printed.

## Day 2 challenge

Add a function:

```python
def count_messages_by_role(
    conversation: list[dict[str, str]],
) -> dict[str, int]:
```

For the valid conversation, it should return:

```python
{"system": 1, "user": 1, "assistant": 1}
```

## Remember

```text
if / elif / else  → decisions
for                → process each item
continue           → skip current item
break              → stop loop
dictionary         → structured AI/API data
list comprehension → concise transformations
```

When you complete the program and challenge, send your code or output.
