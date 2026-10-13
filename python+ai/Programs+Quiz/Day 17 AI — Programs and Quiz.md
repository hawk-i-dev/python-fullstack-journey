## Day 17 AI — Programs and Quiz

Tool calls can be zero, one, or many in a single model response, so your agent loop must handle every requested call safely. Keep tool schemas strict, validate inputs again in Python, and allow only approved tools. [Official OpenAI function-calling guide](https://developers.openai.com/api/docs/guides/function-calling)

### Program 1: Expense categorizer tool

Add this inside `execute_tool()` in `agent_tools.py`, before the final unknown-tool return:

```python
if name == "categorize_expense":
    title = arguments.get("title")

    if not isinstance(title, str) or not title.strip():
        return {"ok": False, "error": "title must be a non-empty string"}

    normalized_title = title.lower()

    categories = {
        "Food": ("restaurant", "breakfast", "lunch", "dinner", "grocery", "swiggy"),
        "Transport": ("uber", "ola", "petrol", "fuel", "metro", "bus"),
        "Bills": ("electricity", "wifi", "internet", "rent", "mobile"),
        "Shopping": ("amazon", "clothes", "shoes", "shopping"),
    }

    for category, keywords in categories.items():
        if any(keyword in normalized_title for keyword in keywords):
            return {"ok": True, "category": category}

    return {"ok": True, "category": "Other"}
```

Also add this tool schema to `TOOLS`:

```python
{
    "type": "function",
    "name": "categorize_expense",
    "description": "Categorize an expense title into a personal-finance category.",
    "parameters": {
        "type": "object",
        "properties": {
            "title": {
                "type": "string",
                "description": "Expense title, such as Uber ride or Grocery shopping.",
            },
        },
        "required": ["title"],
        "additionalProperties": False,
    },
    "strict": True,
},
```

### Program 2: Test your tool

Add to `tests/test_agent_tools.py`:

```python
from agent_tools import execute_tool


def test_categorize_uber_as_transport():
    result = execute_tool(
        "categorize_expense",
        {"title": "Uber ride to office"},
        chunks=[],
    )

    assert result == {"ok": True, "category": "Transport"}


def test_categorize_grocery_as_food():
    result = execute_tool(
        "categorize_expense",
        {"title": "Grocery shopping"},
        chunks=[],
    )

    assert result == {"ok": True, "category": "Food"}


def test_categorize_rejects_empty_title():
    result = execute_tool(
        "categorize_expense",
        {"title": "   "},
        chunks=[],
    )

    assert result["ok"] is False
```

Run:

```bash
pytest
```

### Program 3: Tool safety audit

Create `tool_audit.py`:

```python
from agent_tools import execute_tool


tool_requests = [
    ("calculate_budget", {"monthly_limit": 25000, "spent": 18000}),
    ("categorize_expense", {"title": "Electricity bill"}),
    ("delete_all_expenses", {}),
]

for tool_name, arguments in tool_requests:
    result = execute_tool(tool_name, arguments, chunks=[])

    print(f"Tool: {tool_name}")
    print(f"Arguments: {arguments}")
    print(f"Result: {result}")
    print("-" * 40)
```

Expected learning:

- Approved tools run.
- Invalid or dangerous tool names are rejected.
- An agent never gets unrestricted Python/database/file access.

### Assignment

Create one more read-only tool: `calculate_savings_rate`.

Inputs:

```python
{
    "income": 60000,
    "spent": 35000
}
```

Expected output:

```python
{
    "ok": True,
    "savings": 25000,
    "savings_rate_percent": 41.67
}
```

Add three tests:

- correct savings rate
- zero income rejected
- negative amount rejected

---

## Day 17 AI Quiz

Reply like: `1.B, 2.A, ...`

1. What is an AI agent tool?

   A. A random Python file  
   B. A controlled capability the model may request  
   C. An API key  
   D. A database table  

2. Who actually executes a function tool?

   A. The LLM directly  
   B. The browser automatically  
   C. Your application code  
   D. The JSON schema  

3. Why must tool arguments be validated in Python?

   A. Tool-call arguments can be incorrect or unsafe  
   B. JSON is always invalid  
   C. Models cannot return text  
   D. Python has no types  

4. What should happen if the model requests an unknown tool?

   A. Run it anyway  
   B. Reject it with a safe error result  
   C. Delete the tool list  
   D. Stop the server permanently  

5. Why use an allowlist of tools?

   A. To give the model unlimited access  
   B. To restrict the agent to approved capabilities  
   C. To avoid writing tests  
   D. To remove all errors  

6. What is the correct high-level order for a tool-calling loop?

   A. Final answer → tool → user  
   B. Tool result → question → model  
   C. Model request → tool call → Python execution → tool output → final response  
   D. Database deletion → response  

7. What does `strict: True` help with?

   A. More reliable adherence to the function schema  
   B. Faster internet speed  
   C. Automatic authentication  
   D. Deleting invalid code  

8. With strict schemas, what should be set to `False`?

   A. `required`  
   B. `additionalProperties`  
   C. `type`  
   D. `description`  

9. Why should an agent have `max_rounds`?

   A. To make tool names longer  
   B. To prevent unlimited looping and cost  
   C. To remove tool results  
   D. To avoid JSON  

10. Which tool is safest for the first finance-agent version?

   A. `delete_all_expenses`  
   B. `transfer_money`  
   C. `calculate_budget`  
   D. `change_bank_password`  

11. If the model requests two tool calls, what should your code assume?

   A. Only one can ever be valid  
   B. It may need to process both safely  
   C. Ignore both calls  
   D. Restart Python  

12. Why should tool results be returned as structured JSON?

   A. So the model receives clear, machine-readable facts  
   B. To hide all results from the model  
   C. To skip validation  
   D. To avoid unit tests  

13. What should a production agent log?

   A. Tool name, result status, timing, and safe audit details  
   B. User passwords and API keys  
   C. Nothing  
   D. Only CSS styles  

14. Which action normally needs explicit user confirmation?

   A. Searching local RAG chunks  
   B. Calculating remaining budget  
   C. Transferring money  
   D. Categorizing “Uber ride”  

15. Why do we test `execute_tool()` without a live LLM API call?

   A. It is faster, cheaper, repeatable, and isolates tool logic  
   B. Agents never need tests  
   C. APIs cannot return text  
   D. Python cannot use functions
