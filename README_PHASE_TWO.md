# ====================KAINGA_ORA_HOUSING_BUILD_NOTES====================

# Staging Layer, Mart, Synthetic Data and Streamlit Application

**Status:** Complete. All 19 reconciliation checks pass.
**Built:** 23–24 August 2026
**Staging + mart + synthetic:** `db/housing.duckdb` (47.8 MB)
**Published extract:** `public/kainga_ora_housing_public.duckdb` (18.3 MB)
**App:** `app/kainga_ora_housing_nz.py` — 1,648 lines, live at
<https://celnic-housing-nz.streamlit.app>

```bash
python -m streamlit run public_repo/app/kainga_ora_housing_nz.py --server.port 8520
```

---

# ====================THE_PLAN_FLIPPED_DRIVING_FROM_BUSINESS_OUTCOME====================

The app spec came first and the transformation layer was designed to serve it, not
the other way round. Each tab was reduced to the single question it has to answer,
and each question to the grain that answers it.

| Tab | Question | Grain required | Mart table built |
|---|---|---|---|
| 📊 National Overview | How much public housing is there, who is waiting, and what does the market charge? | quarter × national | `M_NATIONAL_QUARTER`, `M_KO_STOCK_PORTFOLIO` |
| 🗺️ Stock Map | Where is the stock? | property × H3 cell, coloured by four metrics | `SYN_KO_PROPERTY` × `M_KO_STOCK_TA_BEDROOM` |
| 💰 Market Rent vs IRR | How far below market does a state tenant pay? | quarter × TA × bedrooms | `M_MARKET_RENT_TA_BEDROOM`, `SYN_KO_TENANCY` |
| 📋 Housing Register | Who is waiting, and does the stock match what they need? | quarter × TA × priority × bedrooms required | `M_REGISTER_*` |
| 🔧 Asset & Maintenance | What condition is the stock in and what does it cost? | work order × property | `SYN_MAINTENANCE` |
| ⚙️ Pipeline | Can I trust any of this? | file, check, lineage | `META_*`, `VALIDATION_RESULTS` |

**What that analysis forced into the design**, working back from the questions:

1. Every tab needs a **period, a place and a measure**, so the facts share one
   conformed shape — `PERIOD` as `YYYYQn`, `TA_NAME` canonicalised, one measure
   column — rather than mirroring each agency's worksheet.
2. Tab 3 is the centrepiece and needs **market rent by TA *and* bedrooms**. That
   cut does not exist in the public files (see the mart section), so the design
   forced a derived table and an explicit `IS_DERIVED` flag rather than quietly
   presenting an estimate as a measurement.
3. Tab 2 needs **coordinates Kāinga Ora never publishes** — hence a bundled TA
   and local-board geography and pre-computed H3 cells at resolutions 8–12.
4. Tab 4's payoff is the **bedroom mismatch**, which requires demand and supply on
   the same bedroom axis from two different agencies — hence one shared bedroom
   vocabulary (`Bedsit, 1, 2, 3, 4, 5+`) across MSD and Kāinga Ora staging.
5. Nothing below region-by-bedroom is published, so four tabs need **synthetic
   data that reconciles to published aggregates** rather than plausible noise.
6. Every figure has to declare itself, so **real / derived / synthetic** is a
   property of the table, recorded in `META.LINEAGE` and surfaced in tab 6.

---

# ====================STAGING_LAYER====================

## The problem

The RAW layer holds 115 tables from 404 files across three genuinely different
shapes: MBIE's tidy CSVs, MSD's and Kāinga Ora's presentation workbooks as
faithful cell grids, and a decade of Kāinga Ora PDFs. Writing a parser per file
would mean hundreds of parsers that break at every release.

## The solution: four resolvers

Inspecting the corpus showed four layouts. Each gets one resolver in
`scripts/05_stage.py`.

| Resolver | Shape | Applies to |
|---|---|---|
| **A. Periods-as-columns** | Header row carries `Jun-21`, `Sep-21`…; each data row is one dimension value | MSD register workbooks |
| **B. Bedroom matrix** | Merged "Number of bedrooms" banner over `Bedsit \| 1 \| 2 \| 3 \| 4 \| 5+ \| Total` | Kāinga Ora XLSX, Sept 2025 onward |
| **C. PDF geometry grid** | Numbers snapped to header column x-centres | Kāinga Ora archive PDFs, 2015Q3–2025Q2 |
| **D. Tidy CSV** | Already long | MBIE rental bonds |

