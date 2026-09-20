# GitHub Actions Practical Exam — Reusable Step-by-Step Guide

> **Purpose:** This repository is a reusable reference for the GitHub Actions practical exam.
>
> The exam is expected to be **similar in workflow to the practice exercise**: the scenario, repository, application, test files, branch names, thresholds, or exact requirements may change, but the **core GitHub Actions flow is expected to remain largely the same**.
>
> **Do not memorize commands or YAML.** The exam is guided. Use the instructions given in the exam and use this README to understand **what to do, where to do it, and what to verify**.

---

## 1. The Core Exam Flow — Remember This

Almost every similar question can be approached like this:

```text
Given repository / existing repo
        ↓
Open in GitHub Codespaces
        ↓
Read the guided requirements carefully
        ↓
Inspect existing files
        ↓
Run the existing tests
        ↓
Run coverage if required
        ↓
Create CI workflow(s)
        ↓
Trigger workflow using a Pull Request
        ↓
Check GitHub Actions result
        ↓
If coverage/security/test requirement exists → enforce it
        ↓
Protect the target branch
        ↓
Intentionally observe / diagnose a failure if requested
        ↓
Fix the actual problem
        ↓
Push fix to the same branch
        ↓
Actions run again
        ↓
All required checks GREEN
        ↓
Merge Pull Request
```

### The mental model

```text
CODE
 ↓
PULL REQUEST
 ↓
GITHUB ACTIONS
 ↓
TESTS / COVERAGE / OTHER CHECKS
 ↓
PASS or FAIL
 ↓
BRANCH PROTECTION
 ↓
MERGE only when required checks pass
```

---

# 2. First 2 Minutes of the Exam: Read the Question Like a Checklist

Before touching code, identify these things from the guided instructions.

| Question item | What you must identify |
|---|---|
| Repository | Existing repo or new repo? |
| Environment | Codespace or local? |
| Language | Python / Node / Java / etc. |
| Test command | e.g. `pytest`, `npm test`, etc. |
| Dependency file | `requirements.txt`, `package.json`, etc. |
| Target branch | Usually `main`, but read the question |
| Trigger | PR? push? schedule? manual? |
| Number of workflows | One or more? |
| Coverage threshold | e.g. 80%, 85%, etc. |
| Branch protection | Required? Which checks? |
| Failure demonstration | Required? What failure? |
| Final action | Merge? Report? Screenshot? |
| PR description | Required? What must it explain? |

### Very important

Do **not** assume the values from this README are always correct.

For example:

```text
This README says 85%
Exam could say 80%

This README says main
Exam could say master/develop/main

This README uses pytest
Exam could use npm test
```

Always follow the **exam's exact requirement first**. Use this README for the process.

---

# 3. STEP 1 — Open / Inspect the Repository

If the exam gives an existing repository:

1. Open the repository.
2. Open it in **GitHub Codespaces**.
3. Look at the Explorer.
4. Read the README/instructions if provided.
5. Identify the source folder, tests, dependency file, and existing workflows.

Typical structure:

```text
project/
├── src/
├── tests/
├── requirements.txt       # or package.json, etc.
└── .github/
    └── workflows/
```

### Do NOT immediately start changing files

First understand:

- What is already present?
- What tests already exist?
- Which tests are disabled/commented?
- Are workflows already present?
- What does the question actually ask you to add/change?

---

# 4. STEP 2 — Run the Existing Tests First

This is the baseline.

For Python/pytest examples:

```bash
pip install -r requirements.txt
python -m pytest
```

If coverage is required:

```bash
python -m pytest --cov=src --cov-report=term-missing
```

### What to record

Write down:

```text
Tests passed: ______
Tests failed: ______
Coverage: ______%
Untested / missing functions or lines: ______
```

### Why?

The baseline tells you whether the repository already has a problem and what must be fixed later.

---

# 5. STEP 3 — Git Branch Strategy

If the exam asks you to work through a Pull Request, normally use a feature branch.

Example:

```bash
git checkout -b test-ci
```

or use whatever branch name the instructions request.

Then make the requested change, commit, and push:

```bash
git add .
git commit -m "Describe the change"
git push -u origin test-ci
```

### Typical flow

```text
main
  ↑
  | Pull Request
  |
feature/test branch
```

Do not work directly on `main` when the exercise is specifically teaching PR-based CI.

---

# 6. STEP 4 — Workflow A: Run Tests on Pull Requests

A very common requirement is:

> Run tests whenever a Pull Request targets `main`.

Typical Python/pytest workflow:

```yaml
name: Run Tests

on:
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: python -m pytest
```

