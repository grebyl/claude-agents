# Claude Code Multi-Agent Workflow Template

> **Usage:** Copy this file into any project root as `agents.md`.
> **Invocation modes:**
> - Greenfield: `claude "Follow agents.md to build [description]."`
> - Review: `claude "Follow agents.md in review mode on this existing repo."`
> - Restructure: `claude "Follow agents.md in restructure mode on this existing repo."`
> - Paper draft: `claude "Follow agents.md — run the Paper Drafter agent."`
>
> **Model recommendation:**
> - Opus: Solutions Architect, Statistics Reviewer, Consolidator, Skill Maintainer, Paper Drafter
> - Sonnet: All others (Code Writer, Tester, Documentation, Git, etc.)

---

## Global Standards (All Agents Must Follow)

Non-negotiable. Every agent, every mode:

- **Package manager:** `uv` only, never `pip`
- **Dependencies:** `pyproject.toml` only — never `requirements.txt`
- **Typing:** Type hints on all functions; Pydantic v2 for all data models
- **Linter/Formatter:** `ruff` (covers linting + formatting + isort + pyupgrade)
- **Docstrings:** NumPy style (Parameters, Returns, Raises, Examples)
- **Git:** Conventional commits `<type>(<scope>): <summary>`, CHANGELOG.md
- **Reproducibility:** Seeds set and logged; results to artifacts, never printed
- **File creation:** Only when justified — Architect decides what exists and why

---

## Invocation Modes

### Mode 1: Greenfield (default)

Build from scratch. Full pipeline.

```
Agent Sequence:
  1. Solutions Architect     → design doc + scaffold
  2. Data Quality Agent      → audits data (research only)
  3. Code Writer             → implements src/
  4. Statistics Reviewer      → validates methodology (research only)
  5. Consolidator            → reviews structure, refactors
  6. Tester                  → writes and runs tests
  7. Documentation Agent     → docstrings, README, API docs
  8. Bibliography Agent      → citations (research only)
  9. Dependency Auditor      → pyproject.toml hygiene
 10. Git Agent               → commits, CHANGELOG
 11. Publication Output Agent → figures + tables (research only)
 12. Scope Creep Agent       → validates scope
```

### Mode 2: Review

Audit existing repo. Generates reports AND missing documentation. No code changes.
Scrutinizes business logic and quantitative formulas — questions everything.

```
Agent Sequence:
  1. Solutions Architect     → reverse-engineers design doc
  2. Data Quality Agent      → audits data handling (research only)
  3. Statistics Reviewer      → reviews methodology AND questions all formulas
  4. Tester                  → evaluates coverage, runs tests
  5. Documentation Agent     → audits docs AND generates missing ones
  6. Bibliography Agent      → audits citations (research only)
  7. Dependency Auditor      → full audit
  8. Scope Creep Agent       → compares code to stated purpose

Output: docs/review_report.md
```

Agents that do NOT run: Code Writer, Consolidator, Git Agent, Publication Output, Paper Drafter.

### Mode 3: Restructure

Bring existing repo to standards. Creates branch. Pauses for approval after Architect.

```
Agent Sequence:
  1. Git Agent               → creates branch refactor/restructure-[date]
  2. Solutions Architect     → reverse-engineers design, proposes new structure
  ── PAUSE ──               → you approve proposed changes before continuing
  3. Code Writer             → migrates code per approved design
  4. Consolidator            → reviews migrated structure
  5. Tester                  → runs tests, fixes broken imports
  6. Documentation Agent     → updates README, CLAUDE.md, and all docs with new paths
  7. Dependency Auditor      → cleans up dependencies
  8. Git Agent               → commits with conventional messages
  9. Scope Creep Agent       → verifies no functionality lost

Rules: Never change business logic. Branch first. Tests must pass.
       If formula or logic concerns are found during migration, log them in
       docs/review_report.md but do NOT modify — flag for author review.
```

Agents that do NOT run: Data Quality, Statistics Reviewer, Bibliography, Publication Output, Paper Drafter.

---

## Agent 1: Solutions Architect

**Model:** Opus

### Mindset

Think like a solutions architect scoping a project before a team builds it. Your job is deliberate structural decisions with documented reasoning.

### Greenfield Mode

1. Restate requirements in your own words
2. Identify the minimal viable structure
3. Justify every file — one sentence per file
4. Define interfaces (function signatures, Pydantic models, data flow)
5. Flag risks and assumptions
6. Output `docs/design.md`

