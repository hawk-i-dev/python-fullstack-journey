# Day 23 AI — Persistent Agent Memory with SQLite

Today you will store conversations in a database. Until now, your agent state was temporary: Python variables and JSON trace files. A real AI application needs durable, user-owned conversation history.

The principle is:

```text
Authenticated user
    └── owns conversations
            └── contain messages
                    └── create traces and tool actions
```

Model calls are independent unless you preserve and supply conversation context. Durable conversation state lets users continue on another device or after a server restart. [Official OpenAI conversation-state guidance](https://developers.openai.com/api/docs/guides/conversation-state)

## Day 23 outcome

Build these API endpoints:

```text
POST /conversations
GET  /conversations
GET  /conversations/{conversation_id}/messages
POST /assist
```

`/assist` will save the user message and assistant response to the correct conversation.

## 1. Install SQLAlchemy

Same command on Mac and Windows after activating `.venv`:

```bash
python -m pip install sqlalchemy
```

Add this to `.gitignore`:

```gitignore
data/*.db
```

SQLite is excellent for local learning and small applications. Corporate deployments commonly move the same design to PostgreSQL.

## 2. Create `app/database.py`

```python
from collections.abc import Generator
from pathlib import Path

from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Session, sessionmaker


Path("data").mkdir(exist_ok=True)

DATABASE_URL = "sqlite:///./data/agent.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    autocommit=False,
)


class Base(DeclarativeBase):
    pass


def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

## 3. Create `app/models.py`

```python
from datetime import datetime
from uuid import uuid4

from sqlalchemy import DateTime, ForeignKey, String, Text
from sqlalchemy.orm import Mapped, mapped_column

from app.database import Base


class Conversation(Base):
    __tablename__ = "conversations"

    id: Mapped[str] = mapped_column(
        String(36),
        primary_key=True,
        default=lambda: str(uuid4()),
    )
    owner_id: Mapped[str] = mapped_column(String(100), index=True)
    title: Mapped[str] = mapped_column(String(200))
    created_at: Mapped[datetime] = mapped_column(
        DateTime,
        default=datetime.utcnow,
    )


class Message(Base):
    __tablename__ = "messages"

    id: Mapped[int] = mapped_column(primary_key=True)
    conversation_id: Mapped[str] = mapped_column(
        ForeignKey("conversations.id"),
        index=True,
    )
    role: Mapped[str] = mapped_column(String(20))
    content: Mapped[str] = mapped_column(Text)
    created_at: Mapped[datetime] = mapped_column(
        DateTime,
        default=datetime.utcnow,
    )
```

A message role should be one of:

```text
user
assistant
tool
system
```

## 4. Create database tables on startup

In `app/main.py`:

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI

from app.database import Base, engine
from app import models


@asynccontextmanager
async def lifespan(app: FastAPI):
    Base.metadata.create_all(bind=engine)
    yield


app = FastAPI(
    title="Finance Assistant Agent API",
    lifespan=lifespan,
)
```

`create_all()` is appropriate for learning. In client/corporate projects, use database migrations, usually Alembic, so schema changes are controlled and reversible.

## 5. Add conversation routes

Create `app/conversations.py`:

```python
from datetime import datetime

from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, Field
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.auth import CurrentUser, get_current_user
from app.database import get_db
from app.models import Conversation, Message


router = APIRouter(prefix="/conversations", tags=["conversations"])


class ConversationCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)


class ConversationResponse(BaseModel):
    id: str
    title: str
    created_at: datetime


@router.post(
    "",
    response_model=ConversationResponse,
    status_code=status.HTTP_201_CREATED,
)
def create_conversation(
    payload: ConversationCreate,
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> Conversation:
    conversation = Conversation(
        owner_id=current_user.user_id,
        title=payload.title,
    )

    db.add(conversation)
    db.commit()
    db.refresh(conversation)

    return conversation


@router.get("", response_model=list[ConversationResponse])
def list_conversations(
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> list[Conversation]:
    statement = (
        select(Conversation)
        .where(Conversation.owner_id == current_user.user_id)
        .order_by(Conversation.created_at.desc())
    )

    return list(db.scalars(statement))


@router.get("/{conversation_id}/messages")
def list_messages(
    conversation_id: str,
    current_user: CurrentUser = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> list[Message]:
    conversation = db.get(Conversation, conversation_id)

    if conversation is None:
        raise HTTPException(status_code=404, detail="Conversation not found")

    if conversation.owner_id != current_user.user_id:
        raise HTTPException(status_code=403, detail="Access denied")

    statement = (
        select(Message)
        .where(Message.conversation_id == conversation_id)
        .order_by(Message.created_at.asc())
    )

    return list(db.scalars(statement))
```

Register the router in `app/main.py`:

```python
from app.conversations import router as conversations_router

app.include_router(conversations_router)
```

## 6. Save messages in `/assist`

Extend your request schema:

```python
class AssistRequest(BaseModel):
    question: str = Field(min_length=3, max_length=500)
    conversation_id: str | None = None
```

Before the agent workflow runs:

```python
user_message = Message(
    conversation_id=conversation.id,
    role="user",
    content=request.question,
)

db.add(user_message)
db.commit()
```

After the agent responds:

```python
assistant_message = Message(
    conversation_id=conversation.id,
    role="assistant",
    content=result["message"],
)

db.add(assistant_message)
db.commit()
```

Do not load an entire years-long conversation into every LLM request. In later days, you will use:

```text
recent messages + user profile + retrieved RAG context + task-specific tool results
```

## Test checklist

Use Swagger at:

```text
http://127.0.0.1:8000/docs
```

Test this flow:

1. Authorize as Alice.
2. Create a conversation: `My October Budget`.
3. Call `/assist` using that conversation ID.
4. Fetch its messages.
5. Authorize as Bob.
6. Confirm Bob cannot fetch Alice’s conversation or messages.

## Assignment

Complete these three tests:

```text
Alice creates a conversation → 201
Alice lists conversations → sees only her conversations
Bob requests Alice’s messages → 403
```

## Real-world approach to remember

| Data | Owner | Access rule |
|---|---|---|
| Conversation | One user | Only its owner |
| Message | Conversation owner | Only through owned conversation |
| Trace | One user/conversation | Only authorized owner |
| API token | Authentication system | Never return or log it |
| Agent tool action | Authenticated user | Validate, authorize, audit |

By end of today, you will have the foundation of a real multi-user AI assistant: authentication from Day 22 plus persistent, isolated memory from Day 23.
