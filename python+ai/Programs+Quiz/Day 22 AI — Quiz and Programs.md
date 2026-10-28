# Day 22 AI — Quiz and Programs

Today’s theme: **authentication, authorization, and data isolation**.

A secure agent API must decide:

```text
Who is calling? → What can they access? → Can the requested tool/action run?
```

Agent guardrails are useful, but authorization must also be enforced at the API and tool boundary where the side effect happens. [Official OpenAI guardrails guidance](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)

Commands are the same on Mac and Windows after activating your virtual environment:

```bash
pytest -q
python -m uvicorn app.main:app --reload
```

## Program 1 — Test Bearer-token authentication

Create `tests/test_auth_api.py`:

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_assist_rejects_missing_token():
    response = client.post(
        "/assist",
        json={"question": "How much money is remaining?"},
    )

    assert response.status_code == 401


def test_assist_rejects_invalid_token(monkeypatch):
    monkeypatch.setenv(
        "DEMO_API_TOKENS",
        "dev-alice-token:alice",
    )

    response = client.post(
        "/assist",
        headers={"Authorization": "Bearer invalid-token"},
        json={"question": "Show my spending"},
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
            "question": "How much budget remains?",
            "monthly_limit": 10000,
            "spent": 2500,
        },
    )

    assert response.status_code == 200
```

Run:

```bash
pytest -q
```

Expected understanding:

- No token → `401 Unauthorized`
- Wrong token → `401 Unauthorized`
- Correct token → `200 OK`

## Program 2 — Prevent another user from reading a trace

Your trace JSON should contain the server-owned user ID:

```json
{
  "trace_id": "123e4567-e89b-12d3-a456-426614174000",
  "owner_id": "alice",
  "events": []
}
```

Update the trace route in `app/main.py`:

```python
@app.get("/traces/{trace_id}")
def get_trace(
    trace_id: str,
    current_user: CurrentUser = Depends(get_current_user),
) -> dict[str, Any]:
    try:
        safe_trace_id = UUID(trace_id)
    except ValueError as error:
        raise HTTPException(
            status_code=400,
            detail="Invalid trace ID",
        ) from error

    trace_path = TRACE_DIRECTORY / f"{safe_trace_id}.json"

    if not trace_path.is_file():
        raise HTTPException(status_code=404, detail="Trace not found")

    trace_data = json.loads(trace_path.read_text(encoding="utf-8"))

    if trace_data.get("owner_id") != current_user.user_id:
        raise HTTPException(
            status_code=403,
            detail="You cannot access this trace",
        )

    return trace_data
```

Test it:

```python
def test_user_cannot_read_another_users_trace(tmp_path, monkeypatch):
    import json
    import app.main as main

    trace_id = "123e4567-e89b-12d3-a456-426614174000"

    monkeypatch.setattr(main, "TRACE_DIRECTORY", tmp_path)
    monkeypatch.setenv(
        "DEMO_API_TOKENS",
        "dev-alice-token:alice,dev-bob-token:bob",
    )

    (tmp_path / f"{trace_id}.json").write_text(
        json.dumps({
            "trace_id": trace_id,
            "owner_id": "alice",
            "events": [],
        }),
        encoding="utf-8",
    )

    response = client.get(
        f"/traces/{trace_id}",
        headers={"Authorization": "Bearer dev-bob-token"},
    )

    assert response.status_code == 403
```

## Program 3 — Add a protected delete operation

Add a safe example route:

```python
from fastapi import status


@app.delete("/traces/{trace_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_trace(
    trace_id: str,
    current_user: CurrentUser = Depends(get_current_user),
) -> None:
    try:
        safe_trace_id = UUID(trace_id)
    except ValueError as error:
        raise HTTPException(
            status_code=400,
            detail="Invalid trace ID",
        ) from error

    trace_path = TRACE_DIRECTORY / f"{safe_trace_id}.json"

    if not trace_path.is_file():
        raise HTTPException(status_code=404, detail="Trace not found")

    trace_data = json.loads(trace_path.read_text(encoding="utf-8"))

    if trace_data.get("owner_id") != current_user.user_id:
        raise HTTPException(
            status_code=403,
            detail="You cannot delete this trace",
        )

    trace_path.unlink()
```

Real-company improvement: before a destructive action, require explicit confirmation/approval and write an audit event containing user ID, action, resource ID, result, and request ID.

## Day 22 Assignment

Build and test all of these:

1. `/assist` requires a valid Bearer token.
2. Alice can read Alice’s trace.
3. Bob cannot read Alice’s trace.
4. Only the trace owner can delete their trace.
5. Add a README section called `Authentication` with this example:

```text
Authorization: Bearer dev-alice-token
```

Never put real tokens in Git, screenshots, client-side code, logs, or error responses.

# Day 22 AI Quiz

1. What is authentication?  
   A. Checking what a user is allowed to do  
   B. Confirming who is making the request  
   C. Deleting old traces  
   D. Formatting JSON  

2. What is authorization?  
   A. Checking permissions after identity is known  
   B. Creating a Python virtual environment  
   C. Starting Uvicorn  
   D. Encrypting a README  

3. Which header carries a Bearer token?  
   A. `Content-Type`  
   B. `X-Trace-ID`  
   C. `Authorization`  
   D. `Accept-Language`  

4. What does HTTP `401` mean?  
   A. The request succeeded  
   B. The client is not authenticated  
   C. The resource was deleted  
   D. The server crashed  

5. What does HTTP `403` mean?  
   A. The user is authenticated but not allowed  
   B. No API route exists  
   C. Validation failed  
   D. The server restarted  

6. Why should `user_id` not be trusted from request JSON?  
   A. JSON cannot hold text  
   B. The client could claim another user’s identity  
   C. FastAPI removes it  
   D. UUIDs do not support it  

7. Where should user identity come from?  
   A. A browser button label  
   B. A server-validated token/session  
   C. The expense title  
   D. A query parameter only  

8. Which status is suitable when an authenticated user tries to read another user’s trace?  
   A. 200  
   B. 201  
   C. 403  
   D. 500  

9. Why use `secrets.compare_digest()` for tokens?  
   A. It formats JSON  
   B. It compares secrets more safely  
   C. It creates JWT tokens  
   D. It starts the API  

10. Which item must never be committed to Git?  
   A. `.env` containing real secrets  
   B. `README.md`  
   C. `tests/`  
   D. `.gitignore`  

11. What is tenant isolation?  
   A. Separating each user or organization’s data  
   B. Creating more API routes  
   C. Sorting expenses  
   D. Converting a list to JSON  

12. Before deleting a user-owned resource, the API should verify:  
   A. Browser theme  
   B. Ownership and authorization  
   C. Python version  
   D. Internet speed  

13. What should a production audit log record?  
   A. Passwords and API keys  
   B. Only frontend color settings  
   C. Identity, action, resource, result, and request ID  
   D. The entire database  

14. Where must authorization be enforced for a tool that deletes data?  
   A. Only in the frontend  
   B. At the API/tool boundary before the side effect  
   C. Only in the prompt  
   D. Only in a README  

15. What is the safe overall flow?  
   A. Request → run any tool immediately  
   B. Request → trust user ID from JSON → return data  
   C. Authenticate → authorize → validate → run trusted workflow → audit  
   D. Request → expose stack trace → retry  

Reply like this:

```text
1.B
2.A
3.C
...
15.C
```
