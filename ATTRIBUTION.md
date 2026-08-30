# Attribution

This repository demonstrates a data-platform build method. It is **not**
an official publication of any agency named below, and no agency has
endorsed it. Data has been **modified** (downloaded, staged, transformed,
and in places mixed with synthetic records) — treat every figure as
untrusted demonstration output.

## Source datasets

| Dataset | Publisher | Licence | Source |
|---|---|---|---|
| Rental bond data | MBIE / Tenancy Services | CC BY 4.0 | [tenancy.govt.nz](https://www.tenancy.govt.nz/about-tenancy-services/data-and-statistics/rental-bond-data/) |
| Public Housing Quarterly Report and regional factsheets | Ministry of Housing and Urban Development (HUD) | CC BY 4.0 | [hud.govt.nz](https://www.hud.govt.nz/stats-and-insights/public-housing-quarterly-reports) |
| Housing Register and Transfer Register | Ministry of Social Development | **CC BY 3.0 NZ — licence to confirm** | [msd.govt.nz](https://www.msd.govt.nz/about-msd-and-our-work/publications-resources/statistics/housing/housing-register.html) |
| MSD housing support statistics (emergency housing, monthly and quarterly reporting, factsheets) | Ministry of Social Development | **CC BY 3.0 NZ — licence to confirm** | [msd.govt.nz](https://www.msd.govt.nz/about-msd-and-our-work/publications-resources/statistics/housing/emergency-housing.html) |
| Kāinga Ora housing statistics (managed stock, vacant stock, homes removed from service) | Kāinga Ora – Homes and Communities | CC BY 4.0 | [kaingaora.govt.nz](https://kaingaora.govt.nz/en_NZ/publications/oia-and-proactive-releases/housing-statistics/) |
| Statistical Area 2 / TA boundaries and centroids | Stats NZ | **UNVERIFIED — not retrieved, not redistributed** | [datafinder.stats.govt.nz](https://datafinder.stats.govt.nz/) |

All source files were retrieved on 2026-08-23. Per-dataset detail, including
the working-tree files behind each entry, is in
[DATA_SOURCES.yaml](DATA_SOURCES.yaml).

## Licence to confirm

Three registry entries do not carry a plain CC BY 4.0 licence and are flagged
`licence_issue: true` in [DATA_SOURCES.yaml](DATA_SOURCES.yaml):

- **MSD Housing Register / Transfer Register** — released under **CC BY 3.0 NZ**,
  not 4.0. Attribution-only and compatible with redistributing derived work, but
  it must be cited as CC BY 3.0 NZ rather than swept into a blanket CC BY 4.0
  statement. *Recommendation: keep, with this dataset-specific attribution.*
- **MSD housing support statistics** — same position, **CC BY 3.0 NZ**.
  *Recommendation: keep, with this dataset-specific attribution.*
- **Stats NZ geography (registry entry D16)** — recorded in the source registry
  as CC BY 4.0 NZ but **never retrieved**: the Geographic Data Service requires a
  per-account API key, so nothing was downloaded and no Stats NZ data is
  redistributed here. The territorial authority and Auckland local board
  centroids the map uses are a Celnic-curated reference table, not Stats NZ data.
  *Recommendation: drop the entry from the registry, or retrieve and verify the
  licence before claiming it.*

Source data © the named publishers, used under CC BY 4.0
(https://creativecommons.org/licenses/by/4.0/), except the two MSD datasets
above, used under CC BY 3.0 NZ
(https://creativecommons.org/licenses/by/3.0/nz/). Attribution does not
imply endorsement. Synthetic records are generated and carry no
statistical meaning.
