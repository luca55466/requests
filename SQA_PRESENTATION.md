# SQA Analysis: `requests` — "HTTP for Humans"

**Question answered:** How does the project perform Software Quality Assurance?
**Project:** `psf/requests` — Python HTTP client library
**Presentation time:** ≤ 3 minutes · 3 slides

---

# SLIDE STRUCTURE

---

## SLIDE 1 — The Three QA Layers
**Layout:** Full-width content slide. Title + three equal columns, each with a header, one-line definition, and 2–3 bullet points.

### On-Slide Content

**Slide title:**
> QA in `requests`: Three Enforced Layers

**Three equal columns:**

| 🔍 Verification | ✅ Validation | 📋 Process Controls |
|---|---|---|
| *Check code without running it* | *Run the software — does it behave correctly?* | *Make quality a team discipline* |
| Ruff — lint & format | pytest across **23 environments** | Structured issue templates |
| Pyright — strict type checking | Real HTTP server in tests | Security vulnerability SLA |
| CodeQL — security scan | Coverage measured with pytest-cov | Enforced contributor guide |

**Bottom line (bold, centred):**
> All three are automated and gate every pull request — no human can bypass them.

> _Hint for PowerPoint: three equal rectangle shapes or SmartArt "Three-Process"._

---

### Presenter Notes — Slide 1 (≈ 30 sec)

> "The answer to how `requests` does QA comes down to three layers — and what makes them interesting is that all three are automated and enforced on every pull request. No one can merge code that fails them.
>
> Verification checks the code without running it — static analysis. Validation actually runs the software across 23 different environments. And Process Controls are the structural things: how bugs are reported, how security issues are handled, how reviews are conducted. Let me go through each."

---
---

## SLIDE 2 — Verification & Validation (The Technical Evidence)
**Layout:** Two-column content slide. Left column = Verification. Right column = Validation.

### On-Slide Content

**Slide title:**
> Layers 1 & 2: Static Analysis + Automated Testing

---

**LEFT COLUMN — "Verification (Static Analysis)"**

_Subtitle line:_ Every push & PR triggers three automated checks:

**Bullet 1 — Linting & Formatting**
`lint.yml` → runs **Ruff** via pre-commit
- Checks PEP 8, unused imports, leftover debugger calls
- Auto-fixes style before code can be merged

_Code snippet (small font, code block):_
```yaml
# .github/workflows/lint.yml
on: [push, pull_request]
steps:
  - name: Run pre-commit   # enforces Ruff lint + format
    uses: pre-commit/action@...
```

**Bullet 2 — Type Checking**
`typecheck.yml` → runs **Pyright** in **strict mode**
- Every function must have full type annotations
- Runs on Python 3.10 AND 3.14

_Code snippet:_
```toml
# pyproject.toml
[tool.pyright]
typeCheckingMode = "strict"  # highest possible bar
```

**Bullet 3 — Security Scan**
`codeql-analysis.yml` → **CodeQL** scans for Python vulnerability patterns
- Runs on every PR *and* weekly on a schedule

---

**RIGHT COLUMN — "Validation (Dynamic Testing)"**

_Subtitle line:_ `run-tests.yml` — pytest across a full environment matrix:

_Visual: small table (bold the numbers)_

| | Ubuntu | macOS | Windows |
|---|---|---|---|
| Python 3.10–3.14 | ✓ | ✓ | ✓ |
| Python 3.15-dev | ✓ | ✓ | ✓ |
| PyPy 3.11 | ✓ | ✓ | — |
| **Total** | | | **≈ 23 combinations** |

_Code snippet:_
```yaml
# .github/workflows/run-tests.yml
matrix:
  python-version: ["3.10","3.11","3.12","3.13",
                   "3.14","3.14t","3.15-dev","pypy-3.11"]
  os: [ubuntu-22.04, macOS-latest, windows-latest]
```

**Test suite scope:**
```
test_requests.py   106 KB  ← main integration suite
test_utils.py       31 KB
test_lowlevel.py    15 KB  ← raw TLS/socket behaviour
testserver/                ← real HTTP server, live requests
```
- Doctests in source code are also verified (`--doctest-modules`)
- Coverage measured with `pytest-cov` → XML report

