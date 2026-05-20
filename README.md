# Boston Ride-Hailing Analysis · Q4 2018

**Uber as focal platform · Lyft as competitive benchmark**

A personal project built independently outside of coursework — applying SQL, Python, and Tableau to 693,000+ Uber and Lyft ride bookings across 12 Boston neighborhoods in November–December 2018. The analysis covers demand patterns, pricing dynamics, surge behavior, weather effects, and rider ratings, with a Tableau dashboard focused on Uber Q4 KPIs.

---

## TL;DR

- **693,071 rides** across Uber (55.6%) and Lyft (44.6%) in Boston Q4 2018
- Lyft's average fare ($17.36) exceeds Uber's ($15.78) — driven by product mix, not tier-for-tier pricing
- Only **6.9% of Lyft rides** carry any surge; Uber surge is not captured at the row level
- Price-distance correlation is moderate (~0.35), not strong — tier and surge explain as much as distance
- Weather has **effectively no impact on pricing** ($0.22 spread across all conditions)
- Driver ratings are identical across both platforms: mean **4.23**, <0.02 variation across all tiers

---

## Business Problem

Ride-hailing platforms operate in a competitive, surge-driven pricing environment where demand, geography, time of day, and weather all interact. This project treats **Uber as the focal platform** and **Lyft as a competitive benchmark** to answer five questions:

1. When do Bostonians ride, and does demand differ by platform or neighborhood?
2. How does Uber price across service tiers — and how does Lyft compare?
3. When and how aggressively does surge pricing activate?
4. Does weather shift demand or pricing behavior?
5. What does rider satisfaction look like across platforms and tiers?

---

## Dataset Overview

