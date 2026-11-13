# Project Notes – NYC Collision Risk

## Question
**When and where are serious collisions most likely in NYC, and what drives severity?**

## Data
- Source: NYC Open Data, "Motor Vehicle Collisions - Crashes"
  https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95
- Filtered to crash dates 01/01/2023 – 12/31/2025
- Why: full years allow fair year-to-year comparison; skips COVID-era 2020–2021
- Downloaded: 2026-11-05

---

## Log

### 2026-11-05
- Did: set up repo, ignored data/ folder, downloaded filtered dataset
- Next: set up Python/Jupyter and load the data

### 2026-11-07
- Did: set up Python environment (.venv) + Jupyter kernel, added requirements.txt
- Did: loaded data in notebooks/01_load_and_clean.ipynb: 273,483 crashes × 29 columns, 2023-01-01 to 2025-12-31
- Found (missing values):
  - BOROUGH missing ~23%, ZIP ~27%, but LATITUDE/LONGITUDE missing only ~5.5%
      could recover borough from coordinates for the "where" question
  - CONTRIBUTING FACTOR / VEHICLE TYPE 3–5 are >90% empty, which is expected
      since most crashes involve 1–2 vehicles (not a data problem)
  - Injury/death counts have no missing values (good for severity analysis)
  - CRASH TIME "00:00" has ~5,000 crashes (~2× any other round hour).
      Likely a placeholder for unknown time → hour-of-day analysis should flag/exclude it.

  - Decision: two severity tiers
  - fatal_crash (≥1 killed): 737 crashes (0.27%), headline outcome, too rare for fine breakdowns
  - injury_crash (≥1 injured or killed): 118,128 (43%), used for hour/borough/factor breakdowns
  
- Next: Recover Borough from latitude/longitude, then do frequency analysis

### 2026-11-11
- Did: recovered missing BOROUGH from lat/long with k-NN (k=3, sklearn)
  - Removed 2,752 rows with (0,0) coords before training
  - Holdout accuracy 99.93% (80/20 split, 204,502 known rows)
  - Filled 51,130 rows → missing borough 23.4% → 4.7% (rest have no coords)
  - New column borough_filled; original BOROUGH kept unchanged
- Did: notebook 02, frequency + severity analysis
  - Crashes fell ~11%, 2023 → 2025 (−5.5%, −6.3%)
  - Peak 5 PM (~17,000), low 3 AM (~4,800), excluding 00:00 placeholders
  - Per-hour fatal counts too small (17–50) to compare single hours → grouped night (0–5) vs day
  - Night fatal rate 0.57% vs day 0.23%, ~2.5× (two-proportion z = 11.5, p < 0.001)
  - Night crashes: lower injury rate but higher fatal rate → fewer but deadlier
- Next: Poisson check on daily fatal crash counts.

### 2026-11-13
- Did: notebook 03, Poisson check on daily fatal crashes
  - 1,096 days; mean 0.672, variance 0.684 (close, consistent with Poisson)
  - Observed vs predicted: more 0-death days (575 vs 559.5) and 2-death days (152 vs 126.5),
    fewer 1-death days (339 vs 376.2)
  - Chi-square GOF (groups 0/1/2/3+, ddof=1): χ² = 9.69, p = 0.008 → reject Poisson
  - Meaning: risk likely not constant day to day; plain Poisson underestimates bad days
- Did: conditional fatal rates by who was hurt
  - P(fatal | pedestrian hurt) 1.28%, cyclist 0.43%, motorist 0.45%
  - Among injury crashes: pedestrian hurt vs not, 1.28% vs 0.43% (~3×), z = 15.6, p < 0.001
  - Used ped_hurt vs not, since pedestrian vs motorist groups overlap (543 crashes)
  - Data only records pedestrians if hurt, so this is "pedestrian hurt", not "involved"
- Next: write README (Question → Data → Analysis → Findings → Recommendation), add charts