**Resolver behaviours that matter:**

- **Header rows are located, not assumed.** MSD moves the header, renames the
  first column (`TA` / `Territorial Authority` / blank), and publishes a rolling
  five-year window so the column set differs between releases.
- **Deduplication happens after canonicalisation, never before.** MSD renamed
  Auckland mid-series ("Auckland Super City" → "Auckland City"); deduplicating on
  the raw label kept both and counting them inflated 2015Q2 by a third.
- **Suppression is preserved.** MSD's `S` becomes `NULL` with a `SUPPRESSED`
  flag. Never zero.
- **Dashes are zeros, but not all dashes are the same character.** Kāinga Ora
  prints "none" as `U+2500 BOX DRAWINGS LIGHT HORIZONTAL` before 2020 and an en
  dash or hyphen after.
- **Rows are reconciled against their own printed total** and, where exactly one
  cell is missing, repaired from the residual and flagged `REPAIRED`. Rows that
  cannot be reconstructed are flagged `UNRECONCILED` rather than guessed at.

Output: **24 staging tables, 1,291,741 rows**, all of it real published data.
Nothing synthetic enters the platform until step 08.

---

# ====================MART====================

Schema `MART` in `db/housing.duckdb`. **16 tables, 129,056 rows.**

| Table | Rows | Coverage |
|---|---|---|
| `M_MARKET_RENT_TA_BEDROOM` | 44,395 | derived, 2020Q1 – 2026Q3 |
| `M_EH_GRANTS_TA` | 29,714 | 2022Q1 – 2026Q2 |
| `M_REGISTER_TA_BEDROOMS` | 14,741 | 2017Q1 – 2026Q2 |
| `M_KO_STOCK_TA_BEDROOM` | 14,247 | 2015Q3 – 2025Q3 |
| `M_BOND_TA_QUARTER` | 8,800 | 1993Q1 – 2026Q2 |
| `M_KO_STOCK_LOCAL_BOARD` | 4,670 | Auckland's 21 boards |
| `M_BOND_NATIONAL_BEDROOM` | 4,174 | by dwelling type and bedrooms |
| `M_REGISTER_TA_QUARTER` | 3,283 | 2014Q2 – 2026Q2 |
| …and 8 more | | priority, demographics, portfolio, HUD headlines |

## The one derived table, and why it is derived

`M_MARKET_RENT_TA_BEDROOM` is the only modelled table in the mart, and it exists
because of a gap in what MBIE publishes:

- Rent **by territorial authority** is published **without a bedroom split**.
- Rent **by bedroom count** is published against roughly **2,000 six-digit area
  codes for which no code-to-name lookup exists**. MBIE's own market-rent service
  returns suburb names without the matching identifier, and the data.govt.nz
  catalogue entry has no lookup resource.

So a real market rent by TA *and* bedrooms cannot be assembled from the public
files. The mart scales each district's measured rent by the national bedroom
relativity for the same quarter and sets `IS_DERIVED = TRUE`. Tab 3 says so in a
caveat block that opens by default, and every axis label reads "derived".

## Synthetic layer

Schema `SYN`. **4 tables, 364,221 rows.** Seed 42, so the build reproduces.

| Table | Rows | Seeded from |
|---|---|---|
| `SYN_MAINTENANCE` | 200,000 | nothing — no public equivalent at any grain |
| `SYN_KO_PROPERTY` | 72,951 | exact TA × bedroom counts, 2025Q3 release |
| `SYN_KO_TENANCY` | 72,295 | tenanted properties; **market rent joined from real bond data** |
| `SYN_HOUSING_REGISTER` | 18,975 | exact TA totals, priority and bedroom splits, MSD 2026Q2 |

Three properties of the synthetic layer are load-bearing:

1. **Counts match the published table exactly**, district by district and bedroom
   by bedroom — not merely in national total.
2. **Auckland is placed from Kāinga Ora's own local-board table** across 21
   boards. It holds ~45% of the portfolio; scattering it around one centroid
   would put half the country's state housing in a single hexagon.
3. **The subsidy gap is half real.** Market rent is measured from bond data;
   only the income-related rent it is subtracted from is modelled.

---

# ====================VALIDATION====================

`scripts/06_validate.py` — **19 of 19 checks pass.**

The checks that matter are the **cross-source** ones, because a parser can agree
with itself while being wrong.

