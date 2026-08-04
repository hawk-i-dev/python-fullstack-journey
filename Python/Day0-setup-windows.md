Day 0 for Windows laptop: setup only. No coding yet.

As of August 4, 2026, use Python `3.14.x` stable, Node.js `24.x LTS`, Git for Windows, VS Code, and PostgreSQL. Avoid Python `3.15` beta for now.

**Install These**
Open PowerShell and run these one by one:

```powershell
winget install --id Microsoft.PowerShell --source winget
winget install --id Microsoft.WindowsTerminal -e
winget install --id Microsoft.VisualStudioCode -e
winget install --id Git.Git -e
winget install --id Python.Python.3.14 -e
winget install --id OpenJS.NodeJS.LTS -e
winget install --id PostgreSQL.PostgreSQL -e
```

If any `winget` command fails, install manually from official pages:

- Python for Windows: Python.org lists Python `3.14.6` as latest stable Python 3 release. ([python.org](https://www.python.org/downloads/windows/?key5sk1=9adb1b81fc698e5d0b45e99d3e5cc92d83e55057&utm_source=openai))
- VS Code: use the Windows User Setup installer. ([code.visualstudio.com](https://code.visualstudio.com/docs/setup/windows?utm_source=openai))
- Git for Windows: official Git Windows installer. ([gitforwindows.org](https://gitforwindows.org/?utm_source=openai))
- Node.js: install the LTS version, currently `v24.15.0 LTS`. ([nodejs.org](https://nodejs.org/en/download?form=MG0AV3&utm_source=openai))
- PostgreSQL: use the Windows EDB installer with pgAdmin included. ([postgresql.org](https://www.postgresql.org/download/windows/?utm_source=openai))

**VS Code Extensions**
Install these in VS Code:

- Python
- Pylance
- Python Debugger
- Black Formatter
- Ruff
- ESLint
- Prettier
- GitLens
- Thunder Client

**Git Setup**
Run this in PowerShell:

```powershell
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true
git config --global core.editor "code --wait"
```

**Create Your Learning Workspace**

```powershell
cd "$env:USERPROFILE\Documents"
mkdir python-fullstack-journey
cd python-fullstack-journey
git init
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install black ruff pytest
code .
```

If activation is blocked, run this once:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Then close and reopen PowerShell.

**Verification**
Run these and keep the output:

```powershell
python --version
py --version
pip --version
git --version
node --version
npm --version
code --version
psql --version
```

If `python` opens Microsoft Store, fix it here:

`Settings > Apps > Advanced app settings > App execution aliases`

Turn off:

- `python.exe`
- `python3.exe`

Day 0 is complete when all verification commands show versions. Send me those outputs, then we start Day 1 properly.