## Understand the structure, don't memorize it

```text
name
 ↓
on / trigger
 ↓
jobs
 ↓
runs-on
 ↓
steps
 ↓
checkout
 ↓
setup language
 ↓
install dependencies
 ↓
run tests
```

### Key concept

```yaml
on:
  pull_request:
    branches:
      - main
```

means the workflow is intended to run for Pull Requests targeting `main`.

If the question says the workflow should NOT run on direct pushes to `main`, do not add a `push` trigger unless the instructions explicitly ask for it.

---

# 7. STEP 5 — Create the Pull Request

After pushing your branch:

```text
base: main
compare: your-branch
```

Create the PR.

Then open the **Checks / Actions** area.

You should be able to see the test workflow run.

Expected successful state:

```text
Run Tests ✅
```

If it fails:

```text
PR
 ↓
Checks
 ↓
failed workflow
 ↓
failed job
 ↓
failed step
 ↓
logs
```

Never guess what failed. Read the log.

---

# 8. STEP 6 — Workflow B: Coverage Gate

A common second requirement is:

> Run tests with coverage and fail the workflow if coverage is below a threshold.

For Python, an example is:

```yaml
name: Coverage Check

on:
  pull_request:
    branches:
      - main

jobs:
  coverage:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests with coverage
        run: python -m pytest --cov=src --cov-report=term-missing --cov-fail-under=85
```

## The important idea

```text
Coverage >= required threshold → PASS
Coverage < required threshold  → FAIL
```

The threshold must come from the exam question.

For example, if the question says 80%, use 80 instead of 85.

---

# 9. How to Read a Coverage Failure

Typical failure:

```text
ERROR: Coverage failure: total of 52 is less than fail-under=85

TOTAL ... 52%

Error: Process completed with exit code 1
```

Translate it mentally:

```text
Tests may have passed
        ↓
But coverage requirement failed
        ↓
Workflow must fail
```

This is different from a functional test failure.

### Two different kinds of failures

```text
Test failure:
assertion / code behavior problem

Coverage failure:
not enough code is exercised by tests
```

---

# 10. STEP 7 — Protect the Target Branch

If the question says:

> PR must not be mergeable while the coverage check is failing.

then protect the target branch.

Current GitHub UI may use **Rulesets**.

Typical route:

```text
Repository
 → Settings
 → Rules
 → Rulesets
 → New branch ruleset
```

### Basic configuration

```text
Ruleset name: Protect main
Enforcement: Active
Target branch: main
```

Then enable the required rules requested by the question.

Common exam configuration:

```text
✅ Require a pull request before merging
✅ Require status checks to pass before merging
```

Then add the required check.

For the example workflow:

```yaml
jobs:
  coverage:
```

The status check/job you may need to require is typically:

```text
coverage
```

### Important

GitHub's UI can change. The exact menu labels may differ slightly.

The concept does not change:

```text
Target branch
        ↓
PR required
        ↓
Required status check
        ↓
Check fails
        ↓
Merge blocked
```

---

# 11. STEP 8 — Verify Branch Protection, Don't Assume It

This is a very important practical habit.

Open the PR.

If the required check is failing, verify that merge is actually blocked.

Expected:

```text
Tests          ✅
Coverage       ❌

Required check failed
        ↓
Merge blocked
```

If the question specifically asks you to test the protection, **do not just create the rule and assume it works**. Check the actual PR state.

---

# 12. STEP 9 — Demonstrate a Failure When the Question Asks for It

A common exam pattern is:

1. Create a branch.
2. Make a deliberately requested change.
3. Open a PR.
4. Wait for CI to fail.
5. Inspect logs.
6. Find the actual cause(s).
7. Fix them.
8. Push again.
9. Verify checks become green.
10. Merge.

### Do not fix before observing the failure

If the question says:

> Confirm that the pipeline catches the problem.

then first let it fail and inspect the failure.

The point is to demonstrate that the CI setup works.

---

# 13. STEP 10 — Diagnose the Failure

Use this method every time:

```text
PR
 ↓
Checks
 ↓
Failed workflow
 ↓
Failed job
 ↓
Failed step
 ↓
Read logs
 ↓
Identify exact cause
 ↓
Fix ONLY what is necessary
 ↓
Run locally if possible
 ↓
Commit
 ↓
Push
```

### Example: test failure

```text
AssertionError
```

Look at:

- expected value
- actual value
- test function
- source function

### Example: import error

```text
ModuleNotFoundError
```

Check:

- project structure
- import path
- package setup
- command used to run tests

### Example: indentation/syntax error

```text
IndentationError
SyntaxError
```

