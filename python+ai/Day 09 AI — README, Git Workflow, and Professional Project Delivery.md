# Day 9 AI — README, Git Workflow, and Professional Project Delivery

Today you learn how to present and manage an AI project like an institute student, corporate engineer, and client-facing developer.

A good project is not only code.

```text
Professional AI project =
working code
+ tests
+ documentation
+ Git history
+ clear demo
+ safe handling of secrets
```

## Start your environment

**Windows PowerShell**

```powershell
cd "$env:USERPROFILE\Documents\python-ai-mastery"
.\.venv\Scripts\Activate.ps1
```

**Mac Terminal**

```zsh
cd ~/Documents/python-ai-mastery
source .venv/bin/activate
```

Git commands are the same on Mac and Windows.

---

# 1. README: explain your project before someone reads the code

Create `README.md` in your project root.

```md
# Python AI Mastery

A hands-on Python and AI engineering learning project.

## Current Project

Modular AI Learning Request Builder.

The application validates learner input and model settings, then creates a
structured LLM request. It does not call a real LLM yet.

## Features

- Validates topic and learner level
- Validates temperature and token limits
- Builds structured AI request dictionaries
- Uses modular project architecture
- Includes pytest tests
- Uses Ruff for code-quality checks

## Architecture

```text
ai_core/
├── config.py           # Model configuration validation
├── prompt_builder.py   # Prompt construction
├── request_service.py  # Request orchestration
├── agent_state.py      # Agent memory/state
└── conversation_store.py # JSON persistence
```

## Setup

Create and activate a virtual environment, then install dependencies:

```bash
pip install -r requirements.txt
```

## Run

```bash
python main.py
```

## Test

```bash
pytest -q
ruff check .
```

## Security

Never commit `.env`, API keys, passwords, access tokens, or private user data.

## Future Scope

- Connect to an LLM API
- Add RAG document retrieval
- Add agent tool calling
- Create a FastAPI backend
- Add a React frontend
```

---

# 2. Git: version history for your project

First check whether the folder is already a Git repository:

```bash
git status
```

If it says “not a git repository,” run this once:

```bash
git init
```

Check files before adding anything:

```bash
git status
git diff
```

Do not blindly commit secrets. Confirm `.gitignore` contains:

```gitignore
.venv/
.env
__pycache__/
.ipynb_checkpoints/
data/private/
```

Add selected safe files:

```bash
git add README.md main.py ai_core tests .gitignore
git commit -m "feat: add modular AI request builder"
```

View your history:

```bash
git log --oneline
```

---

# 3. Corporate Git workflow

Do not directly develop large features on `main`.

```bash
git switch -c feature/agent-memory-persistence
```

Work, test, and commit:

```bash
pytest -q
ruff check .
git add ai_core/conversation_store.py tests
git commit -m "feat: persist agent conversation memory"
```

Common professional commit prefixes:

```text
feat:     new user-facing feature
fix:      bug fix
test:     add or update tests
docs:     documentation changes
refactor: improve structure without changing behavior
chore:    maintenance/configuration work
```

Before creating a pull request, check:

```bash
git status
git diff main...HEAD
pytest -q
ruff check .
```

---

# 4. Real-time corporate approach

For every task, think like this:

```text
Ticket
→ requirements
→ acceptance criteria
→ technical design
→ implementation
→ tests
→ documentation
→ code review
→ release
```

Example ticket:

```text
Title:
Persist AI agent conversation memory to JSON.

Acceptance criteria:
- Save agent name, tools, messages, and timestamps.
- Load valid JSON successfully.
- Give meaningful errors for missing/invalid files.
- Tests pass.
- No private user data or secrets are committed.
```

---

# 5. Client-level approach

A client does not care whether you used `dataclass` or `Path`. They care about outcomes.

Explain the Day 8–9 project like this:

> “This module validates a learner’s topic and AI settings before creating a structured request. It prevents empty topics and invalid token limits, making the future AI feature safer and more reliable.”

Before building client features, ask:

```text
Who will use this?
What problem are we solving?
What input will users provide?
What is a correct output?
What data is sensitive?
What happens if the AI is wrong?
How will success be measured?
```

Never promise that an AI model is always correct. Explain limitations clearly.

---

# Day 9 Assignment — Professionalize Your AI Project

Complete these tasks:

1. Create the `README.md`.
2. Confirm `.gitignore` protects `.venv` and `.env`.
3. Run:

```bash
pytest -q
ruff check .
```

4. Initialize Git only if necessary.
5. Make one meaningful commit.
6. Prepare a 60-second project explanation:

```text
Problem:
Solution:
Architecture:
Validation:
Testing:
Next feature:
```

## Institute submission checklist

```text
[ ] README.md
[ ] Source-code screenshot
[ ] Test output screenshot
[ ] Ruff output screenshot
[ ] Architecture explanation
[ ] Git log screenshot
[ ] 60-second demo explanation
```

Day 10 AI will begin **NumPy and numerical computing**—the foundation underneath machine learning, embeddings, and neural networks.
