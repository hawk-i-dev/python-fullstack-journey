# Day 24 AI — Stream Agent Responses

Today, your API will send the response gradually instead of waiting for the complete answer.

```text
Normal API:   User waits → full response arrives
Streaming:    User sees → “Your” → “budget” → “remaining…” → done
```

This is how chat applications feel responsive. OpenAI streaming uses typed server-sent events; the key text event is `response.output_text.delta`. [Official OpenAI streaming guide](https://developers.openai.com/api/docs/guides/streaming-responses)

## Today’s outcome

Build:

```text
POST /assist/stream
```

The route will:

1. Authenticate the user.
2. Verify conversation ownership.
3. Save the user’s message.
4. Stream assistant text chunks.
5. Save the final assistant response only after completion.

## Core concepts

| Concept | Meaning |
|---|---|
| `async` | A function that can pause without blocking the server |
| async generator | Uses `yield` to send many values over time |
| SSE | Server-Sent Events: server sends incremental events to client |
| delta | A small piece of generated text |
| final response | Combined text saved after streaming completes |

## 1. Create `app/streaming.py`

This version is a safe local demo. It does not require an API key.

```python
import asyncio
from collections.abc import AsyncGenerator


async def demo_agent_stream(question: str) -> AsyncGenerator[str, None]:
    response = (
        f"I received your question: {question}. "
        "Your streaming AI assistant is working correctly."
    )

    for word in response.split():
        await asyncio.sleep(0.08)
        yield f"{word} "
```

## 2. Add an SSE helper

Add this in `app/main.py`:

```python
import json


def sse_event(event_name: str, data: dict) -> str:
    return (
        f"event: {event_name}\n"
        f"data: {json.dumps(data)}\n\n"
    )
```

Each event sent to the client will look like:

```text
event: token
data: {"text": "Hello "}
```

## 3. Add the streaming endpoint

Import:

```python
from collections.abc import AsyncGenerator

from fastapi.responses import StreamingResponse

from app.streaming import demo_agent_stream
```

Add this route:

```python
@app.post("/assist/stream")
async def stream_assist(
    request: AssistRequest,
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> StreamingResponse:
    conversation = get_owned_conversation(
        db=db,
        conversation_id=request.conversation_id,
        owner_id=current_user.user_id,
    )

    save_message(
        db=db,
        conversation_id=conversation.id,
        role="user",
        content=request.question,
    )

    async def generate_events() -> AsyncGenerator[str, None]:
        complete_answer: list[str] = []

        yield sse_event(
            "status",
            {"message": "Agent started"},
        )

        try:
            async for text_chunk in demo_agent_stream(request.question):
                complete_answer.append(text_chunk)

                yield sse_event(
                    "token",
                    {"text": text_chunk},
                )

            final_answer = "".join(complete_answer)

            save_message(
                db=db,
                conversation_id=conversation.id,
                role="assistant",
                content=final_answer,
            )

            yield sse_event(
                "done",
                {
                    "conversation_id": conversation.id,
                    "message": "Response saved",
                },
            )

        except Exception:
            yield sse_event(
                "error",
                {
                    "message": "The agent could not complete the request",
                },
            )

    return StreamingResponse(
        generate_events(),
        media_type="text/event-stream",
    )
```

## 4. Test from the terminal

First create a conversation in Swagger and copy its ID.

### Mac

```bash
curl -N \
  -X POST "http://127.0.0.1:8000/assist/stream" \
  -H "Authorization: Bearer dev-alice-token" \
  -H "Content-Type: application/json" \
  -d '{"question":"How much can I save?","conversation_id":"YOUR_CONVERSATION_ID"}'
```

### Windows PowerShell

Use `curl.exe`, not `curl`, because PowerShell may map `curl` to another command.

```powershell
curl.exe -N `
  -X POST "http://127.0.0.1:8000/assist/stream" `
  -H "Authorization: Bearer dev-alice-token" `
  -H "Content-Type: application/json" `
  -d "{\"question\":\"How much can I save?\",\"conversation_id\":\"YOUR_CONVERSATION_ID\"}"
```

You should see events arrive one by one:

```text
event: status
data: {"message": "Agent started"}

event: token
data: {"text": "I "}

event: token
data: {"text": "received "}

event: done
data: {"conversation_id": "...", "message": "Response saved"}
```

## 5. Real LLM streaming pattern

Later, replace `demo_agent_stream()` with an OpenAI adapter using your `.env` model setting:

```python
import os

from openai import AsyncOpenAI


async def openai_agent_stream(
    input_messages: list[dict[str, str]],
):
    client = AsyncOpenAI()

    stream = await client.responses.create(
        model=os.environ["OPENAI_MODEL"],
        input=input_messages,
        stream=True,
    )

    async for event in stream:
        if event.type == "response.output_text.delta":
            yield event.delta
```

Do not put your API key in frontend JavaScript. The browser talks only to your FastAPI backend; your backend holds secrets.

## Assignment

Complete these checks:

1. Alice streams a response successfully.
2. Bob cannot stream using Alice’s conversation ID.
3. The final assistant response appears in `GET /conversations/{id}/messages`.
4. Simulate an invalid conversation ID and confirm the API returns `404`.
5. Add a README section: `Streaming Events` with `status`, `token`, `done`, and `error`.

## Real-world mindset

Streaming changes the user experience, but it adds failure cases:

```text
User disconnects midway
Model errors after five tokens
Tool needs approval during generation
Database save fails after output is visible
```

For now, save the assistant message only on successful completion. In later production work, we will introduce message statuses such as `streaming`, `completed`, `failed`, and `cancelled`.

When you finish, ask for **Day 24 AI quiz and programs**.
