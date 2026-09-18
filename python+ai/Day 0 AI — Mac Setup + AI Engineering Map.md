# Day 0 AI — Mac Setup + AI Engineering Map

Create a separate workspace for AI. Do not reuse the existing full-stack `.venv`; each project should have its own isolated environment. Virtual environments prevent package/version conflicts. [Python `venv` docs](https://docs.python.org/3.12/library/venv.html)

Open Terminal and run:

```zsh
cd ~/Documents
mkdir python-ai-mastery
cd python-ai-mastery
```

## 1. Use Python 3.12 for this AI track

Your Python 3.14 is fine for learning core Python, but AI libraries can be slower to support very new Python versions. We will use Python 3.12 for reliable AI/ML packages.

Check whether it is already installed:

```zsh
python3.12 --version
```

If it says `command not found`, install it:

```zsh
brew install python@3.12
```

Then check again:

```zsh
python3.12 --version
```

## 2. Create and activate the AI environment

```zsh
python3.12 -m venv .venv
source .venv/bin/activate
```

Your prompt should now begin with:

```text
(.venv) your-name@Mac python-ai-mastery %
```

Confirm:

```zsh
python --version
which python
```

`which python` must end with:

```text
python-ai-mastery/.venv/bin/python
```

## 3. Install Day 0 tools

```zsh
python -m pip install --upgrade pip
pip install ruff pytest httpx pydantic python-dotenv numpy pandas jupyterlab scikit-learn openai
```

What these are for:

| Tool | Why we need it |
|---|---|
| `ruff` | Finds mistakes and keeps code professional |
| `pytest` | Automated testing |
| `httpx` | Calling APIs |
| `pydantic` | Safe, validated data models |
| `python-dotenv` | Keeps API keys out of source code |
| `numpy`, `pandas` | Data and numerical work |
| `scikit-learn` | Classical machine learning |
| `openai` | Connecting Python applications to LLMs |
| `jupyterlab` | Interactive AI/data experimentation |

We will add PyTorch and local-model tooling later, after you understand the foundations. PyTorch supports macOS installation through `pip`. [PyTorch setup guide](https://docs.pytorch.org/get-started/locally/)

## 4. Configure VS Code

Open the `python-ai-mastery` folder in VS Code through **File → Open Folder**.

Install these extensions:

- Python — Microsoft
- Pylance — Microsoft
- Jupyter — Microsoft
- Ruff — Astral Software

Then:

1. Press `Cmd + Shift + P`
2. Choose **Python: Select Interpreter**
3. Select the one ending in `.venv/bin/python`

VS Code uses the chosen interpreter for running, debugging, autocomplete, linting, and tests. [VS Code Python environments guide](https://code.visualstudio.com/docs/python/environments)

## 5. Create this starting structure

In VS Code Explorer, create:

```text
python-ai-mastery/
├── .venv/              ← do not edit or upload
├── lessons/
├── projects/
├── notebooks/
├── tests/
├── hello_ai.py
├── .env
└── .gitignore
```

Put this inside `.gitignore`:

```gitignore
.venv/
.env
__pycache__/
.ipynb_checkpoints/
```

Never put API keys in GitHub. `.env` is for secrets later.

## 6. First AI readiness test

Put this in `hello_ai.py`:

```python
import numpy as np
import pandas as pd
import sklearn
import pydantic

numbers = np.array([10, 20, 30])

print("Python AI environment is ready.")
print(f"Average: {numbers.mean()}")
print(f"NumPy: {np.__version__}")
print(f"Pandas: {pd.__version__}")
print(f"Scikit-learn: {sklearn.__version__}")
print(f"Pydantic: {pydantic.__version__}")
```

Run:

```zsh
python hello_ai.py
ruff check .
```

## AI engineering map

```text
Python fundamentals
       ↓
Data + SQL + APIs
       ↓
Machine Learning
       ↓
Deep Learning + Transformers
       ↓
LLMs, RAG, Agents
       ↓
Production AI systems
```

Day 1 AI will begin with the most important Python foundation for AI: **variables, objects, types, memory, mutability, and how Python actually executes code**.

First, run the setup through `ruff check .` and send me the output or any error.
