# Day 2 AI — Extra Programs + Quiz

**Same Python code on Mac and Windows.** Activate `.venv`, then run:

```bash
python lessons/file_name.py
ruff check .
```

## Program 1 — Temperature classifier

Create `lessons/day_02_program_01.py`:

```python
def describe_temperature(temperature: float) -> str:
    if temperature < 0 or temperature > 2:
        return "Invalid temperature. Use a value from 0 to 2."

    if temperature <= 0.3:
        return "Focused output"
    elif temperature <= 1.0:
        return "Balanced output"
    else:
        return "Creative output"


for value in [0, 0.2, 0.7, 1.5, 2.5]:
    print(f"{value}: {describe_temperature(value)}")
```

## Program 2 — Clean prompts

Create `lessons/day_02_program_02.py`:

```python
raw_prompts = [
    " Explain Python lists ",
    "",
    "   ",
    "What is RAG?",
    " Explain AI agents ",
]

clean_prompts = [
    prompt.strip()
    for prompt in raw_prompts
    if prompt.strip()
]

for index, prompt in enumerate(clean_prompts, start=1):
    print(f"{index}. {prompt}")
```

## Program 3 — Count chat messages by role

Create `lessons/day_02_program_03.py`:

```python
def count_messages_by_role(
    conversation: list[dict[str, str]],
) -> dict[str, int]:
    counts: dict[str, int] = {
        "system": 0,
        "user": 0,
        "assistant": 0,
    }

    for message in conversation:
        role = message.get("role")

        if role in counts:
            counts[role] += 1

    return counts


conversation = [
    {"role": "system", "content": "You are a helpful AI tutor."},
    {"role": "user", "content": "What is Python?"},
    {"role": "assistant", "content": "Python is a programming language."},
    {"role": "user", "content": "What is RAG?"},
]

print(count_messages_by_role(conversation))
```

Expected:

```python
{'system': 1, 'user': 2, 'assistant': 1}
```

## Program 4 — Find the first user question

Create `lessons/day_02_program_04.py`:

```python
def find_first_user_question(
    conversation: list[dict[str, str]],
) -> str | None:
    for message in conversation:
        if message.get("role") == "user":
            return message.get("content", "").strip()

    return None


conversation = [
    {"role": "system", "content": "You are helpful."},
    {"role": "assistant", "content": "Hello!"},
    {"role": "user", "content": "How does an LLM work?"},
    {"role": "user", "content": "What is an embedding?"},
]

question = find_first_user_question(conversation)

if question is None:
    print("No user question found.")
else:
    print(f"First question: {question}")
```

---

# Day 2 AI MCQ Quiz

Reply like: `1.B 2.C 3.A ...`

1. Which operator compares two values for equality?

   A. `=`  
   B. `==`  
   C. `is`  
   D. `:=`  

2. When should `is` commonly be used?

   A. Comparing two strings  
   B. Comparing two integers  
   C. Checking whether a value is `None`  
   D. Comparing two lists  

3. Which value is falsy?

   A. `"False"`  
   B. `[0]`  
   C. `{}`  
   D. `" "`  

4. What is printed?

```python
prompt = ""

if not prompt:
    print("Missing")
```

   A. Nothing  
   B. `False`  
   C. `Missing`  
   D. Error  

5. What does `continue` do inside a loop?

   A. Ends the complete loop  
   B. Skips the current iteration  
   C. Restarts the program  
   D. Returns a value  

6. What does `break` do inside a loop?

   A. Skips one item only  
   B. Ends the current loop  
   C. Deletes the list  
   D. Raises an exception  

7. What does `enumerate(items, start=1)` provide?

   A. Only each value  
   B. Only each index  
   C. An index and each value  
   D. A dictionary  

8. Which correctly loops through both keys and values of a dictionary?

   A. `for key, value in data.items():`  
   B. `for key, value in data:`  
   C. `for value in data.keys():`  
   D. `for item in data.append():`  

9. What is the output?

```python
numbers = [1, 2, 3]
doubled = [number * 2 for number in numbers]
print(doubled)
```

   A. `[1, 2, 3]`  
   B. `[2, 4, 6]`  
   C. `[1, 4, 9]`  
   D. Error  

10. What does `.strip()` remove?

   A. All spaces in a string  
   B. Only spaces in the middle  
   C. Leading and trailing whitespace  
   D. All punctuation  

11. Which is safest for checking a missing optional value?

   A. `if value == []:`  
   B. `if value is None:`  
   C. `if value is 0:`  
   D. `if value == False:`  

12. What happens when a function reaches `return`?

   A. It starts again  
   B. It skips the next loop item  
   C. It immediately finishes and gives back a value  
   D. It deletes local variables globally  

13. Which is best for simple filtering and transforming a list?

   A. A list comprehension  
   B. `break`  
   C. `id()`  
   D. `type()`  

14. What is the result?

```python
role = "assistant"

if role == "user":
    result = "Question"
else:
    result = "Response"
```

   A. `Question`  
   B. `Response`  
   C. `assistant`  
   D. Error  

15. Why validate AI messages before sending them to an LLM API?

   A. To make Python slower  
   B. To prevent invalid roles, empty content, and bad input  
   C. To remove all dictionaries  
   D. To avoid using functions