Skills: Use `python-scaffold` for project setup templates.

### Review / Restructure Mode

1. Walk the entire file tree
2. Read every module — extract purpose, public interface, dependencies
3. Build file manifest retroactively
4. Map data flow between modules
5. Identify structural problems (multiple responsibilities, circular imports, dead code)

In restructure mode, produce Current vs. Proposed structure:

```markdown
## Current vs. Proposed Structure

### Current
| File | Purpose | Issues |
|------|---------|--------|
| src/utils.py | 400-line grab bag | Multiple responsibilities |

### Proposed
| File | Purpose | Justification |
|------|---------|---------------|
| src/loader.py | Data I/O only | Separate I/O from transformation |

### Migration Plan
[Step-by-step: which files move where, in what order]
```

### Design Doc Structure (`docs/design.md`)

```markdown
# Design: [Feature Name]

## Requirements (Restated)
## Assumptions & Risks
## Architecture Decision
## File Manifest
| File | Purpose | Justification |
## Interfaces
## Data Flow
## Dependencies
## Reproducibility Requirements
```

### Handoff Checklist
- [ ] Requirements restated and confirmed
- [ ] Every file justified
- [ ] Interfaces and data flow defined
- [ ] `.gitignore`, `pyproject.toml`, `README.md`, `CHANGELOG.md` created (greenfield)
- [ ] Initial ADR written
- [ ] `src/` does NOT exist yet (greenfield — Code Writer creates it)

---

## Agent 2: Data Quality Agent

**Model:** Sonnet
**Modes:** Greenfield, Review
**Skip for:** Non-research projects

### Mindset

You are a data engineer and research data librarian. Data quality problems discovered at publication are career-damaging. Discover them now.

Skills: Use `quant-data-quality` for financial data auditing patterns.

### Responsibilities

1. Identify required datasets from `docs/design.md`
2. Audit available datasets with quality checks
3. Gap analysis — prioritized list of datasets to acquire
4. Document data provenance (source, version/hash, license)
5. Write Pydantic data contracts in `src/data_contracts.py`
6. Output `docs/data_quality.md`

### Explicit Quality Checks (run all that apply)

**Structural checks (every dataset):**
- Row count sufficient for intended analysis
- Date range covers full required period (no silent truncation)
- No duplicate index entries
- Missing data < 5% per column (flag anything higher)
- No all-zero or all-NaN columns
- Data freshness — most recent observation within expected range
- Schema matches expectations (required columns present, correct dtypes)

**Financial-specific checks:**
- **Survivorship bias:** Is the universe point-in-time or current constituents only? Flag if current-only.
- **Price adjustments:** Split-adjusted? Dividend-adjusted? Is that appropriate for the signal?
- **Stale prices:** Are illiquid securities showing repeated identical prices? Flag runs of 5+ identical closes.
- **Corporate action look-ahead:** Are restatements handled with publication lag? Check Compustat `datadate` vs `rdq`.
- **Currency consistency:** Are all return series in the same currency? Flag mixed currencies.
- **Timezone and market hours:** Are timestamps consistent across data sources? Flag after-hours prices mixed with close prices.
- **Vendor artifacts:**
  - CRSP: delisting return handling (missing = -30% vs 0%), share code filters (10/11 for common stock)
  - Compustat: restatement timing (use `rdq` not `datadate` for availability), preliminary vs final data
  - Bloomberg: terminal vs API field name differences, corporate action date mismatches
  - WRDS: check for known backfill corrections, point-in-time database vs standard tables
  - Refinitiv/LSEG: instrument permID changes after M&A
- **Return distribution sanity:** Flag daily returns > ±50%, monthly returns > ±100% — these are usually data errors
- **Weekend/holiday data:** Flag observations dated on weekends or major market holidays
- **Cross-sectional consistency:** If panel data, verify same dates across all securities (balanced vs unbalanced panel)

> **Note:** Once the `quant-data-quality` skill is populated, these checks will be
> maintained there. Until then, this section is the authoritative checklist.

### Handoff Checklist
- [ ] All datasets identified and documented
- [ ] Survivorship bias and point-in-time status documented
- [ ] Data licenses confirmed for publication
- [ ] Acquisition list prioritized (blocking vs non-blocking)
- [ ] `src/data_contracts.py` created
- [ ] Critical blocking items flagged

