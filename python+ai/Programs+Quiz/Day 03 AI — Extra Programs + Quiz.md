# Day 3 AI — Extra Programs + Quiz

**Python code is the same on Mac and Windows.**

Run each file:

```bash
python lessons/file_name.py
ruff check .
```

## Program 1 — Keyword-only model configuration

Create `lessons/day_03_program_01.py`:

```python
def create_model_config(
    model: str,
    *,
    temperature: float = 0.7,
    max_tokens: int = 500,
) -> dict[str, object]:
    if not 0 <= temperature <= 2:
        raise ValueError("Temperature must be between 0 and 2.")

    if max_tokens <= 0:
        raise ValueError("max_tokens must be greater than 0.")

    return {
        "model": model,
        "temperature": temperature,
        "max_tokens": max_tokens,
    }


config = create_model_config(
    "gpt-5",
    temperature=0.3,
    max_tokens=250,
)

print(config)
```

## Program 2 — Estimate conversation size

Create `lessons/day_03_program_02.py`:

```python
from typing import Any


def estimate_request_size(request: dict[str, Any]) -> int:
    total_characters = 0

    for message in request.get("messages", []):
        content = message.get("content", "")
        total_characters += len(content)

    return total_characters


request = {
    "model": "gpt-5",
    "messages": [
        {"role": "system", "content": "You are a helpful tutor."},
        {"role": "user", "content": "Explain RAG simply."},
    ],
}

print(f"Total characters: {estimate_request_size(request)}")
```

## Program 3 — Reusable prompt builder

Create `lessons/day_03_program_03.py`:

```python
def build_learning_prompt(
    topic: str,
    *,
    level: str = "beginner",
    include_example: bool = True,
) -> str:
    cleaned_topic = topic.strip()

    if not cleaned_topic:
        raise ValueError("Topic cannot be empty.")

    prompt = f"Explain {cleaned_topic} for a {level} learner."

    if include_example:
        prompt += " Include one practical Python example."

    return prompt


print(build_learning_prompt(" Python dictionaries "))
print(build_learning_prompt("Embeddings", level="advanced", include_example=False))
```

## Program 4 — Handle expected errors

Create `lessons/day_03_program_04.py`:

```python
def validate_max_tokens(max_tokens: int) -> int:
    if max_tokens <= 0:
        raise ValueError("max_tokens must be greater than zero.")

    return max_tokens


values = [500, 0, -10]

for value in values:
    try:
        result = validate_max_tokens(value)
        print(f"{result} is valid.")
    except ValueError as error:
        print(f"{value} is invalid: {error}")
```

`try/except` will be covered deeply in Day 4. For now, understand that it lets the program handle an expected error without crashing.

# Day 3 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. What is a function’s main purpose?

   A. Store files permanently  
   B. Create reusable behavior for a clear task  
   C. Replace Python types  
   D. Install packages  

2. What does this mean?

```python
def build_prompt(topic: str) -> str:
```

   A. Input should be a string and output is expected to be a string  
   B. Python converts every input into a string  
   C. The function cannot return anything  
   D. `topic` must be a list  

3. Which is a keyword argument call?

   A. `create_config("gpt-5", 0.7)`  
   B. `create_config(model="gpt-5", temperature=0.7)`  
   C. `create_config = "gpt-5"`  
   D. `config.create("gpt-5")`  

4. What does `*` do here?

```python
def create_config(model: str, *, temperature: float = 0.7):
```

   A. Multiplies values  
   B. Makes `model` optional  
   C. Requires `temperature` to be passed by name  
   D. Converts temperature to an integer  

5. Which call is valid?

```python
def create_config(model: str, *, temperature: float = 0.7):
    ...
```

   A. `create_config("gpt-5", 0.3)`  
   B. `create_config("gpt-5", temperature=0.3)`  
   C. `create_config(model="gpt-5", 0.3)`  
   D. `create_config(temperature=0.3, "gpt-5")`  

6. What does `raise ValueError("Invalid input")` do?

   A. Ignores invalid input  
   B. Stops normal flow and signals invalid input  
   C. Prints only a warning  
   D. Converts input to a string  

7. Where does a local variable exist?

   A. In every file globally  
   B. Only inside the function/block where it was created  
   C. Only in databases  
   D. Only after the program ends  

8. Why should application code avoid unnecessary `global` variables?

   A. They make code harder to reason about and test  
   B. Python forbids them  
   C. They improve security too much  
   D. Functions cannot access them  

9. What does a default parameter do?

   A. Deletes an argument  
   B. Supplies a value when the caller does not provide one  
   C. Makes every parameter optional  
   D. Prevents validation  

10. What is wrong with this function?

```python
def add_message(message: str, history: list = []):
```

   A. Nothing is wrong  
   B. Functions cannot accept lists  
   C. The same list can be reused across calls  
   D. `message` must be a dictionary  

11. Which is the safe alternative?

   A. `history: list = {}`  
   B. `history: list | None = None`, then create `[]` inside  
   C. `history: str = ""`  
   D. Remove `history` entirely  

12. Why do we validate an LLM request before sending it?

   A. To make the model slower  
   B. To catch empty prompts and invalid settings early  
   C. To remove type hints  
   D. To avoid using dictionaries  

13. What does `return` do?

   A. Continues the loop  
   B. Ends a function and optionally sends a value back  
   C. Restarts the function forever  
   D. Installs a package  

14. Which function design is best?

   A. A 500-line function handling every application task  
   B. A small function with one clear responsibility  
   C. A function depending on many globals  
   D. A function with no validation  

15. Why are keyword-only arguments useful for LLM settings?

   A. They make `temperature` harder to read  
   B. They prevent accidentally mixing up values such as temperature and max tokens  
   C. They stop a function from returning data  
   D. They remove all defaults