| Attribute | Detail |
|---|---|
| **Source** | [Kaggle — Uber and Lyft Dataset Boston, MA](https://www.kaggle.com/datasets/brllrb/uber-and-lyft-dataset-boston-ma) |
| **Raw records** | 693,071 rows |
| **Coverage** | November–December 2018 · 12 Boston neighborhoods |
| **Platforms** | Uber (55.6%, n=385,663) · Lyft (44.6%, n=307,408) |
| **Key fields** | Ride product, price, distance, surge multiplier, weather, driver/customer ratings |

**Structural data quality notes documented in analysis:**

| Column | Null count | Cause |
|---|---|---|
| `price` | 55,095 (7.9%) | Taxi product only — fare not collected |
| `Payment Method` | 591,071 (85.3%) | Dataset-level collection gap, both platforms |
| `Driver/Customer Ratings` | 600,071 (86.6%) | Sparse collection, both platforms |
| `surge_multiplier` (Uber) | — | Always 1.0 — not captured at row level |

---

## Repository Structure

```
uber-lyft-boston-analysis/
├── Data/
│   ├── Raw/                          Raw dataset (693K rows)
│   └── Clean/                        rides_clean.parquet (output of cleaning notebook)
├── Data Viz/
│   ├── Tableau files/                Uber Q4 KPIs.twb · Uber Q4 KPIs.twbx
│   └── Visualizations/               10 EDA charts (PNG)
├── Notebooks/
│   ├── Jupyter/
│   │   ├── Data_cleaning.ipynb       Step-by-step cleaning with documented decisions
│   │   └── EDA.ipynb                 5-section EDA, 10 charts, narrative findings
│   └── SQL - Big Query:DuckDB/
│       └── Structure_SQL.py          BigQuery-compatible SQL for schema profiling & segmentation
└── requirements.txt
```

---

## Methodology

### `Structure_SQL.py` — Data Structuring & Granularity
BigQuery-compatible SQL executed locally via DuckDB. Covers:
- Schema profiling and type casting
- Granularity confirmation: one row = one unique ride booking
- Platform, ride tier, neighborhood, and temporal segmentation
- Structural null audit — diagnosing whether missingness is random or tied to specific products/platforms
- Persistent views registered for downstream use: `rides_tiered`, `uber_rides`, `lyft_rides`, `rides_priceable`

> SQL written in BigQuery-compatible dialect. Queries run unchanged on BigQuery, Redshift, or Snowflake.

### `Data_cleaning.ipynb` — Data Cleaning
Cleaned from raw — the companion "pre-cleaned" file (audited on a sample) only made cosmetic changes and left all structural nulls intact. This notebook:
- Renames all columns to `snake_case`
- Parses datetime and derives `date`, `day_of_week`, `week_number`, `is_weekend`
- Corrects dtypes via `pd.to_numeric(errors='coerce')` — safe for dirty data
- Strips whitespace from all categorical columns
- Assigns consistent ride tier labels across both platforms
- Adds availability flags (`price_available`, `payment_data_available`, `has_ratings`) for transparent downstream filtering
- Flags statistical price outliers without removing them — all 5,114 outliers are Lyft premium products (Lux Black, Lux Black XL), consistent with Lyft's higher premium ceiling
- Exports `rides_clean.parquet`

### `EDA.ipynb` — Exploratory Data Analysis
Five analytical sections across 10 charts:

| Section | Focus | Charts |
|---|---|---|
| A · Demand | Hourly volume by platform, day-of-week heatmap, top corridors | 3 |
| B · Pricing | Price distributions, tier comparison, price vs. distance | 3 |
| C · Surge | Lyft surge frequency distribution, surge rate by hour | 2 |
| D · Weather | Volume by condition, price by condition, temperature correlation | 3 |
| E · Ratings | Driver rating distributions, tier-level comparison | 2 |

### Tableau Dashboard — Uber Q4 KPIs
Interactive dashboard visualizing Uber-specific Q4 metrics: ride volume, pricing by product, top corridors, and temporal demand patterns. Built in Tableau Desktop — open `Uber Q4 KPIs.twbx` directly in Tableau Desktop or Tableau Public.

---

## Key Findings

**Demand** — Ride volume is relatively flat across the 24-hour cycle, with mild peaks around midnight and 23:00. Both platforms follow nearly identical hourly curves — Uber runs ~25% higher volume at every hour but shows no platform-specific use case (e.g. Uber for airports, Lyft for bars). Top corridors are compact urban routes: Financial District ↔ South Station, West End ↔ Fenway.

**Pricing** — Lyft's overall average ($17.36) exceeds Uber's ($15.78), but this reflects product mix: Uber's Taxi product has no price data and its higher Economy volume pulls the average down. Tier-for-tier, pricing is competitive. Price-distance correlation is moderate for both platforms (Lyft r=0.361, Uber r=0.337) — tier and surge explain as much of the fare as raw distance.

**Surge** — Only 6.9% of Lyft rides carry any surge above baseline. Surge rate is broadly flat across all hours of the day (~6–8%), with a mild peak at 13:00. The expected late-night bar-closing spike is not present — suggesting supply constraints are distributed across all hours rather than concentrated at night. Uber surge is not captured at the row level in this dataset.

**Weather** — Overcast is the dominant condition by volume (156K rides), reflecting Boston's Q4 climate — not a demand preference. Price differences across all nine weather conditions span only $0.22 (Mostly Cloudy $16.60 → Drizzle $16.38). Temperature shows r = -0.001 with price. Weather has no meaningful impact on pricing.

**Ratings** — Both platforms produce an identical mean driver rating of 4.23, with less than 0.02 points of variation across all six ride tiers. Ratings are available for only 13.4% of rides — conclusions are descriptive of that subset only.

---

## Tools & Stack

| Tool | Purpose |
|---|---|
| DuckDB | BigQuery-compatible SQL — schema profiling, segmentation, structural null audit |
| pandas | Data cleaning, type handling, feature engineering |
| matplotlib · seaborn | Visualization |
| pyarrow | Parquet I/O |
| Tableau Desktop | Interactive Q4 KPI dashboard |
| Jupyter | Notebook execution environment |

---

## How to Run

```bash
# Clone the repo
git clone https://github.com/DanielSantiagoGuzman/uber-lyft-boston-analysis.git
cd uber-lyft-boston-analysis

# Install dependencies
pip install -r requirements.txt

# Run in order:
# 1. SQL structuring (DuckDB — no server needed)
python Notebooks/SQL\ -\ Big\ Query\:DuckDB/Structure_SQL.py

# 2. Data cleaning
jupyter notebook Notebooks/Jupyter/Data_cleaning.ipynb

# 3. EDA
jupyter notebook Notebooks/Jupyter/EDA.ipynb
```

The Tableau workbook (`Data Viz/Tableau files/Uber Q4 KPIs.twbx`) opens directly in Tableau Desktop or Tableau Public.

---

## About This Project

Built independently in personal time — not a class assignment. Topics applied here draw on coursework in data engineering, experimentation, and analytics, but the dataset selection, problem framing, analysis design, and tooling choices were made entirely outside of any academic context.
