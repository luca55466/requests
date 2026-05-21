# SQA Analysis: `requests` — "HTTP for Humans"

**Question answered:** How does the project perform Software Quality Assurance?
**Project:** [`psf/requests`](https://github.com/psf/requests) — Python HTTP client library
**Scale:** ~300 M downloads/week · used by 4 M+ repositories
**Presentation time:** ≤ 3 minutes

---

## The Answer in One Sentence

> The `requests` project performs QA through **three enforced layers**: static analysis (Verification), an automated test matrix (Validation), and a structured peer-review process — all gated by GitHub Actions CI on every single commit and pull request.

---

## Layer 1 — Static Analysis (Verification)

> **SQA concept:** Verification = "Are we building the product right?" — checking code *without* running it.

Three workflows run automatically on **every push and every PR**:

### 1a. Linting & Formatting — `lint.yml`

```yaml
# .github/workflows/lint.yml
name: Lint code
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-24.04
    steps:
    - uses: actions/checkout@...
    - name: Run pre-commit          # <-- enforces all hooks below
      uses: pre-commit/action@...
```

The `pre-commit` hook chain (`.pre-commit-config.yaml`):

```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  hooks:
  - id: check-merge-conflict        # block accidental merge markers
  - id: check-toml                  # validate config files
  - id: trailing-whitespace
  - id: end-of-file-fixer

- repo: https://github.com/astral-sh/ruff-pre-commit
  hooks:
    - id: ruff-check                # linter: PEP8, unused imports, debugger calls
      args: [--fix]
    - id: ruff-format               # formatter: Black-compatible style
```

Ruff rules configured in `pyproject.toml`:

```toml
# pyproject.toml
[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors (PEP 8)
    "W",    # pycodestyle warnings
    "F",    # pyflakes (undefined names, unused imports)
    "I",    # isort (import ordering)
    "UP",   # pyupgrade (modernize syntax)
    "T10",  # flake8-debugger (no leftover breakpoints)
]
```

---

### 1b. Type Checking — `typecheck.yml`

```yaml
# .github/workflows/typecheck.yml
name: Type Check
on: [push, pull_request]

jobs:
  typecheck:
    strategy:
      matrix:
        python-version: ["3.10", "3.14"]   # oldest + newest supported
    steps:
    - name: Run pyright
      run: python -m pyright src/requests/
```

Configured in **strict mode** — every function signature must be fully typed:

```toml
# pyproject.toml
[tool.pyright]
include = ["src/requests"]
typeCheckingMode = "strict"           # highest bar — no untyped code allowed
```

---

### 1c. Security Scanning — `codeql-analysis.yml`

```yaml
# .github/workflows/codeql-analysis.yml
name: "CodeQL"
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 23 * * 0'    # also runs weekly even without new code

steps:
  - name: Perform CodeQL Analysis   # scans for known Python vulnerability patterns
    uses: github/codeql-action/analyze@...
```

**SQA takeaway:** No code reaches `main` without passing lint, type checks, and a security scan.

---

## Layer 2 — Dynamic Analysis (Validation)

> **SQA concept:** Validation = "Are we building the right product?" — running the software and checking behaviour.

### 2a. Automated Test Matrix — `run-tests.yml`

```yaml
# .github/workflows/run-tests.yml
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13", "3.14",
                         "3.14t",         # free-threaded Python
                         "3.15-dev",      # unreleased future version
                         "pypy-3.11"]
        os: [ubuntu-22.04, macOS-latest, windows-latest]
    steps:
    - name: Run tests
      run: make ci         # pytest with JUnit XML output
```

**Result: 8 Python versions × 3 OS = up to 23 combinations per PR**
(PyPy excluded on Windows due to an OpenSSL/Rust constraint — even that exclusion is documented inline.)

Additional edge-case jobs in the same file:
- `no_chardet` — tests without the optional character-detection dependency
- `urllib3` — tests with the older `urllib3 1.x` (backward compatibility)

### 2b. Test Suite Scope

```
tests/
  test_requests.py     106 KB   ← main integration suite
  test_utils.py         31 KB   ← utility functions
  test_lowlevel.py      15 KB   ← raw socket/TLS behaviour
  test_testserver.py   5.5 KB   ← tests against a real HTTP server
  test_structures.py   2.7 KB
  test_hooks.py        0.4 KB
  test_adapters.py     0.3 KB
  testserver/                   ← embedded HTTP server for live request tests
  certs/                        ← real TLS certificates (incl. expired, mTLS)
```

Tests also run **doctests** from the source code itself (`--doctest-modules` in pytest config):

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--doctest-modules"    # code examples in docstrings are verified
testpaths = ["tests"]
```

### 2c. Coverage Measurement

```ini
# .coveragerc
[run]
omit = requests/packages/*    # exclude vendored third-party code
```

```makefile
# Makefile (excerpt)
coverage:
    pytest --cov-report term --cov-report xml --cov=requests tests/
```

Coverage XML output enables integration with tools like Codecov for trend tracking.

**SQA takeaway:** Every PR is validated against the real world — multiple Python runtimes, real HTTP traffic, real TLS handshakes.

---

## Layer 3 — Peer Review & Process Controls

> **SQA concept:** Process controls ensure quality is a team discipline, not an individual responsibility.

### 3a. Structured Issue Reporting

`.github/ISSUE_TEMPLATE/` provides separate forms for:
- **Bug reports** — structured fields for reproduction steps, expected vs actual behaviour
- **Feature requests** — rationale, alternatives considered
- **Custom** — for anything else

This is the project's equivalent of a **defect taxonomy / problem reporting procedure**.

### 3b. Formal Security Vulnerability Process (`SECURITY.md`)

```
Timeline (from SECURITY.md):
  - Within 2 days:   Receipt confirmed
  - Within 2 weeks:  Fix released
  - Release day:     CVE issued, patch published to PyPI,
                     downstream packagers notified in advance
                     (Red Hat, Debian Maintainer)
```

This is a documented **corrective action procedure** for security defects.

### 3c. Contributor Guide (`docs/dev/contributing.rst`)

Covers:
- Code style requirements (enforced by Ruff — not just guidelines)
- Testing requirements before submitting a PR
- The PR review workflow and maintainer expectations

### 3d. Automated Dependency Updates (`dependabot.yml`)

```yaml
# .github/dependabot.yml
updates:
  - package-ecosystem: "github-actions"   # keeps CI actions patched
    schedule: { interval: "weekly" }
  - package-ecosystem: "pip"              # keeps pre-commit hooks updated
    schedule: { interval: "weekly" }
```

**SQA takeaway:** Quality is enforced structurally — CI gates block merges, templates standardise defect reports, and security gets a documented SLA.

---

## Summary Slide

| SQA Layer | What they do | Tool / File |
|-----------|-------------|-------------|
| **Static Analysis** | Lint, format, type-check every commit | `lint.yml`, `typecheck.yml`, Ruff, Pyright |
| **Security Scan** | Weekly + per-PR vulnerability analysis | `codeql-analysis.yml`, `zizmor.yml` |
| **Dynamic Testing** | 23-combination pytest matrix | `run-tests.yml`, `tests/` |
| **Coverage** | pytest-cov + XML report | `.coveragerc`, `Makefile` |
| **Peer Review** | Contributor guide + enforced CI gates | `CONTRIBUTING.md`, `contributing.rst` |
| **Defect Reporting** | Structured issue templates | `.github/ISSUE_TEMPLATE/` |
| **Security SLA** | Formal disclosure + 2-week fix target | `SECURITY.md` |

**Bottom line:** `requests` has no separate "QA team" — the process *is* the QA. Every tool, every workflow, and every template works together so that no defect can merge undetected.
