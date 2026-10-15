# Day 19 AI — Guardrails, Approvals, and Audit Logs

Today you will add a policy layer before agent tools execute.

```text
User / LLM tool request
        ↓
Guardrail validates tool + arguments
        ↓
Allow / deny / needs approval
        ↓
Audit event is recorded
        ↓
Only allowed actions execute
```

Guardrails are automatic checks on input, output, or tool behavior. Human approval is different: it pauses a sensitive action until a person approves or rejects it. Tool-level validation should exist beside the tool that creates the side effect. [Official OpenAI guardrails and approvals guide](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)

## Today’s project

Add a safe policy layer to your Finance Assistant Agent.

Rules:

- Read-only tools such as `search_knowledge` and `calculate_budget` are allowed.
- Unknown tools are denied.
- Sensitive actions such as `transfer_money` or `delete_expense` need approval.
- API keys, passwords, and tokens must never be written to logs.

No Mac/Windows code difference.

## 1. Create `agent_guardrails.py`

```python
from dataclasses import dataclass
from enum import StrEnum
from typing import Any


class Decision(StrEnum):
    ALLOW = "allow"
    DENY = "deny"
    NEEDS_APPROVAL = "needs_approval"


@dataclass(frozen=True)
class GuardrailResult:
    decision: Decision
    reason: str


READ_ONLY_TOOLS = {
    "search_knowledge",
    "calculate_budget",
    "categorize_expense",
    "calculate_savings_rate",
}

SENSITIVE_TOOLS = {
    "transfer_money",
    "delete_expense",
    "update_expense",
}


def evaluate_tool_request(
    tool_name: str,
    arguments: dict[str, Any],
) -> GuardrailResult:
    if tool_name in READ_ONLY_TOOLS:
        return GuardrailResult(
            decision=Decision.ALLOW,
            reason="This is an approved read-only tool.",
        )

    if tool_name in SENSITIVE_TOOLS:
        return GuardrailResult(
            decision=Decision.NEEDS_APPROVAL,
            reason="This action can change financial data or move money.",
        )

    return GuardrailResult(
        decision=Decision.DENY,
        reason=f"Tool '{tool_name}' is not allowed.",
    )
```

## 2. Create `audit_log.py`

```python
import json
from datetime import datetime, timezone
from pathlib import Path
from typing import Any


def write_audit_event(
    *,
    file_path: Path,
    user_id: str,
    tool_name: str,
    decision: str,
    reason: str,
    arguments: dict[str, Any],
) -> None:
    event = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "user_id": user_id,
        "tool_name": tool_name,
        "decision": decision,
        "reason": reason,
        "arguments": arguments,
    }

    file_path.parent.mkdir(parents=True, exist_ok=True)

    with file_path.open("a", encoding="utf-8") as file:
        file.write(json.dumps(event) + "\n")
```

Use `JSON Lines` format: one JSON object per line. It is easy to inspect, filter, and send to a logging system later.

## 3. Create `guarded_executor.py`

```python
from pathlib import Path
from typing import Any

from agent_guardrails import Decision, evaluate_tool_request
from agent_tools import execute_tool
from audit_log import write_audit_event


AUDIT_FILE = Path("data/audit/agent_events.jsonl")


def guarded_execute(
    *,
    user_id: str,
    tool_name: str,
    arguments: dict[str, Any],
    chunks: list[dict[str, Any]],
) -> dict[str, Any]:
    guardrail = evaluate_tool_request(tool_name, arguments)

    write_audit_event(
        file_path=AUDIT_FILE,
        user_id=user_id,
        tool_name=tool_name,
        decision=guardrail.decision,
        reason=guardrail.reason,
        arguments=arguments,
    )

    if guardrail.decision == Decision.DENY:
        return {
            "ok": False,
            "status": "denied",
            "message": guardrail.reason,
        }

    if guardrail.decision == Decision.NEEDS_APPROVAL:
        return {
            "ok": False,
            "status": "pending_approval",
            "message": guardrail.reason,
        }

    result = execute_tool(
        tool_name,
        arguments,
        chunks=chunks,
    )

    return {
        "ok": result["ok"],
        "status": "executed",
        "result": result,
    }
```

## 4. Create `main_guardrails_demo.py`

```python
from guarded_executor import guarded_execute


safe_result = guarded_execute(
    user_id="user-101",
    tool_name="calculate_budget",
    arguments={
        "monthly_limit": 30000,
        "spent": 18450,
    },
    chunks=[],
)

sensitive_result = guarded_execute(
    user_id="user-101",
    tool_name="transfer_money",
    arguments={
        "amount": 5000,
        "recipient": "Someone",
    },
    chunks=[],
)

unknown_result = guarded_execute(
    user_id="user-101",
    tool_name="delete_everything",
    arguments={},
    chunks=[],
)

print(safe_result)
print(sensitive_result)
print(unknown_result)
```

Run:

```bash
python main_guardrails_demo.py
```

Expected behavior:

- `calculate_budget` → executes.
- `transfer_money` → `pending_approval`.
- `delete_everything` → denied.

## 5. Add tests

Create `tests/test_agent_guardrails.py`:

```python
from agent_guardrails import Decision, evaluate_tool_request


def test_read_only_tool_is_allowed():
    result = evaluate_tool_request(
        "calculate_budget",
        {"monthly_limit": 30000, "spent": 10000},
    )

    assert result.decision == Decision.ALLOW


def test_sensitive_tool_needs_approval():
    result = evaluate_tool_request(
        "transfer_money",
        {"amount": 5000},
    )

    assert result.decision == Decision.NEEDS_APPROVAL


def test_unknown_tool_is_denied():
    result = evaluate_tool_request(
        "delete_everything",
        {},
    )

    assert result.decision == Decision.DENY
```

Run:

```bash
pytest
```

## Institute, corporate, and client approach

| Level | What matters |
|---|---|
| Institute | Understand allow, deny, and approval states |
| Corporate | Add authentication, audit logs, input validation, timeouts, and approval workflows |
| Client | Clearly say what will happen before sensitive actions; never hide an approval requirement |

## Day 19 assignment

Add a confirmation token flow for `delete_expense`.

Requirements:

- `delete_expense` initially returns `pending_approval`.
- Generate a random confirmation token.
- Store it temporarily in memory.
- Execute deletion only if the user sends the exact token.
- Expire the token after 10 minutes.
- Add tests for missing, valid, and expired tokens.

By the end of Day 19, you should understand that safety is not one prompt. It is enforced through policy code, validation, approvals, logs, and tests.
