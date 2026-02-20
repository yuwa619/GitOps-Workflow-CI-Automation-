# Getting Started

Welcome to the GitOps Workflow & CI Automation project. This guide will get you from zero to a passing build and your first pull request in **under 1 hour**.

---

## Prerequisites

Make sure you have the following installed before starting:

| Tool | Version | Check |
|------|---------|-------|
| Python | 3.10+ | `python --version` |
| Git | Any recent | `git --version` |
| pip | Bundled with Python | `pip --version` |

You will also need:
- A **GitHub account** with read access to this repository
- A **Trello account** (optional — cards are managed automatically by CI)

---

## Step 1 — Clone the Repository (5 min)

```bash
git clone https://github.com/yuwa619/GitOps-Workflow-CI-Automation-.git
cd GitOps-Workflow-CI-Automation-
```

---

## Step 2 — Set Up Your Environment (10 min)

Create and activate a Python virtual environment:

```bash
# macOS / Linux
python -m venv .venv
source .venv/bin/activate

# Windows (Command Prompt)
python -m venv .venv
.venv\Scripts\activate.bat

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

You should see `(.venv)` in your terminal prompt. Now install dependencies:

```bash
pip install -r requirements.txt
```

---

## Step 3 — Run the Tests (5 min)

Verify everything is working:

```bash
pytest test_app.py -v
```

Expected output — all 8 tests pass:

```
test_app.py::test_health PASSED
test_app.py::test_sum PASSED
test_app.py::test_reverse PASSED
test_app.py::test_sum_with_negative PASSED
test_app.py::test_sum_with_zero PASSED
test_app.py::test_reverse_empty_string PASSED
test_app.py::test_reverse_single_char PASSED
test_app.py::test_sum_missing_keys_defaults_to_zero PASSED

8 passed in 0.XXs
```

Also run the linter to confirm no style issues:

```bash
flake8 app.py test_app.py
```

No output = clean code.

---

## Step 4 — Understand the Branch Workflow (5 min)

This project uses **GitHub Flow**:

```
main  ──────────────────────────────────────────► (always stable)
         ▲                         ▲
         │  merge via PR           │  merge via PR
         │                         │
feature/TRELLO-002-xxx    feature/TRELLO-003-yyy
```

**Rules:**
- Never commit directly to `main`
- Every change needs its own feature branch
- Every branch name must include the Trello ID: `feature/TRELLO-###-short-description`
- Every commit must reference the Trello ID: `[TRELLO-###] Short description`

---

## Step 5 — Make Your First Change (15 min)

### 1. Create a feature branch

```bash
git checkout main
git pull origin main
git checkout -b feature/TRELLO-002-your-description
```

### 2. Make your changes

Edit the relevant files. Keep changes small and focused.

### 3. Verify locally before pushing

```bash
# Run linter
flake8 app.py test_app.py

# Run tests
pytest test_app.py -v
```

Both must pass before you push.

### 4. Commit with the correct format

```bash
git add <your-changed-files>
git commit -m "[TRELLO-002] Short description of what you did"
```

### 5. Push your branch

```bash
git push -u origin feature/TRELLO-002-your-description
```

---

## Step 6 — Open a Pull Request (5 min)

Go to the repository on GitHub. You will see a banner offering to open a PR for your recently pushed branch. Click it, or use:

```bash
gh pr create --title "TRELLO-002 - Your description" --base main
```

Once the PR is open:
- GitHub Actions will automatically run the CI pipeline
- A Trello card will be created and moved to **In Progress**
- If CI passes, the card moves to **Review/QA**

---

## Troubleshooting

### `ModuleNotFoundError: No module named 'flask'`
Your virtual environment is not activated. Run:
```bash
source .venv/bin/activate   # macOS/Linux
.venv\Scripts\activate      # Windows
```

### Flake8 reports errors
Read the error message carefully — it includes the file, line number, and rule code.
Common fixes:
- `W292` — add a newline at the end of the file
- `E302` — add two blank lines before a function definition
- `F401` — remove an unused import

### Tests are failing
Run `pytest test_app.py -v` and read the assertion error. Make sure your changes did not modify the existing API behaviour. If you added a new endpoint, add a corresponding test.

### CI is failing on GitHub but passes locally
Check that you activated the virtual environment and installed from `requirements.txt`. Also confirm you are running Python 3.10+:
```bash
python --version
```

### Accidentally committed to `main`
```bash
git checkout -b feature/TRELLO-###-rescue-branch
git checkout main
git reset --hard origin/main
```
Your commits are now on the feature branch, and `main` is back to its remote state.

---

## Quick Reference

```bash
# Start new work
git checkout main && git pull origin main
git checkout -b feature/TRELLO-###-description

# Before every commit
flake8 app.py test_app.py
pytest test_app.py -v

# Commit
git add <files>
git commit -m "[TRELLO-###] Description"

# Push & open PR
git push -u origin feature/TRELLO-###-description
gh pr create --title "TRELLO-### - Description" --base main
```
