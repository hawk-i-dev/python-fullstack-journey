# Day 27 AI — User Quotas and Cost Protection

Today you protect your AI product from accidental overuse, abusive traffic, and unexpected cost.

There are two different limits:

```text
Provider rate limit  → OpenAI organization/project protection
Your app quota       → Protects each user in your product
```

OpenAI limits are organization/project based, so your own app should add per-user controls such as daily limits. [Official OpenAI rate-limit guide](https://developers.openai.com/api/docs/guides/rate-limits)

## Today’s outcome

Build:

```text
UsageEvent database table
POST /assist/stream checks user quota
GET /usage/me shows remaining requests
```

For learning, set a limit of 20 AI requests per user per UTC day.

## 1. Add `UsageEvent` to `app/models.py`

```python
from datetime import datetime, timezone

from sqlalchemy import DateTime, String
from sqlalchemy.orm import Mapped, mapped_column


class UsageEvent(Base):
    __tablename__ = "usage_events"

    id: Mapped[int] = mapped_column(primary_key=True)

    user_id: Mapped[str] = mapped_column(
        String(100),
        index=True,
    )

    event_type: Mapped[str] = mapped_column(
        String(50),
        default="agent_request",
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(timezone.utc),
        index=True,
    )
```

Because your app already uses `Base.metadata.create_all()`, restart Uvicorn to create the new table.

## 2. Create `app/usage.py`

```python
from datetime import datetime, timezone

from fastapi import HTTPException, status
from sqlalchemy import func, select
from sqlalchemy.orm import Session

from app.models import UsageEvent


DAILY_REQUEST_LIMIT = 20


def start_of_current_utc_day() -> datetime:
    now = datetime.now(timezone.utc)

    return now.replace(
        hour=0,
        minute=0,
        second=0,
        microsecond=0,
    )


def get_daily_request_count(
    db: Session,
    user_id: str,
) -> int:
    statement = (
        select(func.count())
        .select_from(UsageEvent)
        .where(
            UsageEvent.user_id == user_id,
            UsageEvent.event_type == "agent_request",
            UsageEvent.created_at >= start_of_current_utc_day(),
        )
    )

    return db.scalar(statement) or 0


def consume_daily_request_quota(
    db: Session,
    user_id: str,
    limit: int = DAILY_REQUEST_LIMIT,
) -> int:
    used_requests = get_daily_request_count(
        db=db,
        user_id=user_id,
    )

    if used_requests >= limit:
        raise HTTPException(
            status_code=status.HTTP_429_TOO_MANY_REQUESTS,
            detail="Daily AI request limit reached. Try again tomorrow.",
        )

    usage_event = UsageEvent(
        user_id=user_id,
        event_type="agent_request",
    )

    db.add(usage_event)
    db.commit()

    return used_requests + 1
```

This records accepted AI requests before starting the agent. That prevents a user from repeatedly retrying failures to bypass the quota.

## 3. Enforce quota before streaming

In `app/main.py`, import:

```python
from app.usage import consume_daily_request_quota
```

Inside `/assist/stream`, after authentication and ownership verification, add:

```python
consume_daily_request_quota(
    db=db,
    user_id=current_user.user_id,
)
```

The request now follows this safe flow:

```text
Authenticate
→ Verify conversation ownership
→ Check and record quota
→ Save user message
→ Start agent stream
→ Save final assistant message
```

## 4. Add `GET /usage/me`

In `app/main.py`:

```python
from app.usage import (
    DAILY_REQUEST_LIMIT,
    get_daily_request_count,
)
```

Then add:

```python
@app.get("/usage/me")
def get_my_usage(
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> dict[str, int]:
    used = get_daily_request_count(
        db=db,
        user_id=current_user.user_id,
    )

    return {
        "daily_limit": DAILY_REQUEST_LIMIT,
        "used_today": used,
        "remaining_today": max(
            DAILY_REQUEST_LIMIT - used,
            0,
        ),
    }
```

Swagger request header:

```text
Authorization: Bearer dev-alice-token
```

Example response:

```json
{
  "daily_limit": 20,
  "used_today": 3,
  "remaining_today": 17
}
```

## 5. Test the quota

Create `tests/test_usage.py`:

```python
import pytest
from fastapi import HTTPException
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from app.database import Base
from app.usage import (
    consume_daily_request_quota,
    get_daily_request_count,
)


@pytest.fixture
def db():
    engine = create_engine("sqlite://")
    Base.metadata.create_all(bind=engine)

    session_local = sessionmaker(bind=engine)
    session: Session = session_local()

    try:
        yield session
    finally:
        session.close()


def test_usage_event_is_recorded(db):
    consume_daily_request_quota(
        db=db,
        user_id="alice",
        limit=2,
    )

    assert get_daily_request_count(db, "alice") == 1


def test_user_cannot_exceed_daily_limit(db):
    consume_daily_request_quota(
        db=db,
        user_id="alice",
        limit=2,
    )
    consume_daily_request_quota(
        db=db,
        user_id="alice",
        limit=2,
    )

    with pytest.raises(HTTPException) as error:
        consume_daily_request_quota(
            db=db,
            user_id="alice",
            limit=2,
        )

    assert error.value.status_code == 429
```

Run:

```bash
pytest -q
```

## Corporate and client mindset

A real product tracks more than request count:

| Measure | Why it matters |
|---|---|
| Requests per user/day | Prevent abuse |
| Input and output tokens | Understand cost |
| Model name | Cost and quality analysis |
| Tool calls | Find expensive workflows |
| Request ID | Troubleshooting |
| Error status | Reliability monitoring |

Do not estimate customer billing from rough token calculations unless the business has an approved pricing policy. First record accurate usage; then finance/product teams decide limits and pricing.

## Day 27 Assignment

1. Implement the `UsageEvent` table.
2. Set the local limit to `3` temporarily.
3. Send four `/assist/stream` requests as Alice.
4. Confirm request four returns `429`.
5. Check `/usage/me`.
6. Add the remaining-request count to the React UI.

Next, we can build an admin-style usage dashboard and add token-level tracking.
