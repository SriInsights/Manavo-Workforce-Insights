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

Phase 2 — Effective-dated history, 50,000 employees (30K active + 20K alumni)

Rebuilt the employee model the way a real enterprise HRIS does it —
position history as dated records, not single current-state rows — and
learned to handle the performance and integrity problems that only show up
at real scale.

Core build:


30,000 active employees, hire dates weighted for a realistic company
growth curve
Manager hierarchy via grouped lookup (department + job_level) instead of
a full pairwise scan — avoided ~56x slowdown vs. a naive O(n²) approach
Effective-dated position history (86,643 rows): job level, salary,
and location per position version, correctly chained start/end dates
Job level derived from starting_level + promotion count (not
independently randomized), with promotion velocity varying by career
stage (junior levels ~every 3 years, senior levels ~every 9 years,
reflecting real plateau patterns)
Salary assigned per position-history row within level-specific bands,
with real within-level variation
20,000 alumni employees across four realistic tenure categories
(never-started, very-short, short-medium, long-tenure), each with
termination dates constrained to fall before the snapshot date
Three-tier termination structure: employment_status (Active/Completed)
→ disposition (Voluntary/Involuntary/Never Started) → specific
termination_reason
Multi-locale realistic names (50,000 people, ~1.2% name-collision rate)
Department-appropriate job titles (11 departments × 7 levels, following
real industry progression: numbered junior titles → Senior/Staff/
Principal → Director/VP)
Work location, effective-dated alongside position changes


Real bugs found and fixed (full detail in BUILD_LOG.md):


Manager ID corruption from reordering employee_id after manager_id was
already built — caught only by a strict identity check, not a naive
existence check
job_level safety cap not propagating into a separately-calculated
function — 49 employees leaked "Level 8" until caught
Hire-date-first alumni generation could produce termination dates in
the future — fixed by deriving hire_date backward from a constrained
termination_date instead
Missing CEO record — position history existed for the CEO, but no
matching employee record did, a real referential-integrity gap caught
by testing "does E00000 actually exist" rather than assuming it did
Duplicate CEO row from re-running a non-idempotent pd.concat cell
Unseeded randomness — no random.seed() set, meaning re-running cells
produces valid but non-identical data; documented as a known
reproducibility limitation rather than silently ignored


What's next (Phase 3)


Performance reviews and engagement surveys, rebuilt at 50K scale
Compensation history independent of promotions (merit/market adjustments)
FTE %, contract type (permanent/contractor/part-time)
Bronze/silver/gold pipeline at 50K scale → real source database →
Fabric OneLake
AI-agent query layer (Anthropic API + tool calling) on the semantic model
