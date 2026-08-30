# Attribution

This repository demonstrates a data-platform build method. It is **not**
an official publication of any agency named below, and no agency has
endorsed it. Data has been **modified** (downloaded, staged, transformed,
and in places mixed with synthetic records) — treat every figure as
untrusted demonstration output.

## Source datasets

Every licence below was read from the publisher on **2026-08-30** and the page
it was read from is recorded. Nothing here is inferred from a sibling dataset or
carried over from an earlier registry.

| Dataset | Publisher | Licence | Read from | Source |
|---|---|---|---|---|
| Rental bond data | MBIE / Tenancy Services | CC BY 3.0 NZ | [dataset page](https://www.tenancy.govt.nz/about-tenancy-services/data-and-statistics/rental-bond-data/) · [catalogue record](https://catalogue.data.govt.nz/dataset/rental-bond-data-by-quarter-detailed) | [tenancy.govt.nz](https://www.tenancy.govt.nz/about-tenancy-services/data-and-statistics/rental-bond-data/) |
| Public Housing Quarterly Report and regional factsheets | Ministry of Housing and Urban Development (HUD), now MCERT | CC BY 4.0 † | [agency copyright statement](https://www.hud.govt.nz/about-us/copyright-and-disclaimer) | [hud.govt.nz](https://www.hud.govt.nz/stats-and-insights/public-housing-quarterly-reports) |
| Housing Register and Transfer Register | Ministry of Social Development | CC BY 4.0 | [catalogue record](https://catalogue.data.govt.nz/dataset/social-housing-register-december-2020) | [msd.govt.nz](https://www.msd.govt.nz/about-msd-and-our-work/publications-resources/statistics/housing/housing-register.html) |
| MSD housing support statistics (emergency housing, monthly and quarterly reporting, factsheets) | Ministry of Social Development | CC BY 4.0 † | [agency copyright statement](https://www.msd.govt.nz/about-msd-and-our-work/tools/copyright-statement.html) | [msd.govt.nz](https://www.msd.govt.nz/about-msd-and-our-work/publications-resources/statistics/housing/emergency-housing.html) |
| Kāinga Ora housing statistics (managed stock, vacant stock, homes removed from service) | Kāinga Ora – Homes and Communities | **UNVERIFIED** ⚠ | — (publisher unreachable) | [kaingaora.govt.nz](https://kaingaora.govt.nz/en_NZ/publications/oia-and-proactive-releases/housing-statistics/) |
| TA and Auckland local board reference centroids | Celnic Consulting (curated) | CC0 | own work | — |
| Synthetic demonstration data | Celnic Consulting (generated) | CC0 | own work | — |

† The licence was read from the publisher's site-wide copyright statement, not
from a statement specific to this dataset. The dataset page links to that
statement but does not repeat it, and the dataset has no catalogue record of its
own. The licence is not in doubt; its specificity to this dataset is.

⚠ No licence could be established. No licence is claimed for this data. See
below.

All source files were retrieved on 2026-08-23. Per-dataset detail — the exact
licence quote, the URL it was read from, the `licence_basis`, and the
working-tree files behind each entry — is in
[DATA_SOURCES.yaml](DATA_SOURCES.yaml).

## What verification changed

Three of the five recorded licences were wrong. Both corrections went in the
direction the old manifest did not expect: the dataset it had flagged as a
problem was fine, and the ones it had waved through were not.

- **Rental bond data** was recorded as CC BY 4.0. It is **CC BY 3.0 NZ**. The
  tenancy.govt.nz page says so directly — *"This work is licensed under a
  Creative Commons Attribution 3.0 New Zealand License"* — and all three
  data.govt.nz catalogue records for this data carry `licence_id`
  `CC-BY-NZ-3.0`. Still attribution-only and still safe to redistribute in
  derived form, but it must be cited as CC BY 3.0 NZ.
- **Both MSD datasets** were recorded as CC BY 3.0 NZ and flagged. They are
  **CC BY 4.0**. MSD's copyright statement reads *"licensed for re-use under
  Creative Commons Attribution (CC-BY) 4.0 International Licence"*, and the
  Social Housing Register catalogue records switched from `CC-BY-NZ-3.0` to
  `CC-BY-4.0` at the March 2018 release. The old flag is cleared.
- **Kāinga Ora housing statistics** was recorded as CC BY 4.0 on no evidence.
  That assertion has been **withdrawn**, not restated — see below.

## Licence to confirm

Three entries remain flagged `licence_issue: true` in
[DATA_SOURCES.yaml](DATA_SOURCES.yaml):

- **Kāinga Ora housing statistics** — **unverified**. kaingaora.govt.nz sits
  behind an Imperva/Incapsula web application firewall that refuses non-browser
  clients: the statistics page, the site root, and the copyright and terms pages
  all return a challenge document instead of content. Kāinga Ora has no
  organisation entry on data.govt.nz and no catalogue record for these
  statistics. No licence statement could be read, so **no licence is claimed for
  this data**. This is the largest open question in the repository — it is the
  namesake dataset, and derived figures from it are published here.
  *Recommendation: open the page in a browser, read the copyright statement, and
  record the quote and URL in DATA_SOURCES.yaml. Until then, treat this data as
  unlicensed for redistribution.*
- **Public Housing Quarterly Report (HUD/MCERT)** — CC BY 4.0, but read only
  from the agency-wide copyright page. The report page carries no statement of
  its own and has no catalogue record. HUD's functions moved to the Ministry for
  Cities, Environment, Regions and Transport on 1 July 2026; as at 2026-08-30
  the page still resolves under hud.govt.nz, is branded MCERT, and states the
  products *"have now been discontinued and no further releases will be made"*.
  The licence statement did not change with the move.
  *Recommendation: keep, cited as CC BY 4.0 at agency level.*
- **MSD housing support statistics** — CC BY 4.0, agency-level only. The
  emergency-housing page carries no statement of its own and MSD's catalogue
  records do not cover this series.
  *Recommendation: keep, cited as CC BY 4.0 at agency level.*

The former **Stats NZ geography** entry (D16) has been **removed** from the
manifest. It was registered as CC BY 4.0 NZ but never retrieved — the Geographic
Data Service requires a per-account API key — so nothing was downloaded and no
Stats NZ data is redistributed here. There is nothing to attribute. The
territorial authority and Auckland local board centroids the map uses are a
Celnic-curated reference table, not Stats NZ data. The entry and the reason for
dropping it are recorded under `removed_entries` in
[DATA_SOURCES.yaml](DATA_SOURCES.yaml).

## Attribution statement

Source data © the named publishers:

- Rental bond data © Ministry of Business, Innovation and Employment, used under
  **CC BY 3.0 NZ** (https://creativecommons.org/licenses/by/3.0/nz/).
- Housing Register, Transfer Register and housing support statistics © Ministry
  of Social Development, used under **CC BY 4.0**
  (https://creativecommons.org/licenses/by/4.0/).
- Public Housing Quarterly Report © Ministry of Housing and Urban Development /
  Ministry for Cities, Environment, Regions and Transport, used under
  **CC BY 4.0** (https://creativecommons.org/licenses/by/4.0/).
- Kāinga Ora housing statistics © Kāinga Ora – Homes and Communities. **Licence
  unverified — none claimed.**

All are modified. Attribution does not imply endorsement. Synthetic records are
generated and carry no statistical meaning.
