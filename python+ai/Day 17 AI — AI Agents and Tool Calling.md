# Day 17 AI — AI Agents and Tool Calling

Today you will build the core of an AI agent: a system where the model can decide to request a tool, your Python code validates and runs that tool, and the model uses the result to answer.

An agent is not “an LLM with a fancy prompt.” It is a controlled loop:

```text
User question
   ↓
LLM decides whether a tool is needed
   ↓
Your Python code validates and executes an allowed tool
   ↓
Tool result returns to LLM
   ↓
Final answer with evidence
```

Function/tool calling is a multi-step application-controlled process. The model requests a tool; your code executes it; then you send the result back for a final response. Treat every tool call as untrusted input: validate it, whitelist tool names, and do not allow destructive actions automatically. [Official OpenAI function-calling guide](https://developers.openai.com/api/docs/guides/function-calling)

## Today’s project

Extend `day_12_document_chunker` into a **Personal Finance Assistant Agent** with two safe, read-only tools:

- `search_knowledge`: searches your local RAG chunks.
- `calculate_budget`: calculates remaining monthly budget.

No Mac/Windows code difference today.

Activate the environment:

```bash
# Mac
source .venv/bin/activate
```

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

## 1. Create `agent_tools.py`

```python
import json
from typing import Any

from vector_search import SearchResult, search_chunks


TOOLS = [
    {
        "type": "function",
        "name": "search_knowledge",
        "description": "Search the approved local knowledge base for information.",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "A short search query.",
                },
            },
            "required": ["query"],
            "additionalProperties": False,
        },
        "strict": True,
    },
    {
        "type": "function",
        "name": "calculate_budget",
        "description": "Calculate the remaining amount in a monthly budget.",
        "parameters": {
            "type": "object",
            "properties": {
                "monthly_limit": {
                    "type": "number",
                    "description": "The user's monthly budget limit.",
                },
                "spent": {
                    "type": "number",
                    "description": "The amount already spent this month.",
                },
            },
            "required": ["monthly_limit", "spent"],
            "additionalProperties": False,
        },
        "strict": True,
    },
]


def execute_tool(
    name: str,
    arguments: dict[str, Any],
    *,
    chunks: list[dict[str, Any]],
) -> dict[str, Any]:
    """Run only approved, validated, read-only tools."""

    if name == "search_knowledge":
        query = arguments.get("query")

        if not isinstance(query, str) or not query.strip():
            return {"ok": False, "error": "query must be a non-empty string"}

        results: list[SearchResult] = search_chunks(chunks, query.strip())

        return {
            "ok": True,
            "results": [
                {
                    "source": result.source,
                    "text": result.text,
                    "score": round(result.score, 3),
                }
                for result in results
            ],
        }

    if name == "calculate_budget":
        monthly_limit = arguments.get("monthly_limit")
        spent = arguments.get("spent")

        if not isinstance(monthly_limit, (int, float)):
            return {"ok": False, "error": "monthly_limit must be a number"}

        if not isinstance(spent, (int, float)):
            return {"ok": False, "error": "spent must be a number"}

        if monthly_limit < 0 or spent < 0:
            return {"ok": False, "error": "amounts cannot be negative"}

        return {
            "ok": True,
            "monthly_limit": monthly_limit,
            "spent": spent,
            "remaining": monthly_limit - spent,
            "over_budget": spent > monthly_limit,
        }

    return {
        "ok": False,
        "error": f"Tool '{name}' is not approved.",
    }


def tool_output_as_json(result: dict[str, Any]) -> str:
    return json.dumps(result, ensure_ascii=False)
```

Important concepts:

- `TOOLS` is the agent’s capability list.
- `execute_tool()` is the security boundary.
- A model may request a tool, but it must never directly run Python, delete files, send money, or modify a database.
- `strict: True` makes function-call arguments follow the JSON schema more reliably; strict schemas require declared properties and `additionalProperties: false`. [Official OpenAI function-calling guide](https://developers.openai.com/api/docs/guides/function-calling)

## 2. Create `agent_loop.py`

```python
import json
from typing import Any

from agent_tools import TOOLS, execute_tool, tool_output_as_json


SYSTEM_INSTRUCTIONS = """
You are a personal finance learning assistant.

Use search_knowledge only when the answer requires the approved knowledge base.
Use calculate_budget only for arithmetic about a monthly budget.

Never claim you completed an action you cannot verify.
Use tool results accurately.
If a tool returns an error, explain it clearly to the user.
"""


def run_agent_turn(
    client: Any,
    question: str,
    *,
    chunks: list[dict[str, Any]],
    model: str,
) -> str:
    """Run a bounded tool-calling loop."""

    input_items: list[Any] = [
        {
            "role": "user",
            "content": f"{SYSTEM_INSTRUCTIONS}\n\nUser question: {question}",
        }
    ]

    max_rounds = 3

    for _ in range(max_rounds):
        response = client.responses.create(
            model=model,
            input=input_items,
            tools=TOOLS,
            store=False,
        )

        tool_calls = [
            item
            for item in response.output
            if item.type == "function_call"
        ]

        if not tool_calls:
            return response.output_text.strip()

        # Preserve the model output before returning tool results.
        input_items.extend(response.output)

        for tool_call in tool_calls:
            try:
                arguments = json.loads(tool_call.arguments)
            except json.JSONDecodeError:
                result = {
                    "ok": False,
                    "error": "The tool arguments were not valid JSON.",
                }
            else:
                result = execute_tool(
                    tool_call.name,
                    arguments,
                    chunks=chunks,
                )

            input_items.append(
                {
                    "type": "function_call_output",
                    "call_id": tool_call.call_id,
                    "output": tool_output_as_json(result),
                }
            )

    return "I reached the maximum allowed tool steps. Please try a simpler question."
```

The `max_rounds = 3` limit prevents an accidental endless agent loop. In corporate projects, every agent must have limits for tool calls, time, token usage, and spending.

## 3. Create `main_agent.py`

```python
import os

from dotenv import load_dotenv
from openai import OpenAI

from agent_loop import run_agent_turn
from vector_search import load_chunks

load_dotenv()

MODEL = os.getenv("OPENAI_MODEL", "gpt-5.5")
CHUNKS_FILE = "data/processed/ai_rag_chunks.json"


def main() -> None:
    client = OpenAI()
    chunks = load_chunks(CHUNKS_FILE)

    print("Finance Assistant Agent")
    print("Type /exit to close.")

    while True:
        question = input("\nYou: ").strip()

        if question.lower() == "/exit":
            print("Goodbye.")
            break

        if not question:
            continue

        answer = run_agent_turn(
            client,
            question,
            chunks=chunks,
            model=MODEL,
        )

        print(f"\nAssistant: {answer}")


if __name__ == "__main__":
    main()
```

Run it:

```bash
python main_agent.py
```

Try these:

```text
What is RAG?
My monthly limit is 30000 and I spent 18450. How much remains?
Explain chunking using the knowledge base.
```

## Testing before live API use

Create `tests/test_agent_tools.py`:

```python
from agent_tools import execute_tool


def test_calculate_budget_returns_remaining_amount():
    result = execute_tool(
        "calculate_budget",
        {"monthly_limit": 30000, "spent": 18450},
        chunks=[],
    )

    assert result["ok"] is True
    assert result["remaining"] == 11550
    assert result["over_budget"] is False


def test_calculate_budget_rejects_negative_numbers():
    result = execute_tool(
        "calculate_budget",
        {"monthly_limit": 30000, "spent": -10},
        chunks=[],
    )

    assert result["ok"] is False


def test_unknown_tool_is_rejected():
    result = execute_tool(
        "delete_all_expenses",
        {},
        chunks=[],
    )

    assert result["ok"] is False
    assert "not approved" in result["error"]
```

Run:

```bash
pytest
```

## Real project approach

Institute level:

- Understand tool schemas, JSON, validation, and unit tests.
- Build only safe local tools first.

Corporate level:

- Use an allowlist of tools.
- Log tool name, user ID, arguments, result status, latency, and errors.
- Add limits for retries, number of tool calls, cost, and execution time.
- Test tools without making real API calls.

Client level:

- Never allow a financial agent to transfer money, delete expenses, or change budgets without explicit user confirmation.
- Clearly show what data was used and what action will happen.
- Say “I cannot complete that action” instead of pretending it succeeded.

## Day 17 assignment

Add a third safe tool: `categorize_expense`.

Input:

```python
{
    "title": "Uber ride"
}
```

Expected output:

```python
{
    "ok": True,
    "category": "Transport"
}
```

Rules:

- Handle at least Food, Transport, Bills, Shopping, and Other.
- Use simple Python keyword rules; do not use the LLM for classification yet.
- Add three pytest tests.
- Reject an empty title.

By the end of Day 17, you should understand the difference between an LLM response and an agent: an agent can reason about when to use controlled tools, but your application remains responsible for safety.
