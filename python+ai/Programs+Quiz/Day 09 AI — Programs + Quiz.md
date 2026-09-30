# Day 9 AI — Programs + Quiz

**Code and Git commands are the same on Mac and Windows** after activating `.venv`.

## Program 1 — Project Readiness Audit

Create `tools/project_audit.py`:

```python
from pathlib import Path


REQUIRED_PATHS = [
    Path("README.md"),
    Path(".gitignore"),
    Path("main.py"),
    Path("ai_core"),
    Path("tests"),
]


def audit_project(root: Path) -> bool:
    all_present = True

    for required_path in REQUIRED_PATHS:
        full_path = root / required_path
        exists = full_path.exists()

        status = "FOUND" if exists else "MISSING"
        print(f"{status}: {required_path}")

        if not exists:
            all_present = False

    return all_present


if __name__ == "__main__":
    project_root = Path(".")

    if audit_project(project_root):
        print("\nProject structure is ready.")
    else:
        print("\nProject structure needs attention.")
```

Run:

```bash
python tools/project_audit.py
```

## Program 2 — Application Metadata

Create `ai_core/project_metadata.py`:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ProjectMetadata:
    name: str
    version: str
    description: str
    status: str


def get_project_metadata() -> ProjectMetadata:
    return ProjectMetadata(
        name="Python AI Mastery",
        version="0.1.0",
        description="Modular AI learning-request builder.",
        status="Development",
    )
```

Create `tools/show_project_info.py`:

```python
from ai_core.project_metadata import get_project_metadata


metadata = get_project_metadata()

print(f"Project: {metadata.name}")
print(f"Version: {metadata.version}")
print(f"Status: {metadata.status}")
print(f"Description: {metadata.description}")
```

Run:

```bash
python tools/show_project_info.py
```

## Program 3 — Dependency File

Create `requirements.txt` using:

```bash
python -m pip freeze > requirements.txt
```

A teammate can recreate the same environment with:

```bash
pip install -r requirements.txt
```

In corporate projects, dependency versions must be recorded so “works on my machine” does not become a production problem.

## Git Practice Lab

Run these only after checking that `.env` is ignored:

```bash
git status
git add README.md requirements.txt tools ai_core tests
git commit -m "docs: add project documentation and setup files"
git log --oneline
```

# Day 9 AI MCQ Quiz

Reply like: `1.B 2.A 3.C ...`

1. What is the primary purpose of a `README.md` file?

   A. Store passwords  
   B. Explain the project, setup, use, and architecture  
   C. Replace source code  
   D. Run tests automatically  

2. What should `.gitignore` do?

   A. Prevent selected files such as `.env` and `.venv` from being committed  
   B. Delete project files  
   C. Run the program  
   D. Replace Git history  

3. Which command shows changed, staged, and untracked files?

   A. `git diff`  
   B. `git init`  
   C. `git status`  
   D. `git push`  

4. What does `git add` do?

   A. Deletes a file  
   B. Stages selected changes for a commit  
   C. Uploads code to GitHub  
   D. Creates a virtual environment  

5. What does a Git commit represent?

   A. A saved snapshot of staged changes with a message  
   B. An API key  
   C. A deleted branch  
   D. A Python exception  

6. What does this command do?

```bash
git switch -c feature/agent-memory
```

   A. Deletes `main`  
   B. Creates and switches to a new branch  
   C. Commits every file  
   D. Pushes code to a remote server  

7. Why use feature branches in a corporate project?

   A. To isolate work before review and merging  
   B. To avoid testing  
   C. To hide code permanently  
   D. To replace documentation  

8. Which commit message is best for adding a new feature?

   A. `changes`  
   B. `feat: add agent memory persistence`  
   C. `final final latest`  
   D. `do not read`  

9. What is an acceptance criterion?

   A. A vague idea about a feature  
   B. A clear, testable condition proving a requirement is met  
   C. A Git branch name  
   D. A Python package  

10. Which is a good acceptance criterion?

   A. “Make the AI good”  
   B. “Make the code advanced”  
   C. “Empty prompts return a clear validation error”  
   D. “Use many files”  

11. Before committing code, what must never be included?

   A. Tests  
   B. API keys, passwords, and access tokens  
   C. README files  
   D. Python modules  

12. Why run `pytest -q` before committing?

   A. To verify existing behavior still works  
   B. To upload code  
   C. To create a branch  
   D. To install Git  

13. Why run `ruff check .` before committing?

   A. To deploy the application  
   B. To check code-quality problems  
   C. To delete unused files  
   D. To replace tests  

14. What should you explain to a client instead of technical implementation details alone?

   A. Only Python syntax  
   B. The user problem, solution, limitations, and expected outcome  
   C. Your private Git history  
   D. API keys  

15. When should `git init` be run?

   A. Before every commit  
   B. Only once when a folder is not already a Git repository  
   C. After deleting `.gitignore`  
   D. Every time you create a Python file