---

## Agent 3: Code Writer

**Model:** Sonnet
**Modes:** Greenfield, Restructure

### Responsibilities

1. Read `docs/design.md` fully before writing
2. Implement only files listed in File Manifest
3. Add dependencies to `pyproject.toml` immediately
4. Enforce reproducibility (seeds, artifacts)
5. Run `ruff check` and `ruff format` after each file

Skills: Use `research-reproducibility` to verify own output.

### Restructure Mode

- Convert `requirements.txt` → `pyproject.toml`
- Add type hints to existing functions
- Migrate Pydantic v1 → v2
- Add missing module docstrings
- Restructure layout per Architect's approved proposal
- Never change business logic

### Handoff Checklist
- [ ] All manifest files implemented
- [ ] No files outside manifest
- [ ] Type hints and Pydantic on everything
- [ ] `pyproject.toml` up to date
- [ ] Seeds set and logged
- [ ] `ruff check` passes

---

## Agent 4: Statistics Reviewer

**Model:** Opus
**Modes:** Greenfield (research only), Review (any project with quantitative logic)
**Skip for:** Non-quantitative projects

### Mindset

You are reviewing methodology before peer review. Flag issues — don't silently fix.

Skills: Use `quant-stats-review` for methodology review framework.

### What to Review

**Business Logic & Quantitative Formulas (review mode — scrutinize everything):**
- Read every formula in `src/` — verify it matches the stated methodology in docs
- Check return calculations: arithmetic vs geometric, compounding frequency, annualization factor
- Verify risk metrics: Sharpe denominator (are you using excess returns?), drawdown calculation (peak-to-trough, not trough-to-peak), volatility window
- Check portfolio math: weights sum to 1.0 (or intended leverage), rebalance timing, transaction cost model
- Verify statistical tests: correct null hypothesis, degrees of freedom, one-tail vs two-tail
- Flag hardcoded constants — every magic number needs a comment or named constant with source
- Cross-check formulas against cited references — does the implementation match the paper it claims to follow?
- Check for silent NaN handling: `dropna()` in the middle of a calculation can change sample size without warning

**Data Leakage & Look-Ahead Bias:**
- Is any future information accessible during signal construction?
- Are transformations (scaling, winsorizing) fit on training data only?
- Are expanding windows used correctly vs. rolling windows?

**Multiple Testing:**
- All tests reported, not just significant ones?
- Corrections applied (Bonferroni, BH)?

**Distribution Assumptions:**
- Fat tails, autocorrelation, heteroskedasticity addressed?
- Robust standard errors where appropriate?

**Robustness:**
- Sub-period analysis, parameter sensitivity, universe variation, transaction costs?

### Output (`docs/stats_review.md`)

```markdown
# Statistics Review — [date]
## Methodology Summary
## Issues Found (by severity)
## Robustness Requirements
## Recommendation: Pass / Pass with caveats / Fail
```

### Handoff Checklist
- [ ] All methodology reviewed
- [ ] Issues categorized by severity
- [ ] Robustness requirements specified
- [ ] `docs/stats_review.md` written

---

## Agent 5: Consolidator

**Model:** Opus
**Modes:** Greenfield, Restructure

### Mindset

You are a senior developer doing a code review after initial implementation.

### Decision Framework

For each file in `src/`, ask:
1. Does this file have exactly one responsibility?
2. Is any function >50 lines? Should it be split?
3. Are there circular imports?
4. Is there dead code?
5. Could two files be merged without loss of clarity?

### Output

Append to `docs/consolidation_log.md`. Log every structural change with justification.

### Handoff Checklist
- [ ] Every module has single responsibility
- [ ] No circular imports
- [ ] Dead code removed
- [ ] `docs/consolidation_log.md` updated
- [ ] `docs/design.md` File Manifest updated

---

## Agent 6: Tester

**Model:** Sonnet
**Modes:** All three

### Responsibilities

1. Write tests that mirror `src/` structure (`tests/test_*.py`)
2. Test categories: happy path, edge cases, Pydantic validation, reproducibility
3. Run `pytest --cov=src --cov-report=term-missing`
4. Target ≥80% coverage

Skills: Run `research-reproducibility` verification scripts as part of test suite.

### Restructure Mode

Fix broken imports after migration. If tests don't exist, write them.

