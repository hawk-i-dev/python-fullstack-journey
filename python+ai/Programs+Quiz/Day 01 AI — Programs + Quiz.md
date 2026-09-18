# Day 1 AI — Programs + Quiz

**Code is the same on Mac and Windows.**

Run files after activating `.venv`:

```bash
python lessons/file_name.py
```

## Programs

### Program 1 — Variable, type, and object identity

Create `lessons/day_01_program_01.py`:

```python
model_name = "gpt-5"
temperature = 0.7
max_tokens = 500
is_available = True

for value in [model_name, temperature, max_tokens, is_available]:
    print(f"Value: {value}")
    print(f"Type: {type(value).__name__}")
    print(f"Object ID: {id(value)}")
    print("-" * 30)
```

Understand: every value is an object; a variable refers to it.

### Program 2 — Immutable string vs mutable list

Create `lessons/day_01_program_02.py`:

```python
prompt = "Explain Python"
print("Before:", prompt, id(prompt))

prompt += " with examples"
print("After: ", prompt, id(prompt))

history = ["Hello"]
print("\nBefore:", history, id(history))

history.append("Explain Python with examples")
print("After: ", history, id(history))
```

Observe: string ID changes; list ID normally stays the same.

### Program 3 — Reference bug and copy fix

Create `lessons/day_01_program_03.py`:

```python
original_history = ["Hello", "What is an LLM?"]

shared_history = original_history
shared_history.append("What is RAG?")

print("Original after shared reference:")
print(original_history)

safe_history = original_history.copy()
safe_history.append("What is a vector database?")

print("\nOriginal after copy:")
print(original_history)

print("\nCopied history:")
print(safe_history)
```

Understand: `shared_history = original_history` does not create a new list.

### Program 4 — Build an AI request dictionary

Create `lessons/day_01_program_04.py`:

```python
ai_request = {
    "model": "gpt-5",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful Python tutor.",
        },
        {
            "role": "user",
            "content": "Explain Python dictionaries.",
        },
    ],
    "temperature": 0.5,
    "max_tokens": 300,
}

print("Model:", ai_request["model"])
print("User question:", ai_request["messages"][1]["content"])
print("Temperature:", ai_request["temperature"])
```

Understand: dictionaries represent structured data, API payloads, JSON, and AI requests.

### Program 5 — Safe conversation-history function

Create `lessons/day_01_program_05.py`:

```python
def add_conversation_message(
    role: str,
    content: str,
    history: list[dict[str, str]] | None = None,
) -> list[dict[str, str]]:
    if role not in ("user", "assistant"):
        raise ValueError("Role must be 'user' or 'assistant'.")

    cleaned_content = content.strip()

    if not cleaned_content:
        raise ValueError("Content cannot be empty.")

    if history is None:
        history = []

    history.append(
        {
            "role": role,
            "content": cleaned_content,
        }
    )

    return history


conversation = add_conversation_message("user", "What is Python?")
conversation = add_conversation_message(
    "assistant",
    "Python is a readable, general-purpose programming language.",
    conversation,
)

for message in conversation:
    print(f"{message['role'].title()}: {message['content']}")
```

This is the foundation of how a chatbot stores its conversation messages.

Run all checks:

```bash
ruff check .
```

# Day 1 AI MCQ Quiz

Reply in this format: `1.B 2.A 3.C ...`

1. A Python variable is best described as:

   A. A fixed memory box  
   B. A label/reference to an object  
   C. Always a string  
   D. A database column  

2. Which is mutable?

   A. `str`  
   B. `tuple`  
   C. `list`  
   D. `int`  

3. What happens here?

```python
a = [1, 2]
b = a
b.append(3)
```

   A. Only `b` becomes `[1, 2, 3]`  
   B. Both `a` and `b` become `[1, 2, 3]`  
   C. Python raises an error  
   D. `a` becomes empty  

4. Which creates a shallow copy of a list?

   A. `second = first`  
   B. `second = first.copy()`  
   C. `second = id(first)`  
   D. `second = type(first)`  

5. Which is immutable?

   A. Dictionary  
   B. Set  
   C. List  
   D. Tuple  

6. What does `type(value)` return?

   A. Object location  
   B. Object’s data type  
   C. Object’s password  
   D. Number of values  

7. Why can this be risky?

```python
def add_item(item, items=[]):
```

   A. Functions cannot return lists  
   B. The same list may be reused across function calls  
   C. Lists cannot contain items  
   D. Python does not support defaults  

8. What is the safe default pattern?

   A. `items = []` inside the parameter  
   B. `items: list = {}`  
   C. `items=None`, then create `[]` inside the function  
   D. Remove the function  

9. Which structure best represents an AI API request with named fields such as `model` and `temperature`?

   A. Dictionary  
   B. Tuple  
   C. String  
   D. Integer  

10. What does `.strip()` do to a string?

   A. Deletes every space  
   B. Removes leading and trailing whitespace  
   C. Converts it to a list  
   D. Makes it immutable  

11. What is the usual purpose of a function?

   A. Store a database permanently  
   B. Create reusable behavior for one task  
   C. Replace all variables  
   D. Install packages  

12. In this signature, what does `-> str` communicate?

```python
def build_prompt(topic: str) -> str:
```

   A. The function receives only numbers  
   B. The function is expected to return a string  
   C. The function has no output  
   D. Python strictly converts every value automatically  

13. Which is a valid chat-message dictionary?

   A. `["user", "Hello"]`  
   B. `{"role": "user", "content": "Hello"}`  
   C. `("role", "user")`  
   D. `"user: Hello"`  

14. Why use type hints?

   A. They make Python compiled  
   B. They help tools and developers understand expected data  
   C. They replace tests  
   D. They prevent every runtime error  

15. When you run `id(my_list)` before and after `.append()`, the ID normally:

   A. Remains the same  
   B. Always becomes zero  
   C. Changes because a new list is created  
   D. Causes an error
