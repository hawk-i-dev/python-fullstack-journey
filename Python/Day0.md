## Day 0 — Full Python Full-Stack Setup

We’ll set up a clean development environment on your Mac. Do these in order.

### 1. Install core tools

Open Terminal and check whether they already exist:

```bash
python3 --version
git --version
node --version
npm --version
```

Install anything missing:

- **Python 3.12 or newer:** [python.org/downloads](https://www.python.org/downloads/)
- **VS Code:** [code.visualstudio.com](https://code.visualstudio.com/)
- **Git:** [git-scm.com/download/mac](https://git-scm.com/download/mac)
- **Node.js (LTS):** [nodejs.org](https://nodejs.org/)  
  We need Node later for React.

After installing, close and reopen Terminal, then re-run the checks.

### 2. Install VS Code extensions

Open VS Code → Extensions, and install:

- Python (Microsoft)
- Pylance (Microsoft)
- Python Debugger (Microsoft)
- ESLint
- Prettier – Code formatter
- GitLens
- Thunder Client  
  Useful later for testing APIs.

### 3. Configure Git

Replace the example values with your own name and email—the email should ideally be the one you’ll use for GitHub.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
```

Verify:

```bash
git config --global --list
```

### 4. Create GitHub account and your learning repository

Create a GitHub account if you do not have one: [github.com](https://github.com/)

Then create a new repository:

```text
Repository name: python-fullstack-journey
Visibility: Public
Initialize with README: Yes
```

A public repository is helpful for your portfolio.

### 5. Create your local project folder

```bash
cd ~/Documents
mkdir python-fullstack-journey
cd python-fullstack-journey
git init
```

Create this initial structure:

```text
python-fullstack-journey/
├── README.md
├── python-basics/
├── backend/
├── frontend/
├── projects/
└── notes/
```

You can create it using:

```bash
mkdir python-basics backend frontend projects notes
touch README.md
```

Put this in `README.md`:

```markdown
# Python Full-Stack Journey

My learning journey from Python fundamentals to building production-ready full-stack applications.
```

Then save your first Git checkpoint:

```bash
git add .
git commit -m "chore: initialize Python full-stack learning journey"
```

### 6. Create and test a Python virtual environment

Inside `python-fullstack-journey`:

```bash
python3 -m venv .venv
source .venv/bin/activate
python --version
```

When it is active, Terminal should show `(.venv)` at the beginning of the line.

In VS Code, press `Cmd + Shift + P` → **Python: Select Interpreter** → select the one containing `.venv`.

### 7. Database setup — install but don’t study yet

Install **PostgreSQL** using [Postgres.app](https://postgresapp.com/). It is the simplest choice for macOS.

We will start using it in the Django phase. You do not need to learn it today.

### Day 0 completion checklist

- [ ] Python, Git, Node, and npm version commands work
- [ ] VS Code and Python extensions installed
- [ ] Git name/email configured
- [ ] GitHub account and repository created
- [ ] Local folders created
- [ ] `.venv` activates successfully
- [ ] First Git commit completed

Once these are done, send me the output of:

```bash
python3 --version
git --version
node --version
```

Then we’ll begin Day 1 properly.
