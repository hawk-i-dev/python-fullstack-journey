# Day 0 AI — Windows Setup

We will create a separate **Python 3.12 AI workspace**. Do not use the old full-stack project’s `.venv`.

## 1. Create the project folder

Open **PowerShell** and run:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\Documents\python-ai-mastery"
cd "$env:USERPROFILE\Documents\python-ai-mastery"
```

## 2. Check Python 3.12

```powershell
py -3.12 --version
```

If it shows Python 3.12, continue.

If it says Python 3.12 is not installed, run:

```powershell
winget install -e --id Python.Python.3.12
```

Close PowerShell, open it again, return to the project folder, and run the check again.

## 3. Create the AI virtual environment

```powershell
py -3.12 -m venv .venv
```

Earlier, PowerShell blocked activation scripts on your laptop. For this terminal only, allow the environment activation:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Now activate it:

```powershell
.\.venv\Scripts\Activate.ps1
```

You should see this at the beginning of the prompt:

```text
(.venv) PS C:\Users\YourName\Documents\python-ai-mastery>
```

Confirm it is correct:

```powershell
python --version
where.exe python
```

The first Python path must contain:

```text
python-ai-mastery\.venv\Scripts\python.exe
```

## 4. Install the AI learning tools

```powershell
python -m pip install --upgrade pip
pip install ruff pytest httpx pydantic python-dotenv numpy pandas jupyterlab scikit-learn openai
```

We will use PyTorch later, after the Python/data foundations are clear.

## 5. Configure VS Code

Open the `python-ai-mastery` folder in VS Code.

Install these extensions:

- Python — Microsoft
- Pylance — Microsoft
- Jupyter — Microsoft
- Ruff — Astral Software

Then press `Ctrl + Shift + P` → **Python: Select Interpreter** → choose:

```text
Python 3.12 ('.venv')
```

VS Code uses that selected environment for running code, autocomplete, debugging, linting, and tests. [VS Code Python environments](https://code.visualstudio.com/docs/python/environments)

## 6. Create starter files

Create these files/folders in VS Code:

```text
python-ai-mastery/
├── lessons/
├── projects/
├── notebooks/
├── tests/
├── hello_ai.py
├── .env
└── .gitignore
```

Put this in `.gitignore`:

```gitignore
.venv/
.env
__pycache__/
.ipynb_checkpoints/
```

## 7. Test the AI setup

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

```powershell
python hello_ai.py
ruff check .
```

A virtual environment keeps this project’s packages isolated from every other Python project. [Python virtual-environment documentation](https://docs.python.org/3.12/library/venv.html)

After this works, send the output. Then we start **Day 1 AI: How Python works internally — variables, objects, types, memory, and mutability.**