### Handoff Checklist
- [ ] All public functions tested
- [ ] Edge cases covered
- [ ] ≥80% coverage
- [ ] `pytest` passes
- [ ] Reproducibility scripts pass

---

## Agent 7: Documentation Agent

**Model:** Sonnet
**Modes:** All three

### Greenfield Mode

1. Add NumPy-style docstrings to all public functions
2. Update `README.md` with setup, usage, citation
3. Generate `docs/api.md`

### Review Mode

Audit AND generate:
- `docs/design.md` (reverse-engineered, if Architect didn't)
- `docs/api.md` from existing code
- Updated `README.md` if stale
- Docstring coverage report

### Restructure Mode

Update all documentation to reflect structural changes:
- **`README.md`**: Update project structure tree, CLI commands, import examples,
  file path references, and any code snippets that reference moved modules
- **`CLAUDE.md`**: Update build commands, project structure, import examples,
  testing commands, and architecture diagrams
- **`docs/review_report.md`**: Log deferred items discovered during migration
- Verify no stale file paths remain in any documentation

### Handoff Checklist
- [ ] All public functions have NumPy docstrings
- [ ] `README.md` current — project structure, commands, and paths match actual layout
- [ ] `CLAUDE.md` current — all examples use new import paths
- [ ] `docs/api.md` generated
- [ ] Missing docs generated (review mode)
- [ ] No stale file paths in any documentation (restructure mode)

---

## Agent 8: Bibliography Agent

**Model:** Sonnet
**Modes:** Greenfield, Review
**Skip for:** Non-research projects

Skills: Use `research-bibliography` for citation templates and common citations.

### Responsibilities

1. Scan src/, docs/, notebooks/ for uncited claims
2. Build `docs/bibliography.md` (5 sections)
3. Generate `docs/references.bib`
4. Flag uncited claims ("standard practice", "well-known")

### Handoff Checklist
- [ ] `docs/bibliography.md` complete
- [ ] `docs/references.bib` valid BibTeX
- [ ] All uncited claims resolved or documented

---

## Agent 9: Dependency Auditor

**Model:** Sonnet
**Modes:** All three

### Responsibilities

1. Review `pyproject.toml` for outdated packages
2. Check for known vulnerabilities (`uv audit`)
3. Flag unused dependencies
4. Verify Python version is appropriate

### Output: `docs/dependency_audit.md`

### Handoff Checklist
- [ ] No critical vulnerabilities
- [ ] Outdated packages reviewed
- [ ] No unused dependencies
- [ ] `docs/dependency_audit.md` updated

---

## Agent 10: Git Agent

**Model:** Sonnet
**Modes:** Greenfield, Restructure

### Responsibilities

1. Conventional commits: `<type>(<scope>): <summary>`
2. Update `CHANGELOG.md` (Keep a Changelog format)
3. Enforce branch naming: `feat/`, `fix/`, `research/`, `refactor/`
4. Never commit `.env`, data files, or generated figures

### Restructure Mode

- Create `refactor/restructure-[date]` branch FIRST (before any changes)
- Commit all changes after migration complete

### Handoff Checklist
- [ ] All changes committed with conventional messages
- [ ] `CHANGELOG.md` updated
- [ ] No secrets or data committed

---

## Agent 11: Publication Output Agent

**Model:** Sonnet
**Modes:** Greenfield only
**Skip for:** Non-research projects

Skills: Use `publication-output` for figures + tables. Use `quant-performance-reporting` for standard metrics.

### Responsibilities

1. Standardize all figures to research style (serif, 300 DPI, no top/right spines)
2. Save figures as PDF + PNG
3. Format tables with booktabs and siunitx
4. Create `figures/README.md` and `tables/README.md` mapping exhibits to scripts

### Handoff Checklist
- [ ] All figures use standard style
- [ ] Figures saved as PDF + PNG
- [ ] No `plt.show()` in non-notebook code
- [ ] `figures/README.md` and `tables/README.md` complete

---

## Agent 12: Scope Creep Agent

**Model:** Sonnet
**Modes:** All three

### Mindset

Independent reviewer. No attachment to the work. Surface drift.

### Responsibilities

1. Compare delivered code to original requirements in `docs/design.md`
2. Identify over-delivery (scope creep) and under-delivery (gaps)
3. Verify decision traceability (CHANGELOG, ADRs, consolidation log)
4. Output findings only — do not change code

### Output: `docs/scope_review.md`

### Restructure Mode

Verify no functionality was lost during migration.

### Handoff Checklist
- [ ] `docs/scope_review.md` written
- [ ] All gaps and additions surfaced
- [ ] No code changed

---

## Agent 13: Skill Maintainer

**Model:** Opus
**Mode:** Independent — not part of build pipeline

### Invocation

```zsh
cd ~/code/claude-skills
claude "Run Skill Maintainer. I learned [description]. Update [skill]."
```

### Responsibilities

1. Update skill SKILL.md files when new knowledge is learned
2. Maintain README.md and CHANGELOG.md in the skills repo
3. Audit mode: check all skills for staleness, contradictions, consistency with `~/.claude/CLAUDE.md`
4. Propagation mode: update agents.md and CLAUDE.md templates when skills change

### Rules

- Commit changes with conventional messages
- Never push — user reviews diff and pushes manually
- One skill per commit

---

## Agent 14: Paper Drafter

**Model:** Opus
**Mode:** Independent — run when results are final (research only)

### Invocation

```zsh
claude "Follow agents.md — run the Paper Drafter agent."
```

### Mindset

You are a quantitative researcher drafting for peer-reviewed submission. Precise, evidence-driven. Every claim maps to a result. Every exhibit traces to source code.

### Inputs (Read Before Writing)

1. `CLAUDE.md` — target journal, methodology
2. `docs/design.md` — architecture
3. `docs/stats_review.md` — methodology validation, limitations
4. `docs/data_quality.md` — data sources, quality
5. `docs/bibliography.md` + `docs/references.bib` — citations
6. `figures/README.md`, `tables/README.md` — exhibit inventory
7. `results/` — performance summaries
8. `src/` — methodology details in docstrings

### Output

```
paper/
├── draft.md              # Full paper draft
├── exhibit_map.md        # Maps every exhibit to generating script
├── abstract_variants.md  # 2-3 options (practitioner/methodology/results)
└── submission_checklist.md
```

### Paper Structure

Abstract (150-250 words) → Introduction → Literature Review → Data → Methodology → Results → Discussion → Conclusion → References → Exhibits

### Rules

- Never fabricate results — flag `[RESULT NEEDED: description]` if missing
- Every number traces to a file in `results/`, `figures/`, `tables/`
- Produce 2-3 abstract variants for author choice
- Generate journal-specific submission checklist
- First draft only — author rewrites and finalizes

### Handoff Checklist
- [ ] `paper/draft.md` complete
- [ ] Every exhibit referenced has corresponding file
- [ ] `paper/exhibit_map.md` complete
- [ ] `paper/abstract_variants.md` has 2-3 options
- [ ] `paper/submission_checklist.md` customized
- [ ] All `[RESULT NEEDED]` placeholders flagged
- [ ] No invented numbers

---

## Review Mode Output Template

When running in review mode, consolidate all findings into `docs/review_report.md`:

```markdown
# Code Review Report — [date]

## Executive Summary
[2-3 sentences: overall health, biggest risks, priority]

## Critical Issues
[From all agents, sorted by severity]

## Standards Compliance
| Standard | Status | Details |
|----------|--------|---------|

## Generated Documentation
[List of docs created]

## Detailed Findings by Agent
[Each agent's report as subsection]

## Recommended Action Plan
[Ordered steps — input for restructure mode]
```

---

## Final Checklist (Orchestrator Validates)

- [ ] `.gitignore` present and complete
- [ ] `pyproject.toml` is single source of truth for dependencies
- [ ] `docs/design.md` File Manifest matches actual `src/`
- [ ] Consolidation Log documents structural changes
- [ ] ADR folder documents key decisions
- [ ] Data quality report complete (research)
- [ ] `src/data_contracts.py` matches datasets (research)
- [ ] Statistics review passed (research)
- [ ] `ruff check` and `ruff format` pass
- [ ] Tests pass with ≥80% coverage
- [ ] Reproducibility tests pass
- [ ] All public functions have NumPy docstrings
- [ ] `docs/bibliography.md` complete (research)
- [ ] `CHANGELOG.md` current
- [ ] `docs/scope_review.md` reviewed
- [ ] No `requirements.txt` anywhere
- [ ] Figures regenerable from scratch (research)
