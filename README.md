# Lab 3 — NYC Taxi Data Profiling & Cleaning

**Course:** INF4058 Data Engineering
**Dataset:** `yellow_tripdata_2021-01.csv.gz` (1,369,765 rows)
**Scope:** Look → Profile → Find problems → Decide → Clean → **Stop** (no load into PostgreSQL)

---

## What this repo contains

| File | Purpose |
|------|---------|
| `lab3_cleaning.ipynb` | Profiling, problem detection, and the `clean(df)` function |
| `requirements.txt` | Python dependencies |
| `.gitignore` | Excludes data files, `.venv/`, and secrets |

> No data files, no `.venv/`, and no keys are tracked in this repository.

---

## Steps followed

1. **Look** — Read the first 100 rows; inspected `df.head()` and `df.dtypes` to identify date columns and mostly-empty columns.
2. **Profile** — Read the full file in chunks of **100,000 rows** (never loaded at once, data not modified). Built a per-column table of dtype, null count, null %, min, and max.
3. **Find problems** — Counted:
   - pickups outside January 2021
   - zero or negative `fare_amount`
   - zero or negative `trip_distance`
   - `passenger_count = 0`
   - fully duplicated rows
4. **Decide** — Chose **drop**, **fix**, or **flag** for each problem (see table below).
5. **Clean** — Implemented `clean(df)`, which:
   - works on a **copy** (original dataframe is never changed)
   - returns `(cleaned_df, {rule: rows_removed})`
   - never drops rows silently — every removal is counted per rule
6. **Stop** — The pipeline ends at cleaning. Loading into PostgreSQL happens next session.

---

## Cleaning rules

| Problem | Found | Decision | Rows removed | Rows flagged (kept) |
|---------|------:|----------|-------------:|--------------------:|
| Pickup outside Jan 2021 | 24 | Drop | 24 | – |
| Dropoff before/equal to pickup | 6,557 | Drop | 6,557 | – |
| `fare_amount <= 0` | 7,411 | Drop | 7,343 | – |
| Duplicate rows | 0 | Drop | 0 | – |
| `trip_distance <= 0` | 19,952 | Flag | 0 | 18,308 |
| `trip_distance > 100` miles | 55 | Flag | 0 | 55 |
| `passenger_count = 0` | 26,726 | Flag | 0 | 26,654 |
| Null vendor / passenger / payment columns | 98,352 | Fix (Int64) + Flag | 0 | 92,459 |

**Rows before:** 1,369,765  **Rows after:** 1,355,841  **Removed:** 13,924

Rules are applied in order, so a row removed by an earlier rule is not counted again by a later one
(that is why the "Found" and "Removed/Flagged" columns differ).

---

## How to run

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Download the data (not included in the repo)
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz

jupyter notebook lab3_cleaning.ipynb
```
