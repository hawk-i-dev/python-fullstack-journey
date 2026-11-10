# Day 29 AI — User Feedback → Better Agent Quality

Today we add a professional feedback loop to your AI app.

A 👍 / 👎 is not just a UI button. It helps you find real failures, review them, and convert them into test cases so the agent improves over time. This is how AI products become reliable. OpenAI recommends evaluating agent workflows with representative cases and regression testing rather than trusting subjective “it seems good” checks. [OpenAI agent evaluation guide](https://developers.openai.com/api/docs/guides/agent-evals)

## Today’s outcome

By end of today, your app will let a user:

1. Rate an assistant answer as helpful or unhelpful.
2. Optionally select why it was unhelpful.
3. Store feedback securely in SQLite.
4. Ensure a user cannot rate another user’s conversation/message.
5. Turn reviewed bad answers into future evaluation cases.

## The real-world feedback loop

```text
User gets an answer
        ↓
User gives 👍 or 👎
        ↓
Store rating + category + optional comment
        ↓
Developer reviews the answer, trace, retrieved documents, and tool calls
        ↓
Create a clean regression test case
        ↓
Run it before future releases
```

Important: a 👎 is a signal, not proof that the AI was wrong. The user may be unhappy even if the answer was correct. A human review decides whether it becomes an evaluation case.

## Step 1 — Add the `Feedback` database model

In `backend/app/models.py`, add imports if missing:

```python
from sqlalchemy import DateTime, ForeignKey, Integer, String, Text, UniqueConstraint
from sqlalchemy.orm import Mapped, mapped_column
from datetime import datetime, timezone
```

Add this model:

```python
class Feedback(Base):
    __tablename__ = "feedback"
    __table_args__ = (
        UniqueConstraint("user_id", "message_id", name="uq_feedback_user_message"),
    )

    id: Mapped[int] = mapped_column(primary_key=True)

    user_id: Mapped[str] = mapped_column(String(100), index=True)
    message_id: Mapped[int] = mapped_column(
        ForeignKey("messages.id"),
        index=True,
    )

    # 1 = helpful, -1 = unhelpful
    rating: Mapped[int] = mapped_column(Integer)

    # Example: incorrect, irrelevant, unsafe, slow, other
    category: Mapped[str | None] = mapped_column(String(50), nullable=True)
    comment: Mapped[str | None] = mapped_column(Text, nullable=True)

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
    )
```

For development, your existing `Base.metadata.create_all()` will create this new table when the backend restarts.

## Step 2 — Create the feedback request schema

In `backend/app/schemas.py`:

```python
from typing import Literal
from pydantic import BaseModel, Field


class FeedbackCreate(BaseModel):
    message_id: int
    rating: Literal[-1, 1]
    category: Literal[
        "incorrect",
        "irrelevant",
        "unsafe",
        "slow",
        "other",
    ] | None = None
    comment: str | None = Field(default=None, max_length=500)
```

Why use `Literal[-1, 1]`?

- `1` means helpful.
- `-1` means unhelpful.
- Invalid values such as `0`, `4`, or `"good"` are rejected automatically with HTTP `422`.

## Step 3 — Add a protected feedback API

Create `backend/app/routes/feedback.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.auth import CurrentUser, get_current_user
from app.database import get_db
from app.models import Feedback, Message
from app.schemas import FeedbackCreate

router = APIRouter(prefix="/feedback", tags=["Feedback"])


@router.post("", status_code=status.HTTP_201_CREATED)
def create_feedback(
    payload: FeedbackCreate,
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    message = db.get(Message, payload.message_id)

    if message is None:
        raise HTTPException(status_code=404, detail="Message not found")

    # Important: verify message belongs to a conversation owned by this user.
    conversation = message.conversation

    if conversation.owner_id != current_user.user_id:
        raise HTTPException(status_code=403, detail="Not allowed")

    if message.role != "assistant":
        raise HTTPException(
            status_code=400,
            detail="Feedback can only be submitted for assistant messages",
        )

    existing = db.scalar(
        select(Feedback).where(
            Feedback.user_id == current_user.user_id,
            Feedback.message_id == payload.message_id,
        )
    )

    if existing:
        existing.rating = payload.rating
        existing.category = payload.category
        existing.comment = payload.comment
        db.commit()
        db.refresh(existing)
        return existing

    feedback = Feedback(
        user_id=current_user.user_id,
        message_id=payload.message_id,
        rating=payload.rating,
        category=payload.category,
        comment=payload.comment,
    )

    db.add(feedback)
    db.commit()
    db.refresh(feedback)

    return feedback
```

Then register it in `backend/app/main.py`:

```python
from app.routes import feedback
```

```python
app.include_router(feedback.router)
```

## Step 4 — Send the assistant message ID to React

Your frontend can only submit feedback if it knows the database ID of the saved assistant message.

In your streaming endpoint, after saving the final assistant message:

```python
assistant_message = save_message(
    db=db,
    conversation_id=conversation_id,
    role="assistant",
    content=full_answer,
)
```

Send its ID in the final SSE event:

```python
yield sse_event(
    "done",
    {
        "conversation_id": conversation_id,
        "message_id": assistant_message.id,
    },
)
```

Do not expose database IDs for other users’ conversations. Your backend authorization check remains the real security protection.

## Step 5 — Add the frontend API function

In `frontend/src/api.ts`:

```ts
export async function createFeedback(
  token: string,
  payload: {
    message_id: number;
    rating: 1 | -1;
    category?: string;
    comment?: string;
  },
) {
  const response = await fetch(`${API_URL}/feedback`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(payload),
  });

  if (!response.ok) {
    throw new Error("Could not save feedback");
  }

  return response.json();
}
```

Add buttons below every completed assistant message:

```tsx
<button
  type="button"
  onClick={() =>
    void createFeedback(token, {
      message_id: message.dbId!,
      rating: 1,
    })
  }
>
  👍 Helpful
</button>

<button
  type="button"
  onClick={() =>
    void createFeedback(token, {
      message_id: message.dbId!,
      rating: -1,
      category: "incorrect",
    })
  }
>
  👎 Not helpful
</button>
```

Only show these buttons when `message.dbId` exists. That means the assistant response was completed and saved successfully.

## Corporate mindset

Use clear categories. Avoid a vague “bad answer” bucket.

| Category | Meaning |
|---|---|
| `incorrect` | Factually wrong or made up information |
| `irrelevant` | Did not answer the user’s question |
| `unsafe` | Should have been blocked or escalated |
| `slow` | Response or tool execution took too long |
| `other` | A problem not covered above |

Never automatically change your prompt, RAG documents, or model based on raw user feedback. First review:

- User question
- AI answer
- Retrieved context
- Tools used
- Trace ID
- Feedback category
- Expected correct behaviour

## Assignment — Build a feedback regression case

After implementing the API and buttons:

1. Ask your agent a question.
2. Give one answer a 👍 rating.
3. Give another answer a 👎 rating with category `incorrect`.
4. Try sending feedback as another user for the same message; it must return `403 Forbidden`.
5. Create this file:

`backend/eval_cases/feedback_regressions.json`

```json
[
  {
    "name": "expense-policy-answer-is-grounded",
    "input": "Can I claim a personal dinner as a business expense?",
    "expected_behavior": [
      "Does not invent company policy",
      "States uncertainty when policy is unavailable",
      "Asks the user to check the official policy or finance team"
    ],
    "source": "reviewed_user_feedback"
  }
]
```

This is your first feedback-derived regression test. Later, whenever you change prompts, retrieval, model, or tools, you will run this case again.

When you finish, ask for **“Day 29 AI quiz and programs”**.