Fix the exact formatting/syntax problem first.

### Example: coverage failure

```text
Coverage = 52%
Required = 85%
```

Identify uncovered code and add meaningful tests.

---

# 14. STEP 11 — Increase Coverage Properly

Do not simply make random tests.

Look at:

```text
Name                       Stmts   Miss  Cover   Missing
--------------------------------------------------------
src/contact_validator.py      23      4    83%   15, 23, 26, 35
```

The `Missing` column tells you where execution has not happened.

Then add tests that exercise those paths.

### Good test strategy

For a function such as:

```python
def is_valid_phone(phone):
    if not isinstance(phone, str):
        raise TypeError(...)
```

You may need tests for both:

```text
valid input
invalid / error input
```

For a function with multiple branches:

```text
if condition
else condition
```

try to exercise both sides when the question requires coverage.

---

# 15. STEP 12 — Run the Same Kind of Check Locally

Before pushing a fix, run the same style of check locally whenever possible.

Example:

```bash
python -m pytest
```

and:

```bash
python -m pytest --cov=src --cov-report=term-missing
```

If the CI uses a hard threshold:

```bash
python -m pytest --cov=src --cov-report=term-missing --cov-fail-under=85
```

### Goal

Do not rely on GitHub to discover obvious local mistakes.

---

# 16. STEP 13 — Push the Fix and Wait for CI

```bash
git add .
git commit -m "Fix tests and coverage"
git push
```

The existing PR should update automatically.

Expected:

```text
Run Tests       ✅
Coverage Check  ✅
```

If the branch protection requires coverage:

```text
Coverage ✅
     ↓
Merge becomes available
```

---

# 17. STEP 14 — PR Description

If the question asks for an explanation in the PR description, cover these two things:

### A. What was wrong?

Explain the actual issue in your own words.

### B. Why did CI catch it automatically?

Explain that the PR triggered the workflow and that required checks/thresholds prevented merging until the condition was satisfied.

### Reusable template

```text
The initial repository had [describe the actual problem].

I fixed the issue by [describe the actual fix].
I also added/updated tests to cover [describe the uncovered behavior].

The CI workflows run automatically for the pull request.
The required checks verify [tests/coverage/etc.], and the branch protection rule prevents merging until the required checks pass.
```

**Replace the placeholders with the actual problem. Do not invent a failure that did not occur.**

---

# 18. STEP 15 — Final Merge

Only merge after the question's required checks are satisfied.

Final state should normally look like:

```text
Required checks
    ✅

Coverage
    ✅

Tests
    ✅

PR
    Ready to merge
```

Then:

```text
Merge Pull Request
        ↓
Confirm merge
```

---

# 19. Universal Exam Checklist

Before you finish, check these:

```text
[ ] I understood the target branch.
[ ] I used Codespaces if required.
[ ] I inspected the existing repo first.
[ ] I ran the baseline tests.
[ ] I recorded baseline coverage if requested.
[ ] I created the required GitHub Actions workflow(s).
[ ] I used the correct trigger from the question.
[ ] I created a PR when the question requires PR-based CI.
[ ] I verified Actions actually ran.
[ ] I verified failures when the question asks for a failure demonstration.
[ ] I read the failed logs instead of guessing.
[ ] I configured branch protection if required.
[ ] I added the correct required status check.
[ ] I fixed the real problem.
[ ] I reran tests locally where possible.
[ ] I pushed the fix.
[ ] All required checks are green.
[ ] The PR is mergeable.
[ ] I added the requested PR explanation.
[ ] I merged the PR if the question requires it.
```

---

# 20. If the Question Changes — How to Adapt

The **process** stays similar, but these details may change:

### If the language changes

Replace:

```text
Python + pytest
```

with whatever the question specifies, such as:

```text
Node.js + npm test
Java + Maven
```

The Actions structure is still:

```text
checkout
→ setup environment
→ install dependencies
→ run tests
```

### If the coverage percentage changes

Use the exact threshold from the question.

### If the source/test folder changes

Use the exact path from the question.

### If the target branch changes

Use the exact branch named in the question.

### If there are more workflows

Create the additional workflow(s) using the same pattern:

```text
trigger
→ job
→ environment
→ dependencies
→ required command
→ pass/fail condition
```

### If security / lint / build is added

Think:

```text
PR
 ↓
GitHub Actions
 ↓
Build / Test / Lint / Security / Coverage
 ↓
Required checks
 ↓
Merge protection
```

---

# 21. Common Mistakes

## Mistake 1 — Running `pytest` and getting 0 tests

Check:

```text
Is the test file present?
Is it named correctly?
Does it contain test_... functions?
```

