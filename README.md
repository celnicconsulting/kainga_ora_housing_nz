# Kāinga Ora / New Zealand Public Housing Intelligence

An interactive view of New Zealand's public housing system, built entirely from
open government data: who is waiting for a house, what stock exists and where,
and how far the rent a state tenant pays sits below the market.

**Live app:** <https://celnic-housing-nz.streamlit.app>

---

## What this is

New Zealand publishes a great deal about public housing, but it is scattered
across four agencies, three file formats and a decade of renamed publications.
This project collects all of it, reconciles it, and puts it behind one interface.

| Dataset | Source | Coverage | Format |
|---|---|---|---|
| Rental bond data | MBIE / Tenancy Services | Feb 1993 – Jun 2026, monthly | CSV |
| Kāinga Ora managed stock | Kāinga Ora | 2015Q3 – 2025Q3, 41 quarters | PDF + XLSX |
| Housing Register | MSD | 2014Q2 – 2026Q2, 49 quarters | XLSX |
| Emergency housing grants | MSD | 2022Q1 – 2026Q2 | XLSX |
| Public Housing Quarterly Report | HUD | 2020Q4 – 2023Q4 | PDF |

404 files, 197 MB, reconciled into 111 RAW tables and then into the marts the app
reads. **324 of 324 publication period slots are present — no gaps.**

Where the public record stops, synthetic data takes over, seeded so its totals
reconcile back to published figures. Every synthetic surface is banner-marked.

---

## The seven tabs

1. **National Overview** — supply, register and market rent since 2014, from the
   published record.
2. **Stock Map** — H3 hexagons at resolutions 8–12. Property *counts* per
   district and bedroom count are real; *locations* are modelled.
3. **Market Rent vs Income-Related Rent** — the centrepiece. Real market rent
   against modelled income-related rent, and the subsidy gap between them.
4. **Housing Register** — register trend, priority split, and the bedroom
   mismatch between who is waiting and what exists. Both series are real.
5. **Asset & Maintenance** — fully synthetic. No agency publishes property
   condition or maintenance at any grain.
6. **Pipeline** — every source with its URL, licence, download date and
   checksum; the real vs synthetic lineage; and the reconciliation checks.
7. **Build Notes** — the Phase Two engineering write-up, rendered from
   [README_PHASE_TWO.md](README_PHASE_TWO.md): how the transformation layer was
   designed from the app spec, the mart contract, the synthetic data, and what
   deployment taught.

---

## Real, derived, synthetic

The app labels all three, everywhere. The distinction is the point of the build.

**Real** — bond rents, Kāinga Ora stock by district and bedroom count, the
Housing Register and its priority and bedroom splits, emergency housing grants,
HUD's headline series to 2023Q4.

**Derived** — market rent by territorial authority *and* bedroom count. MBIE
publishes rent by TA without a bedroom split, and by bedroom against ~2,000
six-digit area codes for which it publishes no code-to-name lookup. That cut does
not exist in the public files, so it is modelled by scaling each district's
measured rent by the national bedroom relativity, and labelled derived.

**Synthetic** — property locations, condition scores, tenancy-level rents,
applicant records, and all maintenance. Kāinga Ora publishes nothing below
region-by-bedroom.

The synthetic portfolio matches the published stock table **exactly, in every
district and bedroom count** — 72,951 properties — and the synthetic register
matches MSD's published totals in every territorial authority.

---

## How the numbers were checked

Seventeen reconciliation checks run in the pipeline and are shown in the app. The
ones that matter are cross-source, because a parser can agree with itself while
being wrong.

An early version of this pipeline dropped four territorial authority rows in five
from the 2015–2019 Kāinga Ora PDFs — those releases print "none" as a
box-drawing character rather than a hyphen, so every row containing a zero failed
to parse. The surviving rows still summed to their own totals perfectly. What
caught it was comparing Kāinga Ora's stock against the figure **HUD** publishes
for Kāinga Ora: different agency, different document, different format, no shared
code path. Those two now agree exactly in two of the three overlapping quarters
and to within 21 homes in the third.

---

## Running it locally

```bash
pip install -r requirements.txt
streamlit run app/kainga_ora_housing_nz.py
```

The app reads `data/kainga_ora_housing_public.duckdb` (18 MB) read-only. No
credentials, no network access, no API keys.

---

## Caveats worth knowing before you cite anything

- **Bond data is provisional.** Tenancy Services migrated to Bond Hub (fully
  online from 29 June 2026). Recent periods draw on both the legacy and new
  systems and are subject to revision, with a known discontinuity of roughly
  17,550 extra active bonds from recording differences.
- **Market rent excludes government properties** by design — that is what makes
  it a valid comparator for social housing.
- **HUD's Public Housing Quarterly Report ceased after December 2023.** Its
  successor is an embedded Power BI dashboard with no export, so CHP homes,
  transitional housing and Housing First stop at 2023Q4.
- **MSD randomly rounds to base 3 and suppresses small counts.** Summing
  territorial authorities understates the national total slightly. Suppressed
  cells are preserved as null, never as zero.
- **Nothing here is an official statistic.**

---

## Licence and attribution

Application code: MIT. Derived data: Creative Commons Attribution, per the
originating agencies. See [LICENSE](LICENSE).

Independent build by Celnic Consulting. Enquiries and corrections are welcome
through the issues page of this repository.

Not affiliated with, endorsed by, or produced on behalf of Kāinga Ora, MSD,
HUD/MCERT or MBIE.
