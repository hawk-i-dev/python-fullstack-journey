# Day 28 AI — Token Usage, Latency, and Agent Run Records

Day 27 counted requests. Today you record what each completed AI run actually used:

```text
user → conversation → agent run
                     ├── model
                     ├── input tokens
                     ├── output tokens
                     ├── duration
                     └── outcome
```

Accurate token tracking helps control prompt size, analyze latency, and estimate costs. Output-token usage can be larger than visible text because it can include formatting, reasoning, or tool-related tokens. [Official OpenAI token-counting guide](https://developers.openai.com/api/docs/guides/token-counting)

## 1. Add `AgentRun` to `app/models.py`

```python
from datetime import datetime, timezone
from uuid import uuid4

from sqlalchemy import DateTime, Integer, String
from sqlalchemy.orm import Mapped, mapped_column


class AgentRun(Base):
    __tablename__ = "agent_runs"

    id: Mapped[str] = mapped_column(
        String(36),
        primary_key=True,
        default=lambda: str(uuid4()),
    )

    user_id: Mapped[str] = mapped_column(
        String(100),
        index=True,
    )

    conversation_id: Mapped[str] = mapped_column(
        String(36),
        index=True,
    )

    model: Mapped[str] = mapped_column(String(100))

    status: Mapped[str] = mapped_column(
        String(30),
        default="completed",
    )

    input_tokens: Mapped[int] = mapped_column(
        Integer,
        default=0,
    )

    output_tokens: Mapped[int] = mapped_column(
        Integer,
        default=0,
    )

    total_tokens: Mapped[int] = mapped_column(
        Integer,
        default=0,
    )

    duration_ms: Mapped[int] = mapped_column(
        Integer,
        default=0,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
    )
```

Restart Uvicorn so the learning-project database creates the new table.

## 2. Create `app/observability.py`

```python
from sqlalchemy.orm import Session

from app.models import AgentRun


def record_completed_agent_run(
    db: Session,
    user_id: str,
    conversation_id: str,
    model: str,
    input_tokens: int,
    output_tokens: int,
    duration_ms: int,
) -> AgentRun:
    agent_run = AgentRun(
        user_id=user_id,
        conversation_id=conversation_id,
        model=model,
        status="completed",
        input_tokens=input_tokens,
        output_tokens=output_tokens,
        total_tokens=input_tokens + output_tokens,
        duration_ms=duration_ms,
    )

    db.add(agent_run)
    db.commit()
    db.refresh(agent_run)

    return agent_run
```

Do not store raw prompts, passwords, API keys, or private document content in this usage table.

## 3. Read usage from the completed stream event

Refactor the streaming adapter in `app/llm_client.py`:

```python
from dataclasses import dataclass
from typing import Literal


@dataclass(frozen=True)
class AgentStreamEvent:
    kind: Literal["text", "completed"]
    text: str = ""
    input_tokens: int = 0
    output_tokens: int = 0
```

Then:

```python
async def openai_agent_stream(
    input_messages: list[dict[str, str]],
):
    stream = await start_response_stream(input_messages)

    async for event in stream:
        if event.type == "response.output_text.delta":
            yield AgentStreamEvent(
                kind="text",
                text=event.delta,
            )

        elif event.type == "response.completed":
            usage = event.response.usage

            yield AgentStreamEvent(
                kind="completed",
                input_tokens=usage.input_tokens,
                output_tokens=usage.output_tokens,
            )
```

## 4. Record latency and usage after completion

In `app/main.py`:

```python
import os
from time import perf_counter

from app.observability import record_completed_agent_run
```

Before streaming starts:

```python
started_at = perf_counter()
input_tokens = 0
output_tokens = 0
```

Inside the streaming loop:

```python
async for agent_event in openai_agent_stream(history):
    if agent_event.kind == "text":
        complete_answer.append(agent_event.text)

        yield sse_event(
            "token",
            {"text": agent_event.text},
        )

    elif agent_event.kind == "completed":
        input_tokens = agent_event.input_tokens
        output_tokens = agent_event.output_tokens
```

After the final assistant message is saved:

```python
duration_ms = int(
    (perf_counter() - started_at) * 1000
)

record_completed_agent_run(
    db=db,
    user_id=current_user.user_id,
    conversation_id=conversation.id,
    model=os.environ["OPENAI_MODEL"],
    input_tokens=input_tokens,
    output_tokens=output_tokens,
    duration_ms=duration_ms,
)
```

## 5. Create a personal usage-summary endpoint

Add to `app/main.py`:

```python
from sqlalchemy import func, select

from app.models import AgentRun
```

```python
@app.get("/usage/me/summary")
def get_my_usage_summary(
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> dict[str, int]:
    statement = select(
        func.count(AgentRun.id),
        func.coalesce(func.sum(AgentRun.input_tokens), 0),
        func.coalesce(func.sum(AgentRun.output_tokens), 0),
        func.coalesce(func.sum(AgentRun.total_tokens), 0),
        func.coalesce(func.avg(AgentRun.duration_ms), 0),
    ).where(
        AgentRun.user_id == current_user.user_id,
        AgentRun.status == "completed",
    )

    (
        total_runs,
        input_tokens,
        output_tokens,
        total_tokens,
        average_duration_ms,
    ) = db.execute(statement).one()

    return {
        "total_runs": int(total_runs),
        "input_tokens": int(input_tokens),
        "output_tokens": int(output_tokens),
        "total_tokens": int(total_tokens),
        "average_duration_ms": int(average_duration_ms),
    }
```

Example response:

```json
{
  "total_runs": 8,
  "input_tokens": 4200,
  "output_tokens": 1250,
  "total_tokens": 5450,
  "average_duration_ms": 1840
}
```

## Day 28 Assignment

1. Add the `AgentRun` table.
2. Record a completed run after streaming succeeds.
3. Test `/usage/me/summary` with Alice and Bob.
4. Confirm Alice cannot see Bob’s totals.
5. Add these values to the React screen:
   - total requests
   - total tokens
   - average response time
6. Do not calculate customer billing yet. Record accurate data first.

## Remember

```text
Request count answers: “How often?”
Token usage answers:  “How much AI work?”
Latency answers:      “How fast?”
Outcome/status:       “Did it succeed?”
```

This is the foundation for an AI usage dashboard, cost controls, quality monitoring, and production troubleshooting.
