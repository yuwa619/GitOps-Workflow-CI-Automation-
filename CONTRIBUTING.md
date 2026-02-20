# Contributing Guide

This document defines the complete development workflow for this project. All contributors — regardless of experience level — must follow these standards. They exist to protect the stability of `main`, reduce review friction, and keep the Trello board accurate.

---

## Table of Contents

1. [Development Workflow Overview](#development-workflow-overview)
2. [Branching Rules](#branching-rules)
3. [Commit Format](#commit-format)
4. [Pull Request Process](#pull-request-process)
5. [CI/CD Explanation](#cicd-explanation)
6. [Coding Standards](#coding-standards)
7. [Failure Handling](#failure-handling)

---

## Development Workflow Overview

```
1. Pick a Trello card from Backlog
2. Create a feature branch from main
3. Write code + tests
4. Verify locally (lint + test)
5. Push branch and open PR
6. CI runs automatically
7. On CI pass → PR is reviewed
8. On approval → merge to main
9. Trello card moves to Done automatically
```

Every step is either enforced by automation or checked during PR review. There are no exceptions.

---

## Branching Rules

### Branch naming

All branches **must** follow this format:

```
feature/TRELLO-###-short-description
```

| Component | Rule |
|-----------|------|
| `feature/` | Required prefix |
| `TRELLO-###` | Zero-padded card number (e.g. `TRELLO-003`) |
| `short-description` | Lowercase, hyphen-separated, max 5 words |

**Valid examples:**
```
feature/TRELLO-002-input-validation
feature/TRELLO-003-update-readme
feature/TRELLO-004-add-health-tests
```

**Invalid examples:**
```
my-fix              ← missing TRELLO ID
feature/fix         ← missing TRELLO ID
TRELLO-002          ← missing feature/ prefix
feature/TRELLO-2    ← ID not zero-padded
```

### Protected branches

| Branch | Rule |
|--------|------|
| `main` | No direct pushes. All changes via PR only. |
| `feature/**` | Free to push; CI runs on every push |

### Keeping your branch up to date

Before opening a PR, rebase your branch on the latest `main`:

```bash
git fetch origin
git rebase origin/main
```

Resolve any conflicts locally. Never merge `main` into your feature branch — rebase only.

---

## Commit Format

Every commit **must** follow this format:

```
[TRELLO-###] Short imperative description

Optional longer explanation of what changed and why.
Keep lines under 72 characters.
```

### Rules

- The `[TRELLO-###]` prefix is **mandatory** on every commit
- Use imperative mood: "Add", "Fix", "Update", "Remove" — not "Added", "Fixed"
- Subject line must be under 72 characters
- Do not end the subject line with a period
- Leave a blank line between subject and body (if a body is needed)

### Valid examples

```
[TRELLO-001] Add GitHub Actions CI pipeline

[TRELLO-002] Add input validation to /sum endpoint

Validates that a and b are integers before processing.
Returns 400 if invalid types are provided.

[TRELLO-003] Fix flake8 W292 missing newline in test_app.py
```

### Invalid examples

```
fixed bug                  ← no TRELLO ID, not imperative
[TRELLO-1] update          ← ID not zero-padded, too vague
WIP                        ← no TRELLO ID, not descriptive
```

---

## Pull Request Process

### Before opening a PR

Run these checks locally and confirm both pass:

```bash
flake8 app.py test_app.py   # must produce no output
pytest test_app.py -v        # must show 8 passed
```

Do not open a PR with a known failing check.

### PR title format

```
TRELLO-### - Short description of the change
```

Example: `TRELLO-002 - Add input validation to sum endpoint`

### PR description

Include:
- What was changed and why
- Any decisions made (e.g. "chose X over Y because...")
- How to test the change manually if applicable

### Review requirements

- At least **one approval** is required before merging
- All CI checks must be green
- The PR branch must be up to date with `main`

### Merging

Use **squash and merge** or **merge commit** — do not rebase-merge, as it rewrites commit history and breaks the Trello PR URL lookup.

After merging, delete the feature branch from GitHub.

---

## CI/CD Explanation

The CI pipeline is defined in `.github/workflows/ci.yml` and runs on:
- Every push to a `feature/**` branch
- Every pull request targeting `main`
- Every push to `main`

### Pipeline stages

```
┌─────────────────────────────────┐
│  Job: build-test                │
│  ┌───────────────────────────┐  │
│  │ 1. Install Dependencies   │  │
│  │    pip install -r req.txt │  │
│  └───────────┬───────────────┘  │
│              ▼                  │
│  ┌───────────────────────────┐  │
│  │ 2. Lint                   │  │
│  │    flake8 app.py test_...│  │
│  └───────────┬───────────────┘  │
│              ▼                  │
│  ┌───────────────────────────┐  │
│  │ 3. Test                   │  │
│  │    pytest test_app.py -v  │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

Stages run in strict order. If stage 2 (lint) fails, stage 3 (test) does not run.

### Trello automation jobs

In addition to `build-test`, the pipeline includes Trello automation:

| Job | Trigger | Action |
|-----|---------|--------|
| `trello-in-progress` | PR opened / push to PR | Create card in Backlog → move to In Progress |
| `trello-review-qa` | CI passes | Move card to Review/QA + comment |
| `trello-ci-failed` | CI fails | Move card back to In Progress + comment |
| `trello-done` | PR merged | Move to Done + green label + merge comment |

### Required secrets

The following GitHub repository secrets must be set for Trello automation to work:

| Secret | Description |
|--------|-------------|
| `TRELLO_KEY` | Your Trello API key |
| `TRELLO_TOKEN` | Your Trello API token |
| `TRELLO_BOARD_ID` | ID of the Trello board |
| `TRELLO_LIST_BACKLOG` | ID of the Backlog list |
| `TRELLO_LIST_IN_PROGRESS` | ID of the In Progress list |
| `TRELLO_LIST_REVIEW_QA` | ID of the Review/QA list |
| `TRELLO_LIST_DONE` | ID of the Done list |

---

## Coding Standards

This project follows **PEP 8**, enforced by flake8.

### Key rules

- 4 spaces for indentation (no tabs)
- Maximum line length: 79 characters (flake8 default)
- Two blank lines between top-level functions
- One blank line between methods inside a class
- No trailing whitespace
- Every file must end with a newline character

### Test standards

- Every new endpoint or function requires at least one test
- Tests must cover the happy path and at least one edge case
- Test function names must be descriptive: `test_sum_with_negative_numbers` not `test2`
- All 8 tests must pass before any PR can be merged

### Import order (enforced by flake8)

```python
# 1. Standard library
import os

# 2. Third-party
from flask import Flask, request, jsonify

# 3. Local
from app import app
```

---

## Failure Handling

### CI lint failure

**Symptom:** GitHub Actions shows a red ✗ on the Lint step.

**Fix:**
```bash
flake8 app.py test_app.py
# Read the output — it shows file:line:col: RULE message
# Fix each issue, then re-run to confirm clean
git add <files>
git commit -m "[TRELLO-###] Fix flake8 lint errors"
git push
```
CI will automatically re-run on the new push.

### CI test failure

**Symptom:** GitHub Actions shows a red ✗ on the Test step.

**Fix:**
```bash
pytest test_app.py -v
# Read the failing assertion to understand what broke
# Fix the code (never fix the test to hide a bug)
git add <files>
git commit -m "[TRELLO-###] Fix failing test for /endpoint"
git push
```

### Merge conflict

**Symptom:** GitHub PR shows "This branch has conflicts that must be resolved".

**Fix:**
```bash
git fetch origin
git rebase origin/main
# Resolve conflicts in your editor
git add <resolved-files>
git rebase --continue
git push --force-with-lease
```

Use `--force-with-lease` (never `--force`) to avoid overwriting others' work.

### Accidentally pushed to `main`

```bash
# Create a new branch from current main (your commits are here)
git checkout -b feature/TRELLO-###-rescue

# Reset main to match remote
git checkout main
git reset --hard origin/main

# Push the rescue branch and open a PR
git push -u origin feature/TRELLO-###-rescue
```

Never `git push --force` to `main`.