| Check | Result |
|---|---|
| Kāinga Ora stock vs **HUD's** published Kāinga Ora count | Pass — 3 quarters, 2 exact, worst 21 homes (0.031%) |
| TA rollup vs the page-1 portfolio total in the same PDF | Pass — 41 quarters, worst 0.34% |
| Mart national series vs published portfolio total | Pass — 41 quarters, worst 0.34% |
| Every TA row's cells sum to its own printed total | Pass — 2,563 rows, 2 disagree by ≤3 |
| Register TA sum never exceeds the national total | Pass — 49 quarters, no double counting |
| Register priority split vs national summary | Pass — worst 0.08% |
| Register TA shortfall within suppression | Pass — worst 3.04% |
| Synthetic properties vs published stock | Pass — exact in all 281 TA × bedroom cells |
| Synthetic register vs published register | Pass — exact in every TA |
| Bond header repair applied | Pass — 4,317 rows carry a log std dev |
| National median rent rises 1993 → 2026 | Pass — $160 → $600 per week |
| H3 present at every resolution 8–12 | Pass — 12,139 res-8 cells, 72,144 res-12 |

Tolerances are tied to a documented cause — MSD's base-3 random rounding, its
suppression of small counts, transcription slips in the published PDFs, and
definitional differences between agencies — not chosen to make checks go green.
Every residual is printed in the check output and shown in tab 6.

---

# ====================FOUR_WAYS_A_PARSER_LIES====================

Each produced numbers that looked entirely plausible. All four were caught by
reconciliation, not by inspection.

**The dash that is not a hyphen.** Matching only `-` dropped every table row
containing a zero — four TA rows in five in the 2015–2019 releases. It dropped
them *consistently*, so the survivors still summed to their own totals perfectly
and a parts-versus-total check found nothing. What exposed it was the TA count per
quarter: 12 districts in a country with 67.

**The blank cell that shifts its neighbours.** `Ashburton District 22 68 85 17 192`
is five numbers for six bedroom columns, and `(1,2,3,4)` and `(2,3,4,5+)` both sum
to the published total. Fixed by snapping numbers to header x-centres.

**The label that wraps.** Long district names wrap with the numbers printed on the
line *between* the two halves, so requiring a label on the same line dropped
exactly the longest-named districts — Queenstown-Lakes, Palmerston North, Western
Bay of Plenty and a dozen more.

**The rename that doubles a city.** Deduplicating on the raw TA label before
canonicalising kept both "Auckland Super City" and "Auckland City", and counting
them inflated the 2015Q2 national register by a third.

---

# ====================APPLICATION====================

Built to the `snowflake-streamlit-development` conventions, with the
`snowflake-streamlit-development-excel-export` pattern for downloads.

**What follows the skill exactly:** the section separators; every query a
`@st.cache_data` method taking `df_db_schema`; every visual a `render_*` method;
a thin `main()`; UPPERCASE dataframe keys; `GROUP BY ALL`; pydeck `H3HexagonLayer`
with a CartoDB basemap (never a `mapbox://` URL); `build_styled_excel` in
`STATIC_METHODS` with the `st.columns([3, 1])` header row placing a right-justified
`📥 Excel` button beside every detail dataframe title.

**What is deliberately different:** the session block opens a local DuckDB
instead of calling `get_active_session()`. All access goes through one
`run_query()` method, so porting to Snowflake means changing that single function.

## The file is a visual specification

The module docstring carries a screen layout map, the tab order with each tab's
real/synthetic status, and the visual vocabulary — what 🔶 means, when a banner is
amber rather than red, which colour is measured supply and which is pressure.
Every `render_*` docstring is a top-to-bottom layout of the block it draws, and
records *why* where the choice carries meaning. Every cached query names the
element it feeds. All 44 top-level functions carry a docstring.

## Honesty built into the interface

- A **hazard-striped provenance banner** sits above the tabs, not in a footer, so
  it is read before any figure rather than found underneath one.
- **Synthetic banners are graded**: amber where some series on the tab are
  published data (tabs 2 and 4), red where nothing is measured (tab 5).
- The single synthetic metric on tab 1 carries 🔶 and its own caption rather than
  bannering the whole tab, which would wrongly imply the other four are modelled.
- The **CHP line is dotted and stops mid-chart**, so the end of HUD's publication
  reads as a publication decision rather than as community housing disappearing.
