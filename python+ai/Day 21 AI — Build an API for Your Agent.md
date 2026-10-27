# Day 21 AI — Build an API for Your Agent

We are continuing the AI track. Today your Finance Assistant Agent becomes a real backend service that a web frontend, mobile app, or client application can call.

```text
Frontend / Swagger / mobile app
            ↓ HTTP request
       FastAPI endpoint
            ↓
 Agent workflow + guardrails + traces
            ↓
        JSON response
```

A production AI system needs an application layer around the model, tools, state, safety policies, and logs. You control this layer when using your own backend workflow. [OpenAI agent-runtime overview](https://developers.openai.com/api/docs/guides/agents)

## Goal by EOD

Build these endpoints:

```text
GET  /health
POST /assist
```

## 1. Install backend packages

Activate your environment first.

```bash
# Mac
source .venv/bin/activate
```

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

Install:

```bash
python -m pip install fastapi "uvicorn[standard]"
```

Create this structure beside your existing agent files:

```text
day_12_document_chunker/
├── app/
│   ├── __init__.py
│   ├── main.py
│   └── schemas.py
├── agent_workflow.py
├── observed_workflow.py
└── ...
```

## 2. Create `app/schemas.py`

```python
from typing import Any

from pydantic import BaseModel, Field


class AssistRequest(BaseModel):
    user_id: str = Field(
        min_length=3,
        max_length=100,
        description="Safe application user identifier.",
    )
    question: str = Field(
        min_length=3,
        max_length=500,
        description="User question for the finance assistant.",
    )
    expense_title: str | None = Field(
        default=None,
        max_length=200,
    )
    monthly_limit: float | None = Field(
        default=None,
        ge=0,
    )
    spent: float | None = Field(
        default=None,
        ge=0,
    )


class AssistResponse(BaseModel):
    ok: bool
    intent: str
    message: str
    results: list[dict[str, Any]]
    trace_file: str | None = None


class HealthResponse(BaseModel):
    status: str
```

Pydantic validates API input before your workflow runs. For example, a negative `spent` value returns a validation response instead of entering the agent system.

## 3. Create `app/main.py`

```python
import logging
from typing import Any

from fastapi import FastAPI, HTTPException

from app.schemas import AssistRequest, AssistResponse, HealthResponse
from observed_workflow import run_observed_workflow


logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="Finance Assistant Agent API",
    version="0.1.0",
)


@app.get("/health", response_model=HealthResponse)
def health_check() -> HealthResponse:
    return HealthResponse(status="ok")


@app.post("/assist", response_model=AssistResponse)
def assist(request: AssistRequest) -> AssistResponse:
    try:
        result: dict[str, Any] = run_observed_workflow(
            question=request.question,
            expense_title=request.expense_title,
            monthly_limit=request.monthly_limit,
            spent=request.spent,
            chunks=[],
        )

        return AssistResponse(
            ok=result["ok"],
            intent=result["intent"],
            message=result["message"],
            results=result["results"],
            trace_file=result.get("trace_file"),
        )

    except Exception:
        logger.exception(
            "Assistant workflow failed for user_id=%s",
            request.user_id,
        )

        raise HTTPException(
            status_code=500,
            detail="The assistant could not process the request.",
        )
```

Never return the raw internal error to a real client. Log it securely on the server, but show a safe message to the user.

## 4. Run the API

Run this from the project root, where the `app` folder exists:

```bash
python -m uvicorn app.main:app --reload
```

The command is the same on Mac and Windows.

Open Swagger UI:

```text
http://127.0.0.1:8000/docs
```

Test `GET /health`. Expected response:

```json
{
  "status": "ok"
}
```

Test `POST /assist` using:

```json
{
  "user_id": "user-101",
  "question": "Categorize this expense and check my remaining budget.",
  "expense_title": "Uber ride to office",
  "monthly_limit": 30000,
  "spent": 18450
}
```

Expected intent:

```json
"intent": "multi_tool"
```

## 5. Add API tests

Create `tests/test_api.py`:

```python
from fastapi.testclient import TestClient

from app.main import app


client = TestClient(app)


def test_health_endpoint():
    response = client.get("/health")

    assert response.status_code == 200
    assert response.json() == {"status": "ok"}


def test_assist_rejects_negative_spent_amount():
    response = client.post(
        "/assist",
        json={
            "user_id": "user-101",
            "question": "What is my remaining budget?",
            "monthly_limit": 30000,
            "spent": -100,
        },
    )

    assert response.status_code == 422


def test_assist_runs_budget_workflow():
    response = client.post(
        "/assist",
        json={
            "user_id": "user-101",
            "question": "What is my remaining budget?",
            "monthly_limit": 30000,
            "spent": 18450,
        },
    )

    assert response.status_code == 200
    assert response.json()["intent"] == "calculate_budget"
```

Run:

```bash
pytest
```

## Real-time project strategy

| Layer | Responsibility |
|---|---|
| API schema | Validate request shape and basic limits |
| Workflow | Route, plan, and run approved tools |
| Guardrails | Allow, deny, or require approval |
| Trace | Record safe workflow events |
| Tests | Prove the endpoint behaves correctly |

## Day 21 assignment

Add `GET /traces/{trace_id}`.

Requirements:

- Read only files inside `data/traces/`.
- Return `404` if the trace does not exist.
- Never accept a full arbitrary file path from the client.
- Add tests for a valid trace and missing trace.

This is the same thinking used in corporate APIs: validate at the boundary, call trusted internal code, return safe JSON, log failures, and test the contract.
