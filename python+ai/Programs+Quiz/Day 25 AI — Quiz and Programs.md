# Day 25 AI — Quiz and Programs

Your frontend consumes the streamed response through `fetch()` and `ReadableStream`. Use `fetch`, not browser `EventSource`, because this project sends a `POST` body and an `Authorization` header.

Streaming output reaches the browser incrementally, so production applications must also handle partial output, errors, disconnects, and moderation carefully. [Official OpenAI streaming guidance](https://developers.openai.com/api/docs/guides/streaming-responses)

## Program 1 — Extract a reusable SSE parser

Create `frontend/src/sse.ts`:

```typescript
export type ParsedSseEvent = {
  name: string;
  data: Record<string, unknown>;
};

export function parseSseFrames(buffer: string): {
  events: ParsedSseEvent[];
  remaining: string;
} {
  const frames = buffer.split("\n\n");
  const remaining = frames.pop() ?? "";

  const events: ParsedSseEvent[] = [];

  for (const frame of frames) {
    let name = "message";
    let dataText = "";

    for (const line of frame.split("\n")) {
      if (line.startsWith("event: ")) {
        name = line.removeprefix("event: ");
      }

      if (line.startsWith("data: ")) {
        dataText = line.removeprefix("data: ");
      }
    }

    if (dataText) {
      events.push({
        name,
        data: JSON.parse(dataText) as Record<string, unknown>,
      });
    }
  }

  return { events, remaining };
}
```

Use it inside `streamAssistant()`:

```typescript
import { parseSseFrames } from "./sse";

// Inside the while loop:
buffer += decoder.decode(value, { stream: true });

const parsed = parseSseFrames(buffer);
buffer = parsed.remaining;

for (const event of parsed.events) {
  onEvent(event.name, event.data);
}
```

Why this matters: a network chunk is not guaranteed to contain one complete SSE event. It may contain half an event or several events.

## Program 2 — Add a “New conversation” button

In `App.tsx`, add this function:

```tsx
function startNewConversation() {
  if (streaming) {
    return;
  }

  setConversationId("");
  setMessages([]);
  setError("");
}
```

Add this button above the message section:

```tsx
<button
  type="button"
  onClick={startNewConversation}
  disabled={streaming}
>
  New conversation
</button>
```

Expected behavior:

```text
Click New conversation
→ clear existing visible messages
→ clear conversation ID
→ next submitted question creates a new database conversation
```

Do not delete the old conversation from the database. A user should still be able to access their history later.

## Program 3 — Test CORS configuration

Create `tests/test_cors.py`:

```python
from fastapi.testclient import TestClient

from app.main import app


client = TestClient(app)


def test_frontend_origin_is_allowed():
    response = client.options(
        "/assist/stream",
        headers={
            "Origin": "http://localhost:5173",
            "Access-Control-Request-Method": "POST",
        },
    )

    assert response.status_code == 200
    assert response.headers["access-control-allow-origin"] == (
        "http://localhost:5173"
    )
```

Run:

```bash
pytest -q
```

If this test fails, check your FastAPI middleware:

```python
allow_origins=[
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]
```

# Day 25 Quiz

1. Why is CORS needed in this project?  
   A. React and FastAPI run on different origins/ports  
   B. To create a database  
   C. To store API keys  
   D. To replace authentication  

2. Which frontend API reads streamed HTTP response bytes?  
   A. `localStorage`  
   B. `ReadableStream` through `response.body`  
   C. `alert()`  
   D. `JSON.stringify()`  

3. Why is `fetch()` useful for this streaming endpoint?  
   A. It supports a POST body and Authorization header  
   B. It removes CORS automatically  
   C. It stores OpenAI keys  
   D. It replaces FastAPI  

4. Why should the frontend not contain `OPENAI_API_KEY`?  
   A. It is too long  
   B. Browser users can inspect frontend code  
   C. React cannot use variables  
   D. SQLite requires it  

5. Which backend header carries the user’s access token?  
   A. `Content-Type`  
   B. `Authorization`  
   C. `Origin`  
   D. `Accept`  

6. What does `TextDecoder` do in streaming code?  
   A. Converts received bytes into text  
   B. Encrypts tokens  
   C. Creates a database  
   D. Starts Vite  

7. Why keep a `buffer` when parsing SSE?  
   A. A network chunk may contain incomplete event data  
   B. It makes authentication unnecessary  
   C. It stores passwords  
   D. It replaces React state  

8. What should happen when a `token` event arrives?  
   A. Append its text to the current assistant message  
   B. Delete the conversation  
   C. Create a new user  
   D. Reload the page  

9. What should happen when an `error` event arrives?  
   A. Show a safe error message  
   B. Display backend stack trace  
   C. Expose environment variables  
   D. Delete all messages  

10. What does a “New conversation” button normally do?  
   A. Clears current UI state for a new chat  
   B. Deletes every database record  
   C. Removes the backend  
   D. Reveals API keys  

11. Which origin is commonly used by a Vite development server?  
   A. `http://localhost:5173`  
   B. `http://localhost:8000`  
   C. `http://localhost:3306`  
   D. `http://localhost:5432`  

12. What must the backend verify before streaming a conversation response?  
   A. User authentication and conversation ownership  
   B. User’s browser color  
   C. Git branch name  
   D. Frontend font choice  

13. Where should the final assistant response be stored?  
   A. Only in React state  
   B. In the backend database after successful completion  
   C. In browser console only  
   D. In `.env`  

14. Why should production CORS avoid `allow_origins=["*"]` with authentication?  
   A. It can allow untrusted websites to call the API  
   B. It makes CSS slow  
   C. It deletes SQLite  
   D. It disables React  

15. What is the correct end-to-end request flow?  
   A. Browser → OpenAI API key → model  
   B. React → FastAPI authentication/authorization → agent stream → React UI  
   C. React → database directly → model  
   D. Browser → `.env` → response  

Reply in this format:

```text
1.A
2.B
3.A
...
15.B
```
