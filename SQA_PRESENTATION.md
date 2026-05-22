# SQA Analysis: `requests` — "HTTP for Humans"

**Question answered:** How does the project perform Software Quality Assurance?
**Project:** `psf/requests` — Python HTTP client library
**Presentation time:** ≤ 3 minutes · 3 slides

---

# SLIDE STRUCTURE

---

## SLIDE 1 — Verification (Static Analysis)
**Layout:** Title + two-column content. Left = bullet points. Right = two stacked screenshots.

### On-Slide Content

**Slide title:**
> Verification: Checking Code Without Running It

**Left column — bullets:**
- Three automated workflows fire on **every push and every PR**
- **Ruff** (`lint.yml`) — enforces PEP 8, catches unused imports, blocks leftover debugger calls, auto-formats code
- **Pyright** (`typecheck.yml`) — type-checks in **strict mode**: every function must be fully annotated. Runs on Python 3.10 AND 3.14
- **CodeQL** (`codeql-analysis.yml`) — scans for Python vulnerability patterns. Runs on every PR *and* on a **weekly schedule**

**Right column — screenshots (see below)**

---

### Screenshots for Slide 1

**Screenshot A — PRIMARY**
File: `pyproject.toml`, lines 83–117
_(open in VS Code or GitHub, crop to just these lines)_

```
[tool.ruff]
target-version = "py310"
...
[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # pyflakes
    "I",      # isort
    "UP",     # pyupgrade
    "T10",    # flake8-debugger
]
...
[tool.pyright]
include = ["src/requests"]
typeCheckingMode = "strict"
```

Why: One screenshot proves both tools (Ruff + Pyright) are configured, and `"strict"` is immediately readable at a glance.

---

**Screenshot B — SUPPORTING**
File: `.github/workflows/typecheck.yml`, full file (only 32 lines)

Why: Shows the CI matrix (`python-version: ["3.10", "3.14"]`) and the exact command `python -m pyright src/requests/` — makes it concrete that this runs automatically, not manually.

---

### Presenter Notes — Slide 1 (≈ 1 min)

> "The first layer is Verification — checking the code without running it, which in SQA terms means static analysis.
>
> Every single push or pull request automatically triggers three tools. Ruff is a linter and formatter — it checks code style, import ordering, and even blocks leftover breakpoints from debugging. If it fails, the PR cannot be merged.
>
> Pyright does type checking, and look at the config on the right — `typeCheckingMode = "strict"`. That's the highest setting Pyright has. Every single function parameter and return value must have a type annotation. No exceptions.
>
> And CodeQL runs a security scan — not only on every PR, but also on a weekly schedule even when no code has changed. So new vulnerability patterns are caught retroactively."

---
---

## SLIDE 2 — Validation (Dynamic Testing)
**Layout:** Title + left bullets + right screenshot (the matrix YAML is the hero visual).

### On-Slide Content

**Slide title:**
> Validation: Does the Software Actually Behave Correctly?

**Left column — bullets:**
- `run-tests.yml` runs **pytest** across a full environment matrix
- **8 Python versions** × **3 operating systems** = **≈ 23 combinations per PR**
- Includes Python **3.15-dev** (not yet released) and **PyPy**
- Tests run against a **real embedded HTTP server** — not mocked requests
- Coverage measured with `pytest-cov` → XML report

**Test suite size (small table or text block):**
```
test_requests.py   106 KB   ← main integration suite
test_utils.py       31 KB
test_lowlevel.py    15 KB   ← raw TLS/socket behaviour
```

**Right column — screenshot (see below)**

---

### Screenshots for Slide 2

**Screenshot A — PRIMARY**
File: `.github/workflows/run-tests.yml`, lines 1–21
_(crop to the `matrix:` block — the version list is the centrepiece)_

```yaml
name: Tests
on: [push, pull_request]
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13",
                         "3.14", "3.14t", "3.15-dev", "pypy-3.11"]
        os: [ubuntu-22.04, macOS-latest, windows-latest]
```

Why: The version list is visually striking — 8 versions, 3 OSes in one block. The audience can count them. `"3.15-dev"` in particular makes a strong point about how thorough this is.

---

**Screenshot B — SUPPORTING**
File: `tests/` directory — open in VS Code file explorer or GitHub file browser

Show the folder listing including file sizes:
```
test_requests.py     (106 KB)
test_utils.py         (31 KB)
test_lowlevel.py      (15 KB)
testserver/
certs/
```

Why: Makes the *volume* of tests tangible. `testserver/` and `certs/` show they test against real network infrastructure, not mocks.

---

### Presenter Notes — Slide 2 (≈ 1 min)

> "Validation is the second layer — this means actually running the software and verifying it behaves correctly. Where Verification was about reading the code, Validation is about executing it.
>
> Look at the matrix on the right. Their test workflow runs pytest across 8 Python versions and 3 operating systems — that's 23 combinations on every single pull request. They even test against Python 3.15, which hasn't been officially released yet.
>
> And they're not faking it — the `testserver/` folder in the test suite is a real embedded HTTP server. Tests make actual HTTP requests, with real TLS certificates. The main test file is 106 kilobytes of test cases.
>
> Coverage is also measured and reported as XML — so there's a traceable quality metric, not just a feeling that tests exist."

