# Day 23 AI — Quiz and Programs

Today’s practical goal: persist user conversations safely and give the agent only the recent relevant messages.

A conversation is durable state; it can hold messages, tool calls, and outputs. But do not send unlimited history to an LLM—use a bounded recent window to control context size and cost. [Official OpenAI conversation-state guide](https://developers.openai.com/api/docs/guides/conversation-state)

## Program 1 — Create or verify an owned conversation

Create `app/repositories.py`:

```python
from fastapi import HTTPException
from sqlalchemy.orm import Session

from app.models import Conversation


def get_owned_conversation(
    db: Session,
    conversation_id: str,
    owner_id: str,
) -> Conversation:
    conversation = db.get(Conversation, conversation_id)

    if conversation is None:
        raise HTTPException(
            status_code=404,
            detail="Conversation not found",
        )

    if conversation.owner_id != owner_id:
        raise HTTPException(
            status_code=403,
            detail="Access denied",
        )

    return conversation
```

Use this function in every route that reads, updates, deletes, or adds messages to a conversation.

Corporate rule: do not repeat ownership logic differently in many places. Put it in one trusted function and test it well.

## Program 2 — Save and load recent agent memory

Add this to `app/repositories.py`:

```python
from sqlalchemy import select

from app.models import Message


def save_message(
    db: Session,
    conversation_id: str,
    role: str,
    content: str,
) -> Message:
    message = Message(
        conversation_id=conversation_id,
        role=role,
        content=content,
    )

    db.add(message)
    db.commit()
    db.refresh(message)

    return message


def get_recent_messages(
    db: Session,
    conversation_id: str,
    limit: int = 8,
) -> list[Message]:
    statement = (
        select(Message)
        .where(Message.conversation_id == conversation_id)
        .order_by(Message.created_at.desc())
        .limit(limit)
    )

    newest_first = list(db.scalars(statement))
    return list(reversed(newest_first))
```

Convert messages into agent context:

```python
def to_agent_history(messages: list[Message]) -> list[dict[str, str]]:
    return [
        {
            "role": message.role,
            "content": message.content,
        }
        for message in messages
        if message.role in {"user", "assistant"}
    ]
```

Use it in `/assist`:

```python
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

recent_messages = get_recent_messages(
    db=db,
    conversation_id=conversation.id,
)

history = to_agent_history(recent_messages)

result = run_observed_workflow(
    user_id=current_user.user_id,
    question=request.question,
    history=history,
    chunks=[],
)

save_message(
    db=db,
    conversation_id=conversation.id,
    role="assistant",
    content=result["message"],
)
```

## Program 3 — Test ownership and message order

Create `tests/test_repositories.py`:

```python
import pytest
from fastapi import HTTPException
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from app.database import Base
from app.models import Conversation
from app.repositories import (
    get_owned_conversation,
    get_recent_messages,
    save_message,
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


def test_owner_can_access_conversation(db):
    conversation = Conversation(
        owner_id="alice",
        title="October Budget",
    )

    db.add(conversation)
    db.commit()

    result = get_owned_conversation(
        db=db,
        conversation_id=conversation.id,
        owner_id="alice",
    )

    assert result.id == conversation.id


def test_other_user_cannot_access_conversation(db):
    conversation = Conversation(
        owner_id="alice",
        title="Private Budget",
    )

    db.add(conversation)
    db.commit()

    with pytest.raises(HTTPException) as error:
        get_owned_conversation(
            db=db,
            conversation_id=conversation.id,
            owner_id="bob",
        )

    assert error.value.status_code == 403


def test_recent_messages_are_oldest_to_newest(db):
    conversation = Conversation(
        owner_id="alice",
        title="Test Conversation",
    )

    db.add(conversation)
    db.commit()

    save_message(db, conversation.id, "user", "First question")
    save_message(db, conversation.id, "assistant", "First answer")
    save_message(db, conversation.id, "user", "Second question")

    messages = get_recent_messages(
        db=db,
        conversation_id=conversation.id,
        limit=2,
    )

    assert [message.content for message in messages] == [
        "First answer",
        "Second question",
    ]
```

Run:

```bash
pytest -q
```

## Day 23 Quiz

1. Why do we store conversations in a database?  
   A. To make Python faster  
   B. To preserve user state after restart  
   C. To avoid authentication  
   D. To replace APIs  

2. What does a conversation usually contain?  
   A. Only a password  
   B. Messages and related agent activity  
   C. Only frontend CSS  
   D. Only a database URL  

3. Which field establishes the owner of a conversation?  
   A. `title`  
   B. `created_at`  
   C. `owner_id`  
   D. `content`  

4. Before reading a conversation, the API must verify:  
   A. Browser theme  
   B. Conversation ownership  
   C. Internet speed  
   D. README length  

5. If Bob accesses Alice’s existing conversation, which status is appropriate?  
   A. 200  
   B. 201  
   C. 403  
   D. 500  

6. Which database is ideal for local learning and a small prototype?  
   A. SQLite  
   B. Excel only  
   C. Git  
   D. HTML  

7. What is a foreign key used for in `Message`?  
   A. It connects a message to its conversation  
   B. It encrypts a password  
   C. It starts the server  
   D. It creates a token  

8. Why should we limit recent messages sent to an LLM?  
   A. To avoid storing conversations  
   B. To manage cost and context size  
   C. To remove authentication  
   D. To prevent SQLAlchemy imports  

9. What is the correct chronological order for agent history?  
   A. Newest message first  
   B. Random message order  
   C. Oldest relevant message first  
   D. Only assistant messages  

10. Which roles are normally useful in chat history?  
   A. `user` and `assistant`  
   B. `database` and `server`  
   C. `html` and `css`  
   D. `token` and `secret`  

11. What should `get_owned_conversation()` do if the conversation does not exist?  
   A. Create a new unrelated conversation  
   B. Return `404`  
   C. Return `500`  
   D. Return Alice’s first conversation  

12. What should it do if the owner does not match?  
   A. Return `403`  
   B. Return all conversations  
   C. Change the owner  
   D. Delete the database  

13. Why use a repository function for ownership checks?  
   A. It centralizes important security logic  
   B. It replaces tests  
   C. It hides all errors  
   D. It removes the database  

14. In production, what should replace `Base.metadata.create_all()` for schema changes?  
   A. Manual file deletion  
   B. Alembic migrations  
   C. Browser refresh  
   D. Swagger UI  

15. What is the correct safe flow when a user sends a chat message?  
   A. Trust any conversation ID and call the model  
   B. Authenticate → verify ownership → save message → load bounded history → run agent → save response  
   C. Send all database records to the model  
   D. Delete older messages immediately  

Reply in this format:

```text
1.B
2.B
...
15.B
```