For Python import issues, trying:

```bash
python -m pytest
```

can be useful instead of plain `pytest`.

---

## Mistake 2 — Coverage says 100% but no tests ran

If you see:

```text
collected 0 items
```

do not treat the coverage number as meaningful.

No tests executed = no useful baseline.

---

## Mistake 3 — Coverage workflow runs on push when it should only run on PR

Check the trigger.

For PR-only behavior:

```yaml
on:
  pull_request:
    branches:
      - main
```

Do not add:

```yaml
push:
```

unless the question asks for it.

---

## Mistake 4 — Coverage is displayed but the workflow still passes below the threshold

Reporting coverage is not the same as enforcing coverage.

You need an actual failure condition, such as:

```text
--cov-fail-under=<threshold>
```

or another explicit enforcement mechanism requested by the question.

---

## Mistake 5 — Required status check added incorrectly

Check the workflow/job name shown by GitHub.

For example:

```yaml
jobs:
  coverage:
```

means the job/check may appear as:

```text
coverage
```

Do not guess the check name; look at the actual PR Checks/Actions page.

---

## Mistake 6 — Merge is still available while a required check is failing

Check:

```text
Target branch correct?
Ruleset active?
PR required?
Required status check added?
Correct check/job selected?
```

Then refresh the PR.

---

## Mistake 7 — Fixing a failure before observing it

If the question says to demonstrate that CI catches the issue:

```text
First observe failure
→ read logs
→ diagnose
→ fix
```

---

# 22. Quick Git Command Reference

These are here as a reference because the exam is guided. **Do not treat them as memorization requirements.**

### Check status

```bash
git status
```

### Create a branch

```bash
git checkout -b <branch-name>
```

### Stage files

```bash
git add .
```

### Commit

```bash
git commit -m "Describe the change"
```

### Push first time

```bash
git push -u origin <branch-name>
```

### Push later changes

```bash
git push
```

### Switch branch

```bash
git checkout <branch-name>
```

---

# 23. Quick GitHub Actions Cheat Sheet

### Workflow file location

```text
.github/workflows/<workflow-name>.yml
```

### Typical flow

```yaml
name: ...

on:
  pull_request:
    branches:
      - main

jobs:
  ...
```

### Python CI building blocks

```yaml
uses: actions/checkout@v4
```

```yaml
uses: actions/setup-python@v5
```

```yaml
run: pip install -r requirements.txt
```

```yaml
run: python -m pytest
```

Coverage example:

```yaml
run: python -m pytest --cov=src --cov-report=term-missing --cov-fail-under=85
```

Again: **copy/adapt from the exam requirement; do not blindly reuse values.**

---

# 24. If You Use AI During Preparation / Allowed Practical Work

Give the AI the **exercise text and the current state/output**.

A good prompt is:

```text
I am doing a guided GitHub Actions practical.

Do only ONE step at a time.
Do not jump ahead.
I will send you the current terminal output or GitHub screenshot after each step.

First inspect the requirement and tell me exactly what I should do next.
If there is an error, diagnose the exact error from the log.
Do not invent requirements.
```

When an Actions job fails, give the AI:

```text
1. The workflow YAML
2. The failed log
3. The current branch/PR state
```

Then ask:

```text
Tell me only the next step.
```

This keeps the process controlled and prevents changing too many things at once.

---

# 25. Final 30-Second Memory Trick

If the exam feels confusing, remember just this:

```text
REPO
 ↓
CODE / TESTS
 ↓
BASELINE
 ↓
WORKFLOW
 ↓
PR
 ↓
ACTIONS
 ↓
FAIL / PASS
 ↓
PROTECT BRANCH
 ↓
FIX
 ↓
RE-RUN
 ↓
GREEN
 ↓
MERGE
```

### The 5 questions to ask yourself

```text
1. What does the question want to run?
2. When should it run?
3. What makes it PASS or FAIL?
4. What branch must be protected?
5. What must be true before merge?
```

If you can answer those five, the rest is mostly following the guided instructions.

---

# 26. Practice Exercise Reference

The practice scenario used for this guide required, among other things:

- baseline test/coverage measurement,
- two PR-targeting-`main` CI workflows,
- an 85% coverage gate,
- protected `main` with a required coverage check,
- demonstrating and diagnosing a failure,
- adding/fixing tests,
- confirming green checks,
- and merging the PR.

The provided exercise also states that the exam itself is fully guided and that commands, menu paths, and YAML syntax are not expected to be memorized.

Use this README as a **process reference**, while the exam's actual guided instructions remain the source of truth for exact names, branches, thresholds, files, commands, and requirements.
