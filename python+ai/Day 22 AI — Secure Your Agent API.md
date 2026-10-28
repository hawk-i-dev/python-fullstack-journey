# Day 22 AI — Secure Your Agent API

Today you turn your Day 21 API from a demo into a safer multi-user service.

Your new rule is:

```text
Client token → identify user on server → authorize request → run workflow → return only that user’s data
```

Do not trust `user_id` sent in a JSON request. A user could change it to another person’s ID. The server must determine identity from authentication.

For agent tools, apply validation and authorization at the actual tool/action boundary—not only in the frontend or prompt. That is the same production principle emphasized in the [official OpenAI guardrails guidance](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals).

## Today’s outcomes

By the end of Day 22, your API will have:

- Bearer-token authentication
- Server-controlled user identity
- Protected `/assist` endpoint
- Trace ownership checks
- Tests for authenticated and unauthenticated requests

Estimated time: 1–3 hours.

## 1. Add a development token to `.env`

In your project root:

```env
DEMO_API_TOKENS=dev-alice-token:alice,dev-bob-token:bob
```

Never commit `.env`. Add this to `.env.example` instead:

```env
DEMO_API_TOKENS=your-token:user-id
```

This is a learning-only token format. In corporate systems, use an identity provider, OAuth, or signed JWT tokens—never a hard-coded password or token in source code.

## 2. Create `app/auth.py`

```python
import os
import secrets
from dataclasses import dataclass

from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer


security = HTTPBearer(auto_error=False)


@dataclass(frozen=True)
class CurrentUser:
    user_id: str


def get_demo_token_map() -> dict[str, str]:
    raw_tokens = os.getenv("DEMO_API_TOKENS", "")

    token_map: dict[str, str] = {}

    for item in raw_tokens.split(","):
        if ":" not in item:
            continue

        token, user_id = item.split(":", maxsplit=1)
        token_map[token.strip()] = user_id.strip()

    return token_map


def get_current_user(
    credentials: HTTPAuthorizationCredentials | None = Depends(security),
) -> CurrentUser:
    if credentials is None or credentials.scheme.lower() != "bearer":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Missing Bearer token",
        )

    for valid_token, user_id in get_demo_token_map().items():
        if secrets.compare_digest(credentials.credentials, valid_token):
            return CurrentUser(user_id=user_id)

    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Invalid token",
    )
```

Important concepts:

- **Authentication**: Who is calling?
- **Authorization**: Are they allowed to access this resource?
- `HTTPBearer` reads `Authorization: Bearer <token>`.
- `compare_digest()` avoids unsafe ordinary secret comparison.
- `CurrentUser` is created on the server, not trusted from browser input.

## 3. Protect `/assist`

In `app/main.py`, import:

```python
from fastapi import Depends

from app.auth import CurrentUser, get_current_user
```

Remove `user_id` from `AssistRequest`. The client should send only its question and optional expense values.

Then update the endpoint:

```python
@app.post("/assist", response_model=AssistResponse)
def assist(
    request: AssistRequest,
    current_user: CurrentUser = Depends(get_current_user),
) -> AssistResponse:
    result = run_observed_workflow(
        user_id=current_user.user_id,
        question=request.question,
        expense_title=request.expense_title,
        monthly_limit=request.monthly_limit,
        spent=request.spent,
        chunks=[],
    )

    return AssistResponse(
        ok=True,
        intent=result["intent"],
        message=result["message"],
        results=result.get("results", []),
        trace_file=result.get("trace_file"),
    )
```

Now Swagger must send this header:

```text
Authorization: Bearer dev-alice-token
```

## 4. Enforce trace ownership

When saving a trace, include its owner:

```python
trace_data["owner_id"] = current_user.user_id
```

Then update the trace endpoint conceptually:

```python
if trace_data.get("owner_id") != current_user.user_id:
    raise HTTPException(status_code=403, detail="You cannot access this trace")
```

Status meanings:

| Status | Meaning |
|---|---|
| `401` | No valid login/token |
| `403` | Logged in, but not allowed |
| `404` | Resource does not exist |
| `422` | Request validation failed |
| `500` | Unexpected server failure |

For security, a real client app often returns `404` instead of confirming another user’s private trace exists. We will cover that more deeply later.

## 5. Test authentication

Create `tests/test_auth_api.py`:

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_assist_rejects_request_without_token():
    response = client.post(
        "/assist",
        json={"question": "How much budget is left?"},
    )

    assert response.status_code == 401


def test_assist_accepts_valid_token(monkeypatch):
    monkeypatch.setenv(
        "DEMO_API_TOKENS",
        "dev-alice-token:alice",
    )

    response = client.post(
        "/assist",
        headers={"Authorization": "Bearer dev-alice-token"},
        json={
            "question": "How much budget is left?",
            "monthly_limit": 10000,
            "spent": 2500,
        },
    )

    assert response.status_code == 200


def test_assist_rejects_invalid_token(monkeypatch):
    monkeypatch.setenv(
        "DEMO_API_TOKENS",
        "dev-alice-token:alice",
    )

    response = client.post(
        "/assist",
        headers={"Authorization": "Bearer wrong-token"},
        json={"question": "Show my expenses"},
    )

    assert response.status_code == 401
```

Run:

```bash
pytest -q
```

## Day 22 Assignment

Implement and verify these:

1. Remove `user_id` from the request body.
2. Add `app/auth.py`.
3. Protect `/assist`.
4. Ensure Alice cannot read Bob’s trace.
5. Write tests for `401`, successful `200`, and `403`/hidden trace behavior.
6. Update your README with the required Authorization header.

## Corporate and client mindset

Before exposing an agent endpoint, ask:

- Who is calling this API?
- How do we prove their identity?
- Which user’s data can they access?
- Can a request cause a financial, destructive, or external action?
- Is that action logged and approval-protected?
- Are secrets excluded from source code, logs, and API responses?

This is how an AI project becomes a client-safe application, not just a working demo.
