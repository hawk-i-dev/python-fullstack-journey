# Day 25 AI — Build the Streaming Chat Frontend

Today you connect your FastAPI AI backend to a React web interface.

By end of day, your application flow will be:

```text
React chat UI
   → Bearer token + question + conversation ID
   → FastAPI /assist/stream
   → streamed SSE events
   → text appears live in the browser
   → final response is stored in SQLite
```

Streaming makes the product feel responsive because the user sees generated text while the response is still being created. [Official OpenAI streaming guide](https://developers.openai.com/api/docs/guides/streaming-responses)

## 1. Enable CORS in FastAPI

Your frontend runs on port `5173`; FastAPI runs on port `8000`. The backend must explicitly allow the frontend origin.

In `app/main.py`, add:

```python
from fastapi.middleware.cors import CORSMiddleware
```

After creating `app`, add:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",
        "http://127.0.0.1:5173",
    ],
    allow_credentials=False,
    allow_methods=["GET", "POST", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

Do not use `allow_origins=["*"]` in a real client project with authentication.

## 2. Create the React project

Use this only if you do not already have a React frontend folder:

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install
npm run dev
```

Mac and Windows commands are the same.

## 3. Create `frontend/src/api.ts`

```typescript
const API_URL = "http://127.0.0.1:8000";

type StreamData = Record<string, unknown>;

export async function createConversation(
  apiToken: string,
  title: string,
): Promise<string> {
  const response = await fetch(`${API_URL}/conversations`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${apiToken}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ title }),
  });

  if (!response.ok) {
    throw new Error("Could not create conversation");
  }

  const conversation = (await response.json()) as { id: string };
  return conversation.id;
}

export async function streamAssistant(
  apiToken: string,
  question: string,
  conversationId: string,
  onEvent: (eventName: string, data: StreamData) => void,
): Promise<void> {
  const response = await fetch(`${API_URL}/assist/stream`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${apiToken}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      question,
      conversation_id: conversationId,
    }),
  });

  if (!response.ok || !response.body) {
    throw new Error("Could not start AI stream");
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  let buffer = "";

  while (true) {
    const { done, value } = await reader.read();

    if (done) {
      break;
    }

    buffer += decoder.decode(value, { stream: true });

    const rawEvents = buffer.split("\n\n");
    buffer = rawEvents.pop() ?? "";

    for (const rawEvent of rawEvents) {
      let eventName = "message";
      let dataText = "";

      for (const line of rawEvent.split("\n")) {
        if (line.startsWith("event: ")) {
          eventName = line.removeprefix("event: ");
        }

        if (line.startsWith("data: ")) {
          dataText = line.removeprefix("data: ");
        }
      }

      if (dataText) {
        onEvent(eventName, JSON.parse(dataText) as StreamData);
      }
    }
  }
}
```

## 4. Replace `frontend/src/App.tsx`

```tsx
import { FormEvent, useState } from "react";

import {
  createConversation,
  streamAssistant,
} from "./api";

type ChatMessage = {
  id: string;
  role: "user" | "assistant";
  content: string;
};

export default function App() {
  const [apiToken, setApiToken] = useState("");
  const [conversationId, setConversationId] = useState("");
  const [draft, setDraft] = useState("");
  const [streaming, setStreaming] = useState(false);
  const [error, setError] = useState("");
  const [messages, setMessages] = useState<ChatMessage[]>([]);

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    const question = draft.trim();

    if (!apiToken.trim() || !question || streaming) {
      return;
    }

    setError("");
    setStreaming(true);
    setDraft("");

    try {
      let activeConversationId = conversationId;

      if (!activeConversationId) {
        activeConversationId = await createConversation(
          apiToken,
          "New Finance Chat",
        );

        setConversationId(activeConversationId);
      }

      const assistantMessageId = crypto.randomUUID();

      setMessages((current) => [
        ...current,
        {
          id: crypto.randomUUID(),
          role: "user",
          content: question,
        },
        {
          id: assistantMessageId,
          role: "assistant",
          content: "",
        },
      ]);

      await streamAssistant(
        apiToken,
        question,
        activeConversationId,
        (eventName, data) => {
          if (eventName === "token") {
            const text = String(data.text ?? "");

            setMessages((current) =>
              current.map((message) =>
                message.id === assistantMessageId
                  ? {
                      ...message,
                      content: message.content + text,
                    }
                  : message,
              ),
            );
          }

          if (eventName === "error") {
            setError(String(data.message ?? "Stream failed"));
          }
        },
      );
    } catch (err) {
      setError(
        err instanceof Error
          ? err.message
          : "Something went wrong",
      );
    } finally {
      setStreaming(false);
    }
  }

  return (
    <main>
      <h1>Finance AI Assistant</h1>

      <label>
        Development API token
        <input
          type="password"
          value={apiToken}
          onChange={(event) => setApiToken(event.target.value)}
          placeholder="dev-alice-token"
        />
      </label>

      {error && <p role="alert">{error}</p>}

      <section>
        {messages.map((message) => (
          <article key={message.id}>
            <strong>
              {message.role === "user" ? "You" : "Assistant"}:
            </strong>{" "}
            {message.content || "Thinking..."}
          </article>
        ))}
      </section>

      <form onSubmit={handleSubmit}>
        <input
          value={draft}
          onChange={(event) => setDraft(event.target.value)}
          placeholder="Ask about your budget..."
          disabled={streaming}
        />

        <button disabled={streaming || !draft.trim()}>
          {streaming ? "Thinking..." : "Send"}
        </button>
      </form>
    </main>
  );
}
```

## 5. Run both applications

Terminal 1 — backend:

```bash
python -m uvicorn app.main:app --reload
```

Terminal 2 — frontend:

```bash
cd frontend
npm run dev
```

Open:

```text
http://localhost:5173
```

Enter:

```text
dev-alice-token
```

Then ask a question.

## Important security rule

The development token is only for local practice. A real app uses login and short-lived access tokens.

Never put this in React code, `.env` files exposed to Vite, or browser JavaScript:

```text
OPENAI_API_KEY
```

Your browser calls FastAPI. Only FastAPI calls the LLM provider.

## Assignment

Complete these before Day 26:

1. Display user and assistant messages with different colors.
2. Add a “New conversation” button that clears messages and conversation ID.
3. Display “Thinking…” while streaming.
4. Confirm the final streamed response exists in your SQLite conversation messages.
5. Try an invalid token and display the `401` error cleanly.

This is your first end-to-end AI product: frontend, backend, streaming, user isolation, and durable conversation memory.
