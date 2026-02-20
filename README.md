# GitOps Workflow & CI Automation

![CI](https://github.com/yuwa619/GitOps-Workflow-CI-Automation-/actions/workflows/ci.yml/badge.svg)
![Python](https://img.shields.io/badge/python-3.10-blue)
![License](https://img.shields.io/badge/license-MIT-green)

A lightweight internal service demonstrating a production-style GitOps workflow connecting **Trello**, **GitHub**, and **GitHub Actions** — with automated linting, testing, and Trello card lifecycle management.

---

## Project Overview

This project implements a GitOps-style development workflow for a small engineering team. It solves three real problems:

1. **Poor visibility** — every task is tracked as a Trello card that moves automatically through Backlog → In Progress → Review/QA → Done via GitHub Actions
2. **Broken code reaching main** — CI enforces lint (flake8) and tests (pytest) on every push and PR; failing CI blocks merges
3. **Slow onboarding** — new developers can be productive in under 1 hour using `GETTING_STARTED.md`

The application itself is a simple Flask API with three endpoints (`/health`, `/sum`, `/reverse-string`). The focus is entirely on workflow, automation, and documentation.

---

## Architecture

```
Developer Machine
      │
      ▼
feature/TRELLO-###-description  ──push──►  GitHub
                                               │
                              ┌────────────────┤
                              │                ▼
                              │     GitHub Actions CI
                              │     ┌─────────────────┐
                              │     │ 1. Install deps  │
                              │     │ 2. Lint (flake8) │
                              │     │ 3. Test (pytest) │
                              │     └────────┬────────┘
                              │              │
                              │    ┌─────────┴──────────┐
                              │    │                    │
                              │  PASS                 FAIL
                              │    │                    │
                              │    ▼                    ▼
                              │  Trello             Trello
                              │  Review/QA          In Progress
                              │    │                (card stays)
                              │    │
                              └────┤ PR merged
                                   ▼
                                  main (stable)
                                   │
                                   ▼
                                Trello → Done
                             (green label + comment)
```

**Trello card lifecycle (fully automated by CI):**

| GitHub Event | Trello Action |
|---|---|
| PR opened / push to PR | Create card in Backlog → move to In Progress |
| CI passes | Move to Review/QA + comment with commit SHA & CI run link |
| CI fails | Move back to In Progress + comment with failure link |
| PR merged | Move to Done + green ✅ Completed label + merge comment |

---

## Workflow Description

This project follows **GitHub Flow**:

1. A Trello card is created for every task (automatically when a PR is opened)
2. Developer creates a feature branch: `feature/TRELLO-###-short-description`
3. Developer commits with format: `[TRELLO-###] Short description`
4. Developer opens a PR to `main`
5. GitHub Actions runs: Install → Lint → Test
6. If CI passes, the PR can be merged; the Trello card moves to Review/QA
7. On merge, the card moves to Done automatically

**Branch protection:**
- No direct pushes to `main`
- All changes go through a PR
- CI must pass before merge

---

## Commit Convention

Every commit **must** reference a Trello card ID:

```
[TRELLO-###] Short description

Optional longer explanation of why this change was made.
```

**Examples:**
```
[TRELLO-001] Add GitHub Actions CI pipeline
[TRELLO-002] Add input validation to /sum endpoint
[TRELLO-003] Write GETTING_STARTED onboarding guide
```

**Rules:**
- Use imperative mood ("Add", "Fix", "Update" — not "Added", "Fixed")
- Keep the subject line under 72 characters
- Reference the Trello ID at the start

---

## Setup Instructions

### Prerequisites
- Python 3.10+
- Git
- pip

### Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/yuwa619/GitOps-Workflow-CI-Automation-.git
cd GitOps-Workflow-CI-Automation-

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # macOS/Linux
# .venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the tests
pytest test_app.py -v

# 5. Run the linter
flake8 app.py test_app.py

# 6. Run the app locally
python app.py
```

The API will be available at `http://127.0.0.1:5000`.

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Returns `{"status": "UP"}` |
| POST | `/sum` | Adds two numbers `{"a": 5, "b": 3}` → `{"result": 8}` |
| POST | `/reverse-string` | Reverses a string `{"text": "hello"}` → `{"result": "olleh"}` |

---

## Reflection Questions

### Q1: Branching Strategy — Which strategy did you choose and why?

I chose **GitHub Flow** as the branching strategy for this project. GitHub Flow is a lightweight, branch-based workflow built around a single long-lived branch (`main`) that is always deployable. Every new piece of work — whether a feature, bug fix, or documentation update — happens on a short-lived feature branch named `feature/TRELLO-###-description`, which is then merged into `main` via a pull request.

I chose GitHub Flow over GitFlow (which has separate `develop`, `release`, and `hotfix` branches) for several reasons. First, this is a small team (1–3 developers) working on a lightweight service, so the added complexity of GitFlow is unnecessary overhead. GitHub Flow is simpler to understand and enforce, which reduces mistakes and speeds up onboarding. Second, GitHub Flow aligns naturally with CI/CD: because `main` is always deployable and every PR goes through automated checks, the workflow enforces quality without requiring a separate staging branch. Third, the naming convention `feature/TRELLO-###-description` creates a direct, readable link between the branch and the Trello card, making it easy to trace any line of code back to the business requirement that generated it. This traceability is one of the core principles of GitOps — Git as the single source of truth, where every change has a clear origin and purpose.

---

### Q2: Preventing Bad Code — How did you ensure code wasn't merged if tests failed?

The primary mechanism is the **GitHub Actions CI pipeline**, which runs automatically on every push to a feature branch and on every pull request targeting `main`. The pipeline executes three stages in strict order: install dependencies, lint with flake8, and run tests with pytest. If any stage fails, the entire workflow is marked as failed and the PR is blocked from merging.

Flake8 enforces PEP 8 style rules, catching issues like unused imports, undefined variables, and formatting inconsistencies before they reach reviewers. Pytest runs all 8 test cases covering the application's endpoints, including edge cases like negative numbers, zero values, and empty strings. A failure in either stage produces a red build status directly on the GitHub PR page, making it immediately visible to anyone reviewing the code.

Beyond automated checks, the project also enforces a pull request workflow — no direct pushes to `main` are permitted. This means every change must be reviewed as a PR, and the CI pipeline must complete before the merge button is available. The combination of automated linting, automated testing, and mandatory PR review creates multiple layers of protection. This mirrors real-world engineering practices where quality is enforced systematically rather than relying on individual discipline. The intentional failure (the missing newline in `test_app.py` that triggered a W292 flake8 error) demonstrated exactly this — the CI caught a code quality issue before it could be merged.

---

### Q3: Scaling to 30 Engineers — What would change?

Scaling from a 1–3 person team to 30 engineers would require changes at every layer of the workflow. On the **branching side**, GitHub Flow would likely need to evolve into a trunk-based development model with feature flags, or a modified GitFlow with a `develop` branch to reduce the risk of integration conflicts when many developers are merging simultaneously. Branch naming conventions and automated enforcement (via commit message linting in CI) would become essential, not optional.

On the **CI/CD side**, a single workflow file would need to be split into parallelised jobs to keep build times under 5 minutes as the test suite grows. Caching of dependencies (`pip cache`, `actions/cache`) would become mandatory. Test sharding — splitting the test suite across multiple runners — would be needed once pytest runs exceed a few minutes. Code coverage reporting (via `pytest-cov`) and coverage thresholds enforced in CI would be introduced to prevent test debt from accumulating silently.

On the **review process**, mandatory code owners (`CODEOWNERS` file) would ensure that changes to critical files (like the CI workflow itself or security-sensitive code) require approval from a designated senior engineer. The PR template would be formalised with a checklist. On the **Trello/task management side**, the board would likely be replaced or supplemented by Jira for its richer sprint planning, dependency tracking, and reporting features. The automation scripts linking GitHub to the task tracker would need to handle concurrent PRs for the same card (multiple developers working on sub-tasks), which the current single-PR-per-card model does not account for.

---

### Q4: Understanding DevOps — What did this project teach you about Ops?

This project fundamentally changed how I think about the relationship between development and operations. Before starting, I understood DevOps as a concept — "developers and ops teams working together" — but building this pipeline made the practical mechanics concrete. The most important lesson was that **automation is a form of documentation**. The `ci.yml` file is not just a script; it is a precise, executable specification of what "good code" means on this project. Anyone reading it immediately knows: code must pass flake8, code must pass all 8 tests, and code can only reach `main` through a reviewed PR. This is more reliable than a wiki page that can go out of date.

The second major lesson was about **feedback loops**. The faster a developer learns their code is broken, the cheaper it is to fix. By running lint and tests on every push — not just on PR — the CI gives feedback within 30–60 seconds of a `git push`. This prevents the situation where a developer spends hours building on top of broken code before discovering the issue. The Trello automation extends this feedback loop to the project management layer: a card's position on the board is always an accurate reflection of the code's state in GitHub, not a manually updated approximation. Third, I learned that **ops work is product work**. Time spent on the CI pipeline, the documentation, and the onboarding guide directly accelerates every future developer who works on the project. It is not overhead — it is infrastructure for human productivity.

---

### Q5: Trello-Git Correlation — How do you correlate cards with commits and why?

The correlation between Trello cards and Git commits is established through two mechanisms that work together. The first is **naming convention**: every branch is named `feature/TRELLO-###-description` and every commit message begins with `[TRELLO-###]`. This creates a human-readable link that appears in `git log`, in GitHub PR lists, and in the branch list — making it trivial to answer "what work was done for card TRELLO-002?" by running `git log --grep="TRELLO-002"`.

The second mechanism is **automated card description**: when the CI creates a Trello card (triggered by PR open), it stores the PR URL in the card's description. All subsequent CI jobs (move to In Progress, move to Review/QA, move to Done) use this PR URL to locate the correct card in the Trello API. This means the card is the authoritative record of what was built, and the PR is the authoritative record of how it was built — and the two are permanently linked in both directions.

The reason this correlation matters is **traceability and accountability**. In a professional engineering team, it must be possible to answer: "Why does this line of code exist?" The answer should trace back through a commit, to a PR, to a Trello card, to a business requirement or bug report. Without this chain, codebases accumulate unexplained decisions that nobody dares to change. With it, the cost of understanding any change is minimal. This is the core principle of GitOps: Git is not just a backup tool, it is the operational record of every decision the team has made.
