## Day 21 AI — Quiz and Programs

Today you make your agent usable by a frontend, mobile app, or client system through a FastAPI backend.

Core idea: the API validates untrusted input, calls only your trusted agent workflow, and returns safe JSON. In a real company, your application owns the tool execution, storage, approvals, and runtime integration. [OpenAI Agents guide](https://developers.openai.com/api/docs/guides/agents)

### Run the project

Same on Mac and Windows after activating the virtual environment:

```bash
python -m pip install fastapi "uvicorn[standard]"
python -m uvicorn app.main:app --reload
```

Open Swagger UI:

```text
http://127.0.0.1:8000/docs
```

### Program 1 — Securely read an existing trace

Add this to `app/main.py`:

```python
import json
from pathlib import Path
from typing import Any
from uuid import UUID

from fastapi import HTTPException

TRACE_DIRECTORY = Path("data/traces")


@app.get("/traces/{trace_id}")
def get_trace(trace_id: str) -> dict[str, Any]:
    try:
        safe_trace_id = UUID(trace_id)
    except ValueError as error:
        raise HTTPException(status_code=400, detail="Invalid trace ID") from error

    trace_path = TRACE_DIRECTORY / f"{safe_trace_id}.json"

    if not trace_path.is_file():
        raise HTTPException(status_code=404, detail="Trace not found")

    return json.loads(trace_path.read_text(encoding="utf-8"))
```

Why UUID validation matters: it blocks invalid values such as `../../secret-file`. Never allow a user to decide an arbitrary server file path.

### Program 2 — Add a request ID to every API response

Add this to `app/main.py`:

```python
from uuid import uuid4

from fastapi import Request


@app.middleware("http")
async def add_request_id(request: Request, call_next):
    request_id = str(uuid4())

    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id

    return response
```

This helps corporate debugging. If a client reports a problem, they can share the request ID and you can locate its logs or trace.

### Program 3 — Test the trace endpoint

Create `tests/test_traces_api.py`:

```python
import json

from fastapi.testclient import TestClient

import app.main as main

client = TestClient(main.app)


def test_get_trace_returns_saved_trace(tmp_path, monkeypatch):
    trace_id = "123e4567-e89b-12d3-a456-426614174000"

    monkeypatch.setattr(main, "TRACE_DIRECTORY", tmp_path)

    saved_trace = {
        "trace_id": trace_id,
        "events": [{"name": "workflow_started"}],
    }

    (tmp_path / f"{trace_id}.json").write_text(
        json.dumps(saved_trace),
        encoding="utf-8",
    )

    response = client.get(f"/traces/{trace_id}")

    assert response.status_code == 200
    assert response.json()["trace_id"] == trace_id


def test_get_trace_returns_404_when_missing(tmp_path, monkeypatch):
    monkeypatch.setattr(main, "TRACE_DIRECTORY", tmp_path)

    response = client.get(
        "/traces/123e4567-e89b-12d3-a456-426614174000"
    )

    assert response.status_code == 404
```

Run:

```bash
pytest -q
```

### Assignment

Add `GET /traces/{trace_id}` to your Day 20 agent API, test these three cases, and commit the work:

1. Valid trace returns `200`.
2. Missing trace returns `404`.
3. Invalid trace ID returns `400`.

Real-world rule: return safe errors to the client, but log the detailed internal error on the server. Never return API keys, stack traces, passwords, or private document content.

## Day 21 Quiz

1. What is an API endpoint?  
   A. A database table  
   B. A URL where an application accepts or returns data  
   C. A Python virtual environment  
   D. A Git branch  

2. Why do we use Pydantic request models?  
   A. To validate incoming data  
   B. To deploy the server  
   C. To store API keys  
   D. To replace pytest  

3. What URL opens FastAPI’s Swagger UI by default?  
   A. `/health`  
   B. `/docs`  
   C. `/trace`  
   D. `/admin`  

4. Which HTTP method is normally used to read a resource?  
   A. DELETE  
   B. POST  
   C. GET  
   D. PATCH  

5. Which HTTP method is suitable for submitting an agent question?  
   A. GET  
   B. POST  
   C. DELETE  
   D. HEAD  

6. A request with `spent: -100` fails Pydantic validation. What status code is normally returned?  
   A. 200  
   B. 422  
   C. 404  
   D. 500  

7. What should the API return when an unexpected internal exception occurs?  
   A. A safe generic error; detailed error only in server logs  
   B. API key and stack trace  
   C. Database password  
   D. Nothing  

8. What does Uvicorn’s `--reload` option do during development?  
   A. Deletes the database  
   B. Restarts the server after code changes  
   C. Encrypts responses  
   D. Creates a new API key  

9. Why validate `trace_id` as a UUID?  
   A. It makes JSON faster  
   B. It helps prevent invalid/path-traversal file access  
   C. It creates a database table  
   D. It replaces authentication  

10. What does FastAPI `TestClient` let you do?  
   A. Test routes without manually running a browser/server  
   B. Deploy to production  
   C. Create an LLM  
   D. Replace Uvicorn  

11. Which layer should validate incoming user data?  
   A. API boundary  
   B. Browser cache only  
   C. Database backup  
   D. GitHub README  

12. What is a health endpoint mainly for?  
   A. Checking whether the service is available  
   B. Creating expenses  
   C. Training the model  
   D. Deleting logs  

13. What does HTTP `500` usually mean?  
   A. User is unauthenticated  
   B. Resource was not found  
   C. Unexpected server-side failure  
   D. Input validation failed  

14. What format should a REST API commonly return to a frontend?  
   A. PDF only  
   B. JSON  
   C. Python source code  
   D. CSV only  

15. What is the safest high-level API flow?  
   A. User input → directly run any tool  
   B. User input → database password → response  
   C. Validate → trusted workflow → safe response and logs  
   D. Browser → delete all traces  

Reply in this format:

```text
1.B
2.A
...
15.C
```
