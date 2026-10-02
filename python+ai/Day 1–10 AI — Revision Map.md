# Day 1–10 AI — Revision Map

You have completed the first foundation phase of Python + AI engineering.

| Days | What you learned | Real AI use |
|---|---|---|
| Day 1 | Variables, objects, mutability, lists, dictionaries | AI request data and chat messages |
| Day 2 | Conditions, loops, validation, comprehensions | Clean prompts and filter invalid messages |
| Day 3 | Functions, arguments, scope, validation | Reusable prompt and request builders |
| Day 4 | Errors, exceptions, logging | Reliable handling of bad input/API failures |
| Day 5 | Pytest and automated tests | Prevent regressions in AI systems |
| Day 6 | Classes, dataclasses, agent state | Agent memory, messages, and tool history |
| Day 7 | Files, JSON, persistence | Save/load agent conversation memory |
| Day 8 | Modules, packages, architecture | Maintainable AI project structure |
| Day 9 | README, Git, corporate/client workflow | Professional project delivery |
| Day 10 | NumPy, vectors, matrices, similarity | Embeddings and the base of RAG search |

Your progress:

```text
Python foundations
      ↓
Reliable, tested code
      ↓
Agent memory and project structure
      ↓
Numerical vectors and embedding similarity
      ↓
Ready for data handling, machine learning, LLMs, RAG, and agents
```

# Phase 1 Capstone Assignment

Build: **AI Learning Assistant Prototype**

It must:

```text
1. Accept a learner topic and level.
2. Build a validated AI request.
3. Store conversation history in JSON.
4. Load saved conversation history.
5. Use NumPy to rank three or more learning documents.
6. Return the most relevant document name.
7. Include pytest tests.
8. Include README.md and .gitignore.
```

Suggested structure:

```text
python-ai-mastery/
├── ai_core/
│   ├── config.py
│   ├── prompt_builder.py
│   ├── request_service.py
│   ├── agent_state.py
│   ├── conversation_store.py
│   └── vector_search.py
├── tests/
├── data/
├── main.py
├── README.md
└── .gitignore
```

## Corporate definition of done

```text
[ ] Clear requirement and acceptance criteria
[ ] Modular code
[ ] Input validation
[ ] Useful error messages
[ ] Tests pass
[ ] Ruff passes
[ ] No secrets committed
[ ] README explains setup and architecture
[ ] Git commit history is meaningful
```

## Client demo explanation

> “This prototype accepts a learning topic, finds the most relevant learning material using vector similarity, stores conversation memory safely, and prepares a validated request for a future LLM integration.”

You are ready to continue with **Day 11 AI — Pandas and preparing real-world AI datasets.**