---
---

## SLIDE 3 — Process Controls
**Layout:** Title + two screenshots side by side + one bullet line underneath each.

### On-Slide Content

**Slide title:**
> Process Controls: Quality as a Team Discipline

**Two screenshots side by side (see below), each with a 1-line caption underneath**

**Caption under Screenshot A:**
> Structured bug report template — forces every defect to be documented consistently

**Caption under Screenshot B:**
> Documented security SLA — 2-day acknowledgement, 2-week fix, CVE issued

**Bottom line (full width, bold):**
> Style rules aren't guidelines — Ruff enforces them in CI. No reviewer can wave through a violation.

---

### Screenshots for Slide 3

**Screenshot A — PRIMARY**
File: `.github/ISSUE_TEMPLATE/Bug_report.md`, full file (37 lines)
_(open on GitHub — it renders as a form preview which looks cleaner than raw markdown)_

```
## Expected Result
## Actual Result
## Reproduction Steps
   import requests ...
## System Information
   $ python -m requests.help
```

Why: This is the project's defect reporting procedure. The structure (Expected / Actual / Steps / System Info) maps directly to SQA defect taxonomy. Very readable, no explanation needed.

---

**Screenshot B — PRIMARY**
File: `.github/SECURITY.md`, lines 31–46 (the "Timeline" section)

```
### Timeline

When you report an issue, one of the project members will respond
to you within two days at the outside. ...

Our goal is to have a fix for any vulnerability released within
two weeks of the initial disclosure. ...
```

Why: A written SLA for defect resolution is exactly what a formal SQA corrective action procedure looks like. The two concrete numbers (2 days, 2 weeks) are immediately legible on a slide.

---

**Optional third screenshot (if space allows)**
File: `.github/ISSUE_TEMPLATE/` — directory view showing three template files:
`Bug_report.md`, `Feature_request.md`, `Custom.md`

Why: Shows that different defect types have separate forms — a taxonomy, not a free-text box.

---

### Presenter Notes — Slide 3 (≈ 1 min)

> "The third layer is process controls — the structures that make quality a team responsibility rather than one person's judgment call.
>
> On the left: the bug report template. When anyone reports a defect, they don't write a free-text email. GitHub forces them through this structured form — Expected Result, Actual Result, Reproduction Steps, System Info. That is a defect reporting procedure, directly equivalent to what a formal QMS would require.
>
> On the right: the security disclosure SLA from SECURITY.md. Two days to acknowledge, two weeks to ship a fix, CVE number issued, Red Hat and Debian notified before the public release. That's a documented corrective action procedure for security defects.
>
> And to tie it back to Verification — the contribution guide isn't just advice. Ruff enforces style automatically in CI, so no reviewer can accidentally approve a violation. The process is structurally enforced, not trust-based."

---

---

# REFERENCE MATERIAL
_(Not for slides — detailed evidence for questions from the audience)_

## All CI/CD workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `.github/workflows/lint.yml` | push, PR | Ruff lint + format via pre-commit |
| `.github/workflows/typecheck.yml` | push, PR | Pyright strict on Python 3.10 + 3.14 |
| `.github/workflows/run-tests.yml` | push, PR | pytest across 23 OS × Python combos |
| `.github/workflows/codeql-analysis.yml` | push to main, PR, weekly | Security vulnerability scan |
| `.github/workflows/zizmor.yml` | push, PR | Scans the CI workflows themselves for security issues |
| `.github/workflows/publish.yml` | release | Builds + publishes to PyPI with attestations |
| `.github/dependabot.yml` | weekly | Auto-updates GitHub Actions + pre-commit hooks |

## Full Ruff config (pyproject.toml lines 83–118)

```toml
[tool.ruff]
target-version = "py310"
src = ["src/requests", "tests"]

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes (undefined names, unused imports)
    "I",    # isort (import ordering)
    "UP",   # pyupgrade (modernise syntax)
    "T10",  # flake8-debugger (no leftover breakpoints)
]
ignore = ["E203", "E501", "UP031"]

[tool.ruff.format]
quote-style = "double"   # Black-compatible
indent-style = "space"

[tool.pytest.ini_options]
addopts = "--doctest-modules"
minversion = "6.2"
testpaths = ["tests"]

[tool.pyright]
include = ["src/requests"]
typeCheckingMode = "strict"
```

## Full pre-commit hook chain (.pre-commit-config.yaml)

```yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v6.0.0
  hooks:
  - id: check-case-conflict
  - id: check-merge-conflict
  - id: check-toml
  - id: check-yaml
  - id: end-of-file-fixer
  - id: mixed-line-ending
  - id: trailing-whitespace

- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.15.12
  hooks:
    - id: ruff-check
      args: [--fix]
    - id: ruff-format
      exclude: tests/test_lowlevel.py
```