- The **register delta is inverted** — a rising waitlist is bad news, and the
  default green-for-up would read as an improvement.
- Tab 6 computes its pass/fail state from the stored results, so it cannot claim
  the checks pass while they are failing.

---

# ====================DECISIONS_AND_DEPARTURES====================

**The Market Rent API was not used.** It requires registering for an API key.
The CSV files behind it are the same bond database, so nothing is lost.

**Stats NZ boundaries were not used.** The Geographic Data Service requires a
registered key for every download and WFS call, which would stop anyone from
rebuilding this pipeline after cloning it. Territorial authority and local-board
centroids are bundled in `scripts/nz_geography.py` instead — 67 TAs and 21 boards.
They are centroids, not boundaries, and every property location is synthetic
anyway.

**Suburb-level bond rows are staged but not surfaced.** 1,162,390 rows sit in
`STG_BOND_AREA_DETAIL` against area codes that cannot be named. They are retained
for anyone who later obtains the lookup and excluded from the published extract.

**HUD's quarterly report ceased after December 2023.** Its successor is an
embedded Power BI dashboard with no export, so CHP homes, transitional housing
and Housing First stop at 2023Q4. Five MSD housing datasets the work packet does
not mention were found while resolving a dead URL and partly fill 2024–2026.

**The regional factsheets are not mined.** 66 HUD regional PDFs are landed in RAW
and left there: they change prose template almost every quarter, and turning them
into numbers risks figures that are quietly wrong.

---

# ====================RUNNING_IT====================

```bash
python scripts/run_all.py            # everything
python scripts/run_all.py --from 05  # resume
python scripts/run_all.py --only 06  # one step
```

| Script | Purpose |
|---|---|
| `01`–`04` | Discover, download, extract, land the RAW layer |
| `05_stage.py` | Resolve RAW into 24 tidy staging tables |
| `06_validate.py` | 19 reconciliation checks; non-zero exit on failure |
| `07_mart.py` | 16 conformed mart tables, including the derived rent table |
| `08_synthetic.py` | 4 synthetic tables, seed 42, reconciled to published totals |
| `09_build_public.py` | The 18 MB extract the published app reads |
| `nz_sources.py` | Source registry — the only place a URL is written |
| `nz_families.py` | Publication renames, every merge and non-merge recorded |
| `nz_geography.py` | TA and local-board reference geography |

One step needs a human first: Kāinga Ora sits behind Imperva bot protection, so
`scripts/.ko_cookie` needs a current browser-harvested session cookie. If it is
stale, discovery falls back to cached copies of the pages under `raw/_pages/`
rather than silently losing a dataset.

---

# ====================ADDENDUM_WHAT_DEPLOYMENT_TAUGHT====================

Added 24 August 2026, after the app went live. Two defects survived a green
19-check validation run, and both were found by looking at the deployed app
rather than at the pipeline.

## A filter that was right for one chart and wrong for another

`M_KO_STOCK_NATIONAL_QUARTER` summed bedroom cells from the table that drops rows
flagged `UNRECONCILED`. That filter is correct for bedroom-level charts and wrong
for a national total: Auckland is flagged in 2017Q1, so the headline series read
**35,019 against a published 62,553** and the supply chart showed a 44% cliff that
never happened.

The existing cross-source check compared *staging* against the published portfolio
total and passed, because the defect was introduced **downstream of staging**.
Validation that stops at the layer before the one the app reads will miss this
whole class of bug.

Fixed by using each district's published row total — reliable even when the cells
beside it are not — and keeping every district. Two mart-level checks now repeat
the comparison against what the app actually reads.

## A cache that outlived its data

Streamlit Community Cloud hot-reloads on a push: it pulls the new files and
re-runs the script, but does **not** clear `cache_resource`. The DuckDB connection
opened before the pull kept reading the replaced file, so a query written against
a newly added column raised a `BinderException` against data sitting correctly on
disk a few bytes away. It did not reproduce locally, because locally the process
restarts.

`get_connection` now takes a fingerprint of the extract — path, size, mtime — so a
data refresh becomes a cache miss and the connection reopens by itself. Query
results are keyed on the same fingerprint, or a reopened connection would still
serve frames cached from the previous file.

This had already been written up in `DEPLOY.md` as "reboot after a data refresh".
A manual step that must be remembered every time is a step that will eventually be
forgotten, so it is now handled in code.
