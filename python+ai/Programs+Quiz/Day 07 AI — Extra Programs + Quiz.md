# Day 7 AI — Extra Programs + Quiz

**Python code is the same on Mac and Windows.**

For files inside `ai_core`, run them from the project root with:

```bash
python -m ai_core.file_name
ruff check .
```

## Program 1 — Save and read a text prompt

Create `ai_core/day_07_program_01.py`:

```python
from pathlib import Path


prompt_file = Path("data") / "prompts" / "rag_prompt.txt"

prompt_file.parent.mkdir(parents=True, exist_ok=True)

prompt = "Explain RAG with one practical Python example."

prompt_file.write_text(prompt, encoding="utf-8")

saved_prompt = prompt_file.read_text(encoding="utf-8")

print(f"Saved prompt: {saved_prompt}")
print(f"File location: {prompt_file}")
```

Run:

```bash
python -m ai_core.day_07_program_01
```

## Program 2 — Save an AI configuration as JSON

Create `ai_core/day_07_program_02.py`:

```python
import json
from pathlib import Path


config_file = Path("data") / "ai_config.json"

config_file.parent.mkdir(parents=True, exist_ok=True)

config = {
    "model": "gpt-5",
    "temperature": 0.5,
    "max_tokens": 300,
    "tools_enabled": ["knowledge_search", "calculator"],
}

with config_file.open("w", encoding="utf-8") as file:
    json.dump(config, file, indent=2)

with config_file.open(encoding="utf-8") as file:
    loaded_config = json.load(file)

print("Loaded configuration:")
for key, value in loaded_config.items():
    print(f"{key}: {value}")
```

## Program 3 — Handle a missing JSON file

Create `ai_core/day_07_program_03.py`:

```python
import json
from pathlib import Path


def load_json_file(file_path: Path) -> dict[str, object] | None:
    try:
        with file_path.open(encoding="utf-8") as file:
            return json.load(file)
    except FileNotFoundError:
        print(f"File not found: {file_path}")
        return None
    except json.JSONDecodeError:
        print(f"Invalid JSON: {file_path}")
        return None


missing_file = Path("data") / "does_not_exist.json"

data = load_json_file(missing_file)

if data is None:
    print("No configuration was loaded.")
else:
    print(data)
```

## Program 4 — Save only recent conversation messages

Create `ai_core/day_07_program_04.py`:

```python
import json
from pathlib import Path


def save_recent_messages(
    messages: list[dict[str, str]],
    file_path: Path,
    limit: int,
) -> None:
    if limit <= 0:
        raise ValueError("limit must be greater than zero.")

    file_path.parent.mkdir(parents=True, exist_ok=True)

    recent_messages = messages[-limit:]

    with file_path.open("w", encoding="utf-8") as file:
        json.dump(recent_messages, file, indent=2)


conversation = [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "What is Python?"},
    {"role": "assistant", "content": "Python is a programming language."},
    {"role": "user", "content": "What is RAG?"},
    {"role": "assistant", "content": "RAG retrieves relevant knowledge first."},
]

memory_file = Path("data") / "recent_messages.json"

save_recent_messages(conversation, memory_file, limit=3)

print(memory_file.read_text(encoding="utf-8"))
```

# Day 7 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. Why use `pathlib.Path`?

   A. It works only on Windows  
   B. It creates portable file paths across operating systems  
   C. It replaces JSON  
   D. It installs Python packages  

2. Which statement correctly creates a portable path?

   A. `path = "C:\\Users\\name\\file.json"`  
   B. `path = Path("data") / "file.json"`  
   C. `path = "/Users/name/file.json"`  
   D. `path = "data" + "\\file.json"`  

3. Which Python types are directly JSON-compatible?

   A. `datetime` and custom class instances  
   B. `str`, `int`, `float`, `bool`, `list`, `dict`, and `None`  
   C. Every Python object  
   D. Only strings  

4. What does `json.dump(data, file)` do?

   A. Reads JSON from a file  
   B. Writes Python data as JSON into a file  
   C. Deletes a JSON file  
   D. Converts JSON into a class automatically  

5. What does `json.load(file)` do?

   A. Reads JSON from a file into Python data  
   B. Writes Python data into a file  
   C. Creates a folder  
   D. Activates `.venv`  

6. Why use `encoding="utf-8"` when reading/writing text?

   A. To make files executable  
   B. To handle text consistently, including non-English characters  
   C. To encrypt the file  
   D. To turn JSON into a database  

7. What does this do?

```python
file_path.parent.mkdir(parents=True, exist_ok=True)
```

   A. Deletes the parent folder  
   B. Creates parent folders when needed without failing if they exist  
   C. Reads all files in a folder  
   D. Renames the file  

8. Why convert a `datetime` using `.isoformat()` before JSON saving?

   A. JSON cannot directly store Python datetime objects  
   B. It makes the date private  
   C. It removes the timestamp  
   D. It converts it into an integer only  

9. Which error occurs when a JSON file has invalid syntax?

   A. `KeyError`  
   B. `FileNotFoundError`  
   C. `json.JSONDecodeError`  
   D. `TypeError`  

10. Which error occurs when a file does not exist?

   A. `ValueError`  
   B. `FileNotFoundError`  
   C. `JSONDecodeError`  
   D. `ImportError`  

11. What does `messages[-3:]` return?

   A. The first three messages  
   B. Every message except the final three  
   C. The final three messages  
   D. An error  

12. Why might an AI agent save only recent messages?

   A. To reduce conversation size, cost, and token usage  
   B. To make JSON invalid  
   C. To remove all agent memory permanently  
   D. To avoid using functions  

13. Which should never be committed to Git or stored in shared JSON files?

   A. A public model name  
   B. API keys and passwords  
   C. A Python file name  
   D. A test name  

14. Which mode opens a file for writing?

   A. `"r"`  
   B. `"w"`  
   C. `"a"` only  
   D. `"json"`  

15. Why run this command?

```bash
python -m ai_core.conversation_store
```

   A. It runs a module as part of the `ai_core` package  
   B. It deletes the module  
   C. It installs `ai_core` from the internet  
   D. It only checks formatting
