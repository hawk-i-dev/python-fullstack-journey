# Day 24 AI — Quiz and Programs

Streaming sends small text pieces while the agent is generating, so the user sees progress immediately. With the Responses API, you normally handle typed events such as `response.output_text.delta`, `response.completed`, and `error`. [Official OpenAI streaming guide](https://developers.openai.com/api/docs/guides/streaming-responses)

## Program 1 — Test SSE event formatting

Create `tests/test_sse.py`:

```python
import json

from app.main import sse_event


def test_sse_event_has_correct_format():
    event = sse_event(
        "token",
        {"text": "Hello"},
    )

    lines = event.splitlines()

    assert lines[0] == "event: token"
    assert lines[1].startswith("data: ")

    payload = json.loads(lines[1].removeprefix("data: "))
    assert payload == {"text": "Hello"}
```

SSE format must end with two newlines:

```text
event: token
data: {"text": "Hello"}

```

## Program 2 — Test the async streaming generator

Create `tests/test_streaming.py`:

```python
import asyncio

from app.streaming import demo_agent_stream


async def collect_stream(question: str) -> str:
    chunks: list[str] = []

    async for chunk in demo_agent_stream(question):
        chunks.append(chunk)

    return "".join(chunks)


def test_demo_agent_stream_returns_complete_text():
    result = asyncio.run(
        collect_stream("How much can I save?")
    )

    assert "How much can I save?" in result
    assert "streaming AI assistant" in result
```

Run:

```bash
pytest -q
```

Important: the frontend receives chunks one by one, but the database should store the final combined assistant message.

## Program 3 — Test a streaming API response

Create `tests/test_stream_response.py`:

```python
from collections.abc import AsyncGenerator

from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from fastapi.testclient import TestClient


app = FastAPI()


async def events() -> AsyncGenerator[str, None]:
    yield "event: token\ndata: {\"text\": \"Hello \"}\n\n"
    yield "event: token\ndata: {\"text\": \"world\"}\n\n"
    yield "event: done\ndata: {\"ok\": true}\n\n"


@app.get("/demo-stream")
async def demo_stream() -> StreamingResponse:
    return StreamingResponse(
        events(),
        media_type="text/event-stream",
    )


def test_stream_response_contains_events():
    client = TestClient(app)

    with client.stream("GET", "/demo-stream") as response:
        body = "".join(response.iter_text())

    assert response.status_code == 200
    assert response.headers["content-type"].startswith(
        "text/event-stream"
    )
    assert "event: token" in body
    assert "event: done" in body
    assert "Hello" in body
```

## Production streaming rules

```text
status → token → token → token → done
                         └── error, if generation fails
```

- Save user message before streaming begins.
- Collect all valid text chunks.
- Save assistant message only after successful completion.
- Never send API keys, internal stack traces, database errors, or private tool details to the browser.
- Keep authentication and conversation ownership checks before streaming starts.

# Day 24 Quiz

1. Why do we stream an AI response?  
   A. To avoid authentication  
   B. To show output before the full response is ready  
   C. To remove the database  
   D. To hide errors  

2. What does SSE mean?  
   A. Server-Sent Events  
   B. Secure SQL Engine  
   C. Static Server Error  
   D. Stream Storage Entry  

3. Which FastAPI response class is used to return a stream?  
   A. `JSONResponse`  
   B. `FileResponse`  
   C. `StreamingResponse`  
   D. `HTMLResponse`  

4. What does an async generator use to send multiple values over time?  
   A. `return` only  
   B. `yield`  
   C. `print`  
   D. `raise`  

5. What media type is commonly used for SSE?  
   A. `application/pdf`  
   B. `application/json`  
   C. `text/event-stream`  
   D. `image/png`  

6. Which event contains incremental generated text in the Responses API?  
   A. `response.output_text.delta`  
   B. `response.database.saved`  
   C. `response.user.login`  
   D. `response.server.start`  

7. What should the API do before streaming a response for a conversation?  
   A. Trust any conversation ID  
   B. Verify authentication and conversation ownership  
   C. Delete existing messages  
   D. Return all users’ history  

8. What should normally be saved to the database after a successful stream?  
   A. Every API key  
   B. Only the first token  
   C. The complete combined assistant answer  
   D. The user’s password  

9. Why collect chunks in `complete_answer`?  
   A. To create the final message to persist  
   B. To bypass authentication  
   C. To create SQL tables  
   D. To stop Uvicorn  

10. What should an API send if generation fails while streaming?  
   A. An `error` event with a safe message  
   B. The server stack trace  
   C. The `.env` file  
   D. Another user’s response  

11. What does `await` help with?  
   A. Pausing an async function without blocking all work  
   B. Deleting a database  
   C. Creating a Git branch  
   D. Encrypting JSON  

12. Which event should tell the client the stream completed?  
   A. `status`  
   B. `token`  
   C. `done`  
   D. `loading`  

13. Where should the OpenAI API key exist?  
   A. Browser JavaScript  
   B. Public GitHub repository  
   C. Backend environment variables  
   D. SSE token event  

14. If a user disconnects before completion, what is the safest beginner approach?  
   A. Save the partial text as complete  
   B. Do not save it as a completed assistant message  
   C. Expose internal errors  
   D. Delete all conversations  

15. What is the correct secure stream flow?  
   A. Stream first, check ownership later  
   B. Authenticate → authorize → save user message → stream → save completed response  
   C. Send the API key to the browser → stream  
   D. Load every user’s history → stream  

Reply like this:

```text
1.B
2.A
3.C
...
15.B
```
