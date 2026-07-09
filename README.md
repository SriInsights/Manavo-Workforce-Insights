# Manavo-Workforce-Insights
Manavo Workforce Insights is a HR Analytics Portfolio for 30K Employees Synthetic HRIS
# HR Analytics Portfolio — 30K Employee Synthetic HRIS

A from-scratch synthetic HRIS dataset simulating a 30,000-employee company, 
built to mirror how enterprise systems like Workday actually structure 
workforce data — including effective-dated position history (not just 
"current state" snapshots).

## Why this exists

Most portfolio HR datasets are static CSVs with one row per employee. Real 
HRIS systems track *history* — every promotion, comp change, and reorg as 
its own dated record — and reading data out of a relational source system, 
not just a CSV. This project builds that properly, including the 
performance problems that show up at real scale and how I fixed them.

## What's built so far

- 30,000-employee core dataset: department, job level (realistic pyramid 
  distribution), hire date (weighted toward recent years to simulate 
  company growth), tenure
- Manager hierarchy assigned via grouped lookup (see `BUILD_LOG.md` for 
  the O(n²) → grouped-lookup optimization story, including a real bug I 
  found and fixed involving ID stability)

## What's next

- Effective-dated position history (promotions/comp changes as dated rows)
- Bronze/silver/gold medallion pipeline at 30K scale
- Load into a real source database, then a Fabric Lakehouse

## Build log

See `BUILD_LOG.md` for a running, honest account of design decisions, 
bugs found, and how they were fixed — written as I built this, not after.
