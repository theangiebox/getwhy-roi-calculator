# GetWhy Business Value Calculator

An interactive, single-page calculator that estimates the business value of replacing
traditional agency research with GetWhy. A visitor sets three inputs and sees a
three-year business case: total annual benefit, ROI, payback, and net benefit.

- **Live:** https://theangiebox.github.io/getwhy-roi-calculator/
- **Hosting:** GitHub Pages, served from the `main` branch root. Every push to `main`
  deploys automatically.

## Quick start

There is no build step and there are no dependencies. It is one self-contained file.

```bash
git clone https://github.com/theangiebox/getwhy-roi-calculator.git
cd getwhy-roi-calculator
open index.html          # or just double-click it
```

To preview the way GitHub Pages serves it (recommended before pushing):

```bash
python3 -m http.server 8000
# visit http://localhost:8000
```

## How it is built

- **`index.html` is the entire app.** Markup, CSS (in a `<style>` block), and logic
  (in a `<script>` block at the bottom) all live in this one file.
- **Fonts and images are embedded as base64 data URIs**, which is why the file is
  ~1.2 MB despite being under 900 lines. The brand fonts (Aeonik, EdictDisplay), the
  logo, and the hero photos are all inline. A reasonable refactor is to move these into
  an `/assets` folder and link them, which would shrink the HTML dramatically.
- **No framework.** Plain HTML/CSS and vanilla JavaScript.

## The calculation model

All logic is in the `<script>` block. The constants are declared at the top with
comments and are sourced from the Nucleus Research *"GetWhy ROI business case tool
(DRAFT V3)"* spreadsheet.

### Inputs (3 sliders)

| Input | id | Range | Default |
| --- | --- | --- | --- |
| Annual agency / panel spend | `agency` | $50K – $2M | $200,000 |
| People on the insights team | `insight` | 1 – 25 | 3 |
| Decision-makers who rely on research | `dm` | 10 – 500 | 100 |

### Fixed assumptions (held at spreadsheet defaults, disclosed in the UI footnote)

| Constant | Value | Meaning |
| --- | --- | --- |
| `AGENCY_ELIM` | 0.97 | Share of agency fees GetWhy eliminates |
| `TEAM_PRODUCTIVITY` | 0.70 | Researcher time freed |
| `DM_PRODUCTIVITY` | 0.05 | Decision-maker productivity lift |
| `INSIGHT_COST` | $80,000 | All-in annual cost per researcher |
| `STAKEHOLDER_COST` | $120,000 | All-in annual cost per decision-maker |
| `WORK_HOURS` | 2,080 | Working hours per FTE per year |
| `PLATFORM_INITIAL` / `RETAINER_ANNUAL` | $100K / $50K | GetWhy platform + retainer |
| `PROFSVC_INITIAL` / `PROFSVC_ANNUAL` | $30K / $10K | Professional services |
| `STAFF_LOADED`, `DEPLOY_HOURS`, `ONGOING_FTE` | $90K, 100, 0.2 | Internal deployment + upkeep |

### Formulas

```
agencyBenefit  = agencySpend × AGENCY_ELIM
teamBenefit    = insightPros × INSIGHT_COST × TEAM_PRODUCTIVITY
dmBenefit      = dmCount     × STAKEHOLDER_COST × DM_PRODUCTIVITY
annualBenefit  = agencyBenefit + teamBenefit + dmBenefit
hrsRecovered   = insightPros × WORK_HOURS × TEAM_PRODUCTIVITY

INITIAL_COST   = PLATFORM_INITIAL + PROFSVC_INITIAL + (DEPLOY_HOURS × STAFF_LOADED / WORK_HOURS)   # ≈ $134,327
ANNUAL_COST    = RETAINER_ANNUAL + PROFSVC_ANNUAL + (ONGOING_FTE × STAFF_LOADED)                    # $78,000

threeYrBenefit = annualBenefit × 3
threeYrCost    = INITIAL_COST + ANNUAL_COST × 3        # = $368,327, matches the spreadsheet exactly
roi            = (threeYrBenefit − threeYrCost) / threeYrCost
paybackMonths  = (INITIAL_COST + ANNUAL_COST) / (annualBenefit / 12)
netAnnual      = annualBenefit − ANNUAL_COST
```

At default inputs this yields ~$962K annual benefit, ~684% three-year ROI, ~2.6-month
payback. This deliberately omits the spreadsheet's small "reduced cloud" and "other"
streams, so the benefit is slightly more conservative than the sheet's $982K/year.

### Live header tiles

The two model-derived tiles in the top proof strip (`#ps-benefit`, `#ps-hrs`) recompute
on every slider change, so the header can never drift from the results below. The `84%`
and `5x` tiles are fixed, Nucleus-verified benchmark rates and are not input-dependent.

## Known stub / next steps for a developer

- **The lead-gen form is front-end only.** `handleSubmit()` validates the email and
  shows a success message; it does not send anything. Wire this to a CRM / marketing
  automation endpoint or a form service.
- **Externalize the embedded assets** (fonts, logo, hero images) into `/assets` to cut
  the HTML payload.
- **Optional:** split the CSS and JS into separate files if the team prefers.

## Copy / brand notes

This is customer-facing GetWhy copy. House style: no em dashes (use periods or colons),
plainspoken, lists in threes. The benchmark figures (84% lower cost, 5x faster, 80%
time-to-insight reduction) are cited to Nucleus Research, Document 2600X; the dollar and
hour figures are modelled from those benchmarks applied to the visitor's inputs.
