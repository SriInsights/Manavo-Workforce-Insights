# Manavo-Workforce-Insights
Manavo Workforce Insights is a HR Analytics Portfolio for 30K Employees Synthetic HRIS
# HR Analytics Portfolio — 30K Employee Synthetic HRIS


A synthetic HRIS + analytics project built to mirror how enterprise systems
like Workday actually structure workforce data, and to learn the real data
engineering practices behind it — not just visualize a clean CSV.

This project is documented as a progression, phase by phase, because the
*mistakes and fixes along the way are as much the point as the final
output*. See `BUILD_LOG.md` for the full, honest decision log.

---

## Phase 1 — Flat dataset, 4,000 employees (started 07/07/2026)

**Goal:** learn the medallion architecture (bronze → silver → gold) and
build a believable synthetic HRIS dataset with real correlations, not
random noise.

**What I built:**
- 4,000-employee dataset with engineered attrition signal — lowest-pay
  quartile employees leave voluntarily at 35.7% vs. 17.5% for the highest
  quartile
- Bronze (raw, intentionally messy) → Silver (cleaned, every fix logged) →
  Gold (Kimball star schema, including a periodic snapshot fact table for
  headcount trending)
- A real relational source database (SQLite, enforced primary/foreign
  keys) upstream of bronze, with a scripted extract job — simulating an
  actual HRIS backend rather than starting from a CSV

**Limitation that motivated Phase 2:** every table was "current state
only" — one row per employee. There was no way to answer "what was this
person's job level or salary a year ago." That's not how real HRIS systems
(Workday included) work.

📁 See `/phase1_4k_employees/`

---

## Phase 2 — Effective-dated history, 30,000 employees *(started 07/08/2026)*

**Goal:** rebuild the employee model the way a real enterprise HRIS does
it — with position history as dated records, not a single current-state
row — and learn to handle the performance and data-integrity problems
that only show up at real scale.

**What I built:**
- 30,000-employee core table, hire dates weighted to produce a realistic
  company growth curve (260 hires in 2016 → 5,141 in 2025)
- Manager hierarchy assigned via grouped lookup by (department, job_level)
  instead of a full pairwise scan — avoided an ~O(n²) approach that would
  have been ~56x slower at this scale, not just 7.5x
- **Effective-dated position history** (50,489 rows): one row per version
  of an employee's job level and salary, with `effective_start_date` /
  `effective_end_date`, correctly chained with no gaps or overlaps
- Promotion velocity that varies by career stage (junior levels promote
  ~every 3 years, senior levels plateau at ~every 9 years) instead of a
  flat rule
- Salary bands per level, with real within-level variation instead of flat
  midpoints

**Real bugs found and fixed (full detail in BUILD_LOG.md):**
1. Sorted employees by hire date and reassigned employee_id *after*
   manager relationships were already built — silently corrupted every
   manager_id. A naive existence check didn't catch it; a stricter
   identity check did.
2. A safety cap on job_level (max 7) applied to the main employee table
   didn't propagate into a separate position-history generation function,
   letting invalid "Level 8" rows leak through for 49 employees.

📁 See `/phase2_30k_employees/`

---

## What's next

- Phase 3: bronze/silver/gold pipeline rebuilt at 30K scale, loaded into a
  real source database, then Fabric OneLake
- Phase 4: AI-agent query layer (Anthropic API + tool calling) on top of
  the semantic model

## Build log

`BUILD_LOG.md` — a running, honest account of every design decision and
bug, written as the project was built, not reconstructed afterward.