---

### Presenter Notes — Slide 2 (≈ 1 min 30 sec)

> "Let's start with **Verification** — the left side. Verification means checking the code is built correctly, without even running it.
>
> Every time someone opens a pull request, three GitHub Actions workflows kick off automatically. First, `lint.yml` runs a tool called Ruff through a pre-commit hook — it enforces PEP 8 style, catches unused imports, and even blocks leftover debug statements. The code literally cannot be merged if this fails.
>
> Second, `typecheck.yml` runs Pyright in *strict* mode. Strict mode means every single function parameter and return value must have a type annotation — that's the highest bar Pyright offers.
>
> Third, CodeQL does a security scan — scanning for known Python vulnerability patterns. And it doesn't just run on PRs, it runs on a weekly schedule too, even if no code changed.
>
> Now **Validation** — the right side. Validation is about running the software and checking it actually behaves correctly. Their pytest suite runs across a matrix: 8 Python versions times 3 operating systems — that's about 23 combinations per PR. They even test against Python 3.15, which hasn't been released yet. The main test file alone is 106 kilobytes. They spin up a real HTTP server during tests — so they're making actual HTTP requests, not just mocking everything."

---
---

## SLIDE 3 — Process Controls & Summary
**Layout:** Content slide — top half: three-column process strip. Bottom half: summary table.

### On-Slide Content

**Slide title:**
> Layer 3: Process Controls — Quality as a Team Discipline

---

**Three-column strip (top half of slide):**

**Column 1 — Structured Defect Reporting**
`.github/ISSUE_TEMPLATE/`
- Separate templates for bugs vs. feature requests
- Forces reporters to document: steps to reproduce, expected vs. actual behaviour
- _SQA equivalent: defect taxonomy / problem report form_

**Column 2 — Security Vulnerability SLA**
`.github/SECURITY.md`
```
≤ 2 days  → acknowledgement
≤ 2 weeks → fix released + CVE issued
Release   → PyPI patch + Red Hat & Debian notified
```
- _SQA equivalent: corrective action procedure_

**Column 3 — Contributor & Review Process**
`docs/dev/contributing.rst`
- Documents code style, testing requirements, PR workflow
- Style rules are *machine-enforced* by Ruff — not just guidelines
- Dependabot opens weekly PRs to keep CI dependencies patched

---

**Summary table (bottom half of slide):**

| SQA Layer | Tool / File | What it enforces |
|-----------|------------|-----------------|
| Linting & Formatting | `lint.yml` · Ruff | Style, imports, no debugger calls |
| Type Checking | `typecheck.yml` · Pyright strict | Full type-safety |
| Security Scan | `codeql-analysis.yml` · CodeQL | Vulnerability patterns |
| Automated Testing | `run-tests.yml` · pytest | ≈ 23 environment combos |
| Coverage | `.coveragerc` · pytest-cov | Measured, XML output |
| Defect Reporting | Issue templates | Structured bug reports |
| Security SLA | `SECURITY.md` | 2-day ACK, 2-week fix |

---

**Closing line (bottom of slide, bold, centred):**
> No separate QA team — the CI pipeline *is* the QA system.

---

### Presenter Notes — Slide 3 (≈ 1 min)

> "The third layer is about process — making quality a team discipline rather than an individual's job.
>
> When someone finds a bug, they don't just write it in any format they like. GitHub issue templates force them to fill in a structured form — separate forms for bugs versus feature requests, with fields for reproduction steps and expected behaviour. That's essentially a defect reporting procedure.
>
> For security specifically, they have a documented SLA in SECURITY.md: two days to acknowledge, two weeks to ship a fix. The fix gets a CVE number and the maintainers of Red Hat and Debian are notified *before* the public release. That's a formal corrective action procedure.
>
> And the contribution guide isn't just a suggestion — code style is machine-enforced by Ruff, so a reviewer can't accidentally wave through a style violation.
>
> To wrap up: `requests` has no dedicated QA team. Instead, the CI pipeline *is* the QA system. Every commit is verified statically, validated dynamically across 23 environments, and reviewed through a structured process. That is modern, professional SQA in practice."

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
