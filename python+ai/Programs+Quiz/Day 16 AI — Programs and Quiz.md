## Day 16 AI — Programs and Quiz

These programs work the same on Mac and Windows once your virtual environment is activated. We are practicing multi-turn RAG chat: store a small local conversation history, retrieve fresh context for every new question, and save the session safely. The Responses API also supports conversation state through manual history or `previous_response_id`; we are learning the local-history approach first. [OpenAI conversation-state guide](https://developers.openai.com/api/docs/guides/conversation-state)

### Program 1: Show only recent chat history

Create `chat_history_demo.py`:

```python
from chat_memory import ChatSession
from chat_prompt import build_history_block

session = ChatSession(session_id="demo-session")

session.add_turn(
    question="What is RAG?",
    answer="RAG retrieves relevant documents before generating an answer.",
    sources=("ai_rag_notes.txt",),
)

session.add_turn(
    question="Why use chunks?",
    answer="Chunks make retrieval more precise.",
    sources=("ai_rag_notes.txt",),
)

session.add_turn(
    question="Why limit history?",
    answer="It reduces irrelevant context and prompt size.",
    sources=("ai_rag_notes.txt",),
)

print(build_history_block(session, limit=2))
```

Expected idea: it should show only the latest two turns, not the complete conversation.

### Program 2: Latest sources helper

Add this function to `chat_memory.py`:

```python
def latest_sources(session: ChatSession) -> tuple[str, ...]:
    if not session.turns:
        return ()

    return session.turns[-1].sources
```

Create `latest_sources_demo.py`:

```python
from chat_memory import ChatSession, latest_sources

session = ChatSession(session_id="sources-demo")

session.add_turn(
    question="What is an embedding?",
    answer="An embedding is a numeric representation of meaning.",
    sources=("embeddings.txt", "rag_notes.txt"),
)

print("Latest sources:", latest_sources(session))
```

### Program 3: Save and reload a chat session

Create `session_file_demo.py`:

```python
from pathlib import Path

from chat_memory import ChatSession, load_session, save_session

file_path = Path("data/sessions/demo_session.json")

session = ChatSession(session_id="demo-session")
session.add_turn(
    question="What is cosine similarity?",
    answer="It measures the similarity between two vectors.",
    sources=("vector_search_notes.txt",),
)

save_session(session, file_path)

restored_session = load_session(file_path)

print("Session:", restored_session.session_id)
print("Turns:", len(restored_session.turns))
print("Latest question:", restored_session.turns[-1].question)
```

Real-world rule: never put API keys, passwords, or sensitive client data into session files.

### Program 4: Test the latest-sources helper

Add to `tests/test_chat_memory.py`:

```python
from chat_memory import ChatSession, latest_sources


def test_latest_sources_returns_empty_tuple_for_new_session():
    session = ChatSession(session_id="test-session")

    assert latest_sources(session) == ()


def test_latest_sources_returns_sources_from_latest_turn():
    session = ChatSession(session_id="test-session")

    session.add_turn(
        question="First question",
        answer="First answer",
        sources=("first.txt",),
    )

    session.add_turn(
        question="Second question",
        answer="Second answer",
        sources=("second.txt", "notes.txt"),
    )

    assert latest_sources(session) == ("second.txt", "notes.txt")
```

Run:

```bash
pytest
```

### Assignment

Add a `/sources` command to `main_chat.py`.

- `/sources` displays source files used by the latest answer.
- If no question has been asked, show: `No sources available yet.`
- Do not call the LLM for `/sources`.
- Add at least two tests.

Corporate approach: conversation memory must be limited, tested, persisted safely, and cleared on user request. Client requirement example: “The assistant must show which documents supported its latest response.”

---

## Day 16 AI Quiz

Reply in this format: `1.B, 2.A, ...`

1. What makes a chat application “multi-turn”?

   A. It has multiple files  
   B. It can use earlier conversation context for follow-up questions  
   C. It uses multiple databases  
   D. It has multiple users  

2. Why should chat history be limited?

   A. To reduce irrelevant context, cost, and privacy exposure  
   B. To make Python faster  
   C. To remove source citations  
   D. To avoid testing  

3. For a new user question in a RAG chat, what should happen first?

   A. Reuse the previous answer without checking  
   B. Retrieve fresh relevant chunks  
   C. Delete all documents  
   D. Call the model with no context  

4. Which data best represents one saved chat turn?

   A. Only the user’s name  
   B. Only the LLM model name  
   C. Question, answer, sources, and timestamp  
   D. Only the source file path  

5. What should `recent_turns(3)` return?

   A. The first three turns  
   B. The latest three turns  
   C. All turns  
   D. No turns  

6. What should `/clear` do?

   A. Remove the local conversation history  
   B. Delete the RAG documents  
   C. Delete the database  
   D. Delete Python packages  

7. Why save a chat session to JSON?

   A. To make the LLM smarter automatically  
   B. To continue the conversation after restarting the app  
   C. To expose secrets  
   D. To replace retrieval  

8. If retrieval finds no relevant context, what is the safest behavior?

   A. Invent an answer confidently  
   B. Use unrelated documents  
   C. Tell the user no relevant information was found  
   D. Save a fake source  

9. What is a benefit of local session memory?

   A. The application controls where and how long data is stored  
   B. It removes the need for authentication  
   C. It guarantees every answer is correct  
   D. It removes all API costs  

10. What does `previous_response_id` help with in the Responses API?

   A. Deleting previous messages  
   B. Chaining a response to prior response state  
   C. Creating embeddings  
   D. Reading JSON files  

11. Why should each RAG answer display sources?

   A. So users can verify where the answer came from  
   B. To increase token usage  
   C. To hide the answer  
   D. To avoid retrieval  

12. How should you test chat-memory code?

   A. Only by making paid LLM calls  
   B. With unit tests and fake/mock provider clients where needed  
   C. Without tests  
   D. By deleting session files manually  

13. What is a risk of sending the full conversation forever?

   A. Large prompts can become costly and include irrelevant context  
   B. Python stops supporting functions  
   C. JSON becomes invalid automatically  
   D. Retrieval becomes impossible  

14. Which action gives a user better privacy control?

   A. Always save every conversation permanently  
   B. Provide `/clear` and avoid storing secrets  
   C. Print API keys in the terminal  
   D. Share session files publicly  

15. What should a production AI chat application define clearly?

   A. Retention, privacy, source citation, and error-handling policies  
   B. Only the page colors  
   C. Only the project name  
   D. Only the Python version
