# Days 1–20 AI: One Complete Mental Model

You have learned one connected system, not 20 separate topics:

```text
Python foundation
  → clean data
  → retrieve knowledge
  → generate grounded answers
  → use controlled tools
  → plan workflows
  → add safety
  → trace and evaluate
```

The common idea in every day is:

> Take input → validate it → process it predictably → produce useful output → test, log, and improve it.

## What each phase built

| Days | Focus | What to remember |
|---|---|---|
| 1–5 | Core Python | Variables, conditions, loops, functions, errors, tests |
| 6–9 | Software engineering | OOP, state, JSON/files, modules, Git, README |
| 10–11 | Data foundation | NumPy arrays and Pandas datasets |
| 12–14 | RAG | Clean documents → chunk → search → answer only from retrieved context |
| 15–16 | LLM application | API integration, prompts, multi-turn memory, sessions |
| 17–18 | Agents | Tools, tool loop, routing, plans, manager vs specialist |
| 19–20 | Production quality | Guardrails, approvals, audit logs, traces, evals |

## The concepts you must remember permanently

### 1. Python is the base

AI applications are still Python applications.

You must be confident with:

- Variables, lists, dictionaries, loops, conditions
- Functions and type hints
- Exceptions and logging
- Classes/dataclasses
- JSON, files, modules, packages
- `pytest`
- Git commits and README documentation

If your Python basics are weak, AI code will feel confusing. If Python is strong, AI frameworks become learnable tools.

### 2. Inputs are never trusted

This is true for user inputs, CSV files, API responses, tool-call arguments, and LLM output.

Always ask:

```text
Is this value present?
Is its type correct?
Is its format valid?
Is it within allowed limits?
Can it cause a harmful action?
```

Example:

```python
if not isinstance(amount, (int, float)):
    raise ValueError("Amount must be a number")

if amount < 0:
    raise ValueError("Amount cannot be negative")
```

### 3. RAG is not “upload file and ask question”

A reliable RAG workflow is:

```text
Documents
  → clean text
  → chunk text
  → create/search vectors
  → retrieve relevant chunks
  → give chunks to the LLM
  → answer with sources
```

The main rule:

> If the retrieved context does not support the answer, say that the information was not found.

RAG reduces hallucination, but it does not automatically guarantee truth. Good chunks, retrieval, citations, and testing matter.

### 4. An LLM is not automatically an agent

A chatbot answers text.

An agent can use tools and take controlled actions.

```text
LLM + instructions + tools + memory/state + safety + evaluation
= agent system
```

The official OpenAI building-agents guidance describes the key building blocks as models, tools, state/memory, and orchestration. [OpenAI building-agents guide](https://developers.openai.com/tracks/building-agents)

### 5. Tool calling is a controlled loop

```text
User asks question
  → Model requests a tool
  → Python validates the request
  → Python executes only an allowed tool
  → Tool result returns to model
  → Model gives final answer
```

The model suggests a tool call. Your Python application decides whether it is allowed.

Never give an LLM unrestricted access to:

- Database deletion
- Money transfer
- File deletion
- Shell commands
- API keys
- User passwords
- Production systems

### 6. Plans make agents reliable

Before execution, the agent should know:

```text
What is the user asking?
Which tools are needed?
What data is missing?
Which actions are risky?
What must be approved?
```

Reliable workflow:

```text
Route → Plan → Validate → Approve if needed → Execute → Report
```

### 7. Safety is code, not only prompt text

A prompt saying “do not delete data” is not enough.

You need:

- Tool allowlists
- Argument validation
- Guardrails
- Approval requirements
- One-time confirmation tokens
- Token expiry
- Audit logs
- Tests

For sensitive side effects, human approval should pause the workflow before execution. [OpenAI guardrails and approvals guide](https://developers.openai.com/api/docs/guides/agents/guardrails-approvals)

### 8. Logs and evals turn projects into products

When something fails, do not only fix it once.

```text
Failure happens
  → inspect trace
  → identify cause
  → write regression test/eval case
  → fix code or prompt
  → rerun tests
  → prevent the same failure later
```

This is how corporate AI teams improve systems. Traces show one run; evaluation datasets measure behavior repeatedly. [OpenAI agent-evaluation guide](https://developers.openai.com/api/docs/guides/agent-evals)

# The strategy you should follow from now

## Daily 1–3 hour structure

| Time | Work |
|---|---|
| 20 min | Review yesterday’s concepts and code |
| 40 min | Learn the new concept |
| 45 min | Build the day’s feature |
| 20 min | Write or update tests |
| 15 min | Debug, document, Git commit |

## Before writing code

Ask these questions:

1. What problem am I solving?
2. Who is the user?
3. What input will I receive?
4. What output should be produced?
5. What could fail?
6. What data is sensitive?
7. What is the acceptance criteria?
8. How will I test it?

## While building

Use this order:

```text
Small working version
  → validation
  → error handling
  → tests
  → logging
  → documentation
  → improvements
```

Do not build everything at once.

## When debugging

Use this order:

```text
1. Read the complete error message
2. Find the exact failing file and line
3. Check input values and types
4. Reproduce with a small test
5. Fix the root cause
6. Add a test so it does not return
```

Never randomly change many files hoping the issue disappears.

# Corporate-level thinking

For every AI feature, think in five layers:

```text
Feature:
What should it do?

Data:
What data is used? Is it clean and safe?

AI:
What prompt, model, RAG context, or tools are needed?

Safety:
What must be validated, blocked, or approved?

Quality:
How will we test, trace, monitor, and improve it?
```

# Your Day 1–20 master checklist

You should now be able to build a small AI project that has:

- Python project structure
- Virtual environment and `.env`
- Typed functions and validation
- Pytest tests
- Local document ingestion
- Chunking and vector search
- Grounded RAG responses with citations
- Multi-turn local memory
- LLM provider adapter
- Tool calling
- Agent routing and plans
- Tool allowlists
- Guardrails and approvals
- Audit logs
- Traces and evaluation cases
- Git commits and a README

The main mindset to carry forward is:

> Build useful AI features, but make every step observable, testable, safe, and understandable.

You are no longer only learning Python or only learning AI. You are learning how to build reliable AI software.
