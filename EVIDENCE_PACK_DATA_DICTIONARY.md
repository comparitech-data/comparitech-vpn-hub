# VPN Benchmark — Data Dictionary

This dataset is an independent comparison of consumer VPN providers. It pairs
**controlled performance measurements** — run on a fleet of real machines — with
**researched provider facts**, and distils them into 0–10 scores across six
categories plus the underlying evidence for each score.

It ships as two site-wide CSV files, plus — for each use-case article (e.g.
"best VPN for Netflix") — an optional **per-use-case scorecard pack** carrying
the exact data behind that page's chart (see File 3). This document defines every
field and explains how to read the numbers, so an automated reader can answer
questions about a provider *and* judge how much to trust each value.

| File | One row per | Contains |
|---|---|---|
| `Comparitech_Raw_Telemetry.csv` | provider | **The scorecard** — the six category scores + overall, plus headline facts (price, speed, servers, no-logs status…). |
| `Comparitech_Feature_Scores_Long.csv` | provider × criterion × protocol | The graded evidence behind every score. |
| `logging_policies.json` | provider | Full no-logs / logging-policy prose, keyed by provider — kept out of the flat CSV so verbose quoted text can't break naive parsers/spreadsheets. |
| `Comparitech_Streaming_Scorecard.csv` | provider × service × region | **One global** streaming-test evidence file (working/partial/blocked counts + latest verdict). Every use-case page links this same file; streaming verdicts are global telemetry, not page-specific (File 3). |
| `use-cases/<slug>_scorecard.{json,csv}` | use case (article) | A page-specific chart's own data — its criteria, per-provider point contributions, and curated provider set. Streaming-related pages reference the global streaming scorecard via `files.streaming_scorecard_csv`. |

All values are read live from the benchmark database at the moment of download.

---

## How to read a score

- Every score is **0–10, higher is better**.
- Each provider is scored in **six categories**:

  | Category | What it rewards |
  |---|---|
  | **Speed** | Fast, consistent download/upload, low latency, low loss — measured. |
  | **Streaming** | Reliably unblocks major streaming services — measured. |
  | **Security** | Passes leak tests, independently audited, privacy-friendly jurisdiction & payments. |
  | **System Performance** | Adds little CPU/RAM overhead to the device — measured. |
  | **Value for Money** | Price across terms, refund window, connection limit, bundled extras. |
  | **Ease of Use** | App/platform coverage, extensions, router support, support quality. |

- A category score is built from **weighted sub-criteria**. The feature file shows
  each criterion, the value we recorded for it, and the points it contributed —
  so every category score is auditable, not a black box.
- The six category scores are combined into a single **overall** rating (see below).
- **Where to find them:** the **scorecard** (`Comparitech_Raw_Telemetry.csv`) carries
  each provider's six category scores and the overall. The **feature file**
  (`Comparitech_Feature_Scores_Long.csv`) is the breakdown — the individual criteria
  that produced each category score.

### How the overall is calculated

The overall is the **equal-weight average of a provider's scored categories**, each
first placed on a 0–10 scale:

- **Security, Streaming, System Performance, Value for Money, Ease of Use** enter at
  face value (they are already 0–10).
- **Speed** is placed on the scale using a **fixed reference band** for the quarter
  (currently `5.0–8.5`; the exact band + quarter are stamped in the Snapshot at the
  end of this download, or in the file pack's `manifest.json`). A Speed at or below
  the band's floor contributes 0, at or above its ceiling contributes 10, linearly
  in between — so Speed is scored *relative to the field*, not absolute.

There are **no hidden per-category weights** — every scored category counts equally.
Categories a provider isn't scored on are **left out of the average** (they neither
help nor hurt); `Categories scored (of 6)` tracks how many counted.

Because of the Speed band, the overall is **not** the plain mean of the six printed
category numbers. Example: a provider whose six scores average 7.82 can show an
overall of 7.98 when its Speed sits high in the band — or below its mean when its
Speed sits low.

---

## Trust: how each value was obtained (the `Provenance` field)

This is the heart of the dataset. Every graded value is labelled by **how we know
it** — so a controlled measurement is never confused with a vendor claim. In the
feature file this is the `Provenance` column; for the telemetry file the table in
§ "File 1" gives the equivalent per column.

| Provenance | Meaning | Trust it for |
|---|---|---|
| `benchmark_run` | **Measured** by us in a controlled test on the device fleet. | Real-world speed, streaming unblocking, leak results. |
| `manual_admin` | **Analyst-verified** — entered/confirmed by a human reviewer and never auto-overwritten. | Facts we deliberately checked (e.g. current pricing). |
| `mixed` | A measured value with an analyst correction layered on. | Same as measured, human-reviewed. |
| `editorial_default` | **Researched** from the provider's own documentation/specifications. | Catalogue facts (devices, audits, jurisdiction). |
| `comparitech_api` | Imported from Comparitech's maintained VPN catalogue. | Catalogue facts. |
| `calculated` | **Derived** by the scoring engine — the category roll-up rows, computed from the graded criteria above. | The published category totals. |

**Rule of thumb for a consuming model:** weight `benchmark_run` / `manual_admin`
above `editorial_default` / `comparitech_api` when a performance claim and a vendor
claim disagree. An `editorial_default` tag is not a weakness — much of a VPN's
profile (price, jurisdiction, refund window) genuinely is documentation, not
something you measure — it simply keeps researched facts from posing as measured
ones.

**Which pillars are measured vs researched:** **Speed**, **Streaming** and **System
Performance** are scored from controlled fleet measurements; **Value for Money** and
**Ease of Use** from researched facts; **Security** is mixed (leak tests are
measured; audit recency, jurisdiction and no-logs status are researched). So a
provider's Speed/Streaming/System-Performance numbers carry more measurement weight
than its Value/Ease numbers.

---

## File 1 — `Comparitech_Raw_Telemetry.csv` (the scorecard)

One row per provider: its scores first, then its headline facts. An empty cell means
**not available / not yet measured** — never read it as a zero.

| Column | Meaning | Type |
|---|---|---|
| `Provider` | Provider name. | — |
| `Overall score /10` | The overall rating — the six category scores combined. | Score |
| `Security /10`, `Speed /10`, `Streaming /10`, `System Performance /10`, `Value for Money /10`, `Ease of Use /10` | The six category scores (0–10, higher is better). Empty = that category is not scored yet for this provider. | Score |
| `Categories scored (of 6)` | How many of the six categories had enough data to be scored — a completeness signal for the overall. | — |
| `Speed countries covered` | How many of the target speed-test countries were actually measured, e.g. `6 of 7`. The feature file's `_coverage` row lists which (and which are missing). Empty = Speed not measured yet. | Measured |
| `Last speed measurement (UTC)` | When the provider's most recent speed sample was actually taken on the fleet — the real freshness anchor, **not** the score-recompute time. | Measured |
| `Netflix`, `Disney+`, `BBC iPlayer`, `ITVX` | Whether the VPN unblocked each service on its most recent test: `working`, `blocked`, or `partial`. **Blank = not tested in the window — treat as unknown, not a failure.** | Measured |
| `Streaming services last tested (UTC)` | When the per-service checks above were last run (14-day window, same as the Streaming score, so they can't contradict it). | Measured |
| `Lowest monthly price — intro (USD)` | Cheapest monthly-equivalent on the introductory term. | Researched |
| `Lowest monthly price — renewal (USD)` | Monthly-equivalent once the intro term ends. | Researched |
| `Standard monthly price (USD)` | Month-to-month price with no commitment. | Researched |
| `Devices Supported` | Platforms with a native app/extension. | Provider-stated |
| `Median download Mbps (best protocol, benchmark)` | **Measured** median download throughput on the provider's fastest protocol. Empty = not yet benchmarked. | Measured |
| `Countries Supported` | Advertised number of countries with servers. | Provider-stated |
| `Servers` | Advertised total server count. | Provider-stated |
| `No-logs status` | Verification level of the no-logs claim: `audited` (independently audited), `claimed` (vendor states it, not independently audited), or `logs_kept` (does retain logs). The full policy prose is in **`logging_policies.json`** (keyed by provider), deliberately kept out of this CSV so quoted multi-sentence text doesn't break flat-file parsers. | Researched |
| `Simultaneous Connections` | Devices connectable at once (number or `Unlimited`). | Provider-stated |
| `Port forwarding (Yes/No)` | Supports port forwarding. | Researched |
| `SmartDNS (Yes/No)` | Offers a Smart DNS service. | Researched |

*"Provider-stated" = the provider's own published figure, recorded by our analysts.
"Researched" = gathered from the provider's documentation. "Measured" = produced by
our own controlled test.*

---

## File 2 — `Comparitech_Feature_Scores_Long.csv`

One row per graded criterion. This is the evidence behind each category score.

| Column | Meaning |
|---|---|
| `Provider` | Provider name. |
| `Category` | One of the six categories above. |
| `Feature` | The criterion (e.g. `Money back duration`, `Four k capable speed`). Rows whose name begins with `_` are scoring roll-ups — see below. |
| `Protocol` | `Overall` = the headline value for the provider. A named VPN protocol (e.g. `WireGuard`, `OpenVPN (UDP)`) = that protocol's own result. Only Speed / System-Performance criteria are measured per protocol; everything else carries only `Overall`. |
| `Raw_Value` | The recorded value, as a labelled bucket — e.g. `tier_good`, `30_plus_days`, `yes`. Booleans appear as `Yes`/`No`. |
| `Score` | The points this criterion contributed to its category (0 = no credit). Empty = informational row, not graded. |
| `Provenance` | How the value was obtained — see the Trust table above. |

**Read the `Overall` rows** for a provider's headline numbers; the named-protocol
rows are the per-protocol breakdown.

**Roll-up rows (`Feature` begins with `_`)** are scoring internals, included so the
math is transparent:

| Row | Meaning |
|---|---|
| `_section_total` | The matrix-graded portion of the category. For Security, Value for Money and Ease of Use this equals the category score; for the measured categories (Speed, Streaming, System Performance) it is only the editorial add-on — **use the scorecard file for the authoritative category /10.** |
| `_…_v3_…`, `_speed_v3_…` | The measured components of the Speed / Streaming / System-Performance scores. |
| `_coverage` | Coverage of the category's measurement panel, as JSON in `Raw_Value` — e.g. Speed records `count`/`expected` countries, the `countries` actually covered, and the full `panel`. Below the bar a category is withheld rather than scored on thin data. The scorecard's `Speed countries covered` summarises the Speed row. |
| other `_…` rows | Intermediate sub-components. Safe to ignore for a top-level read. |

---

## File 3 — per-use-case scorecard packs (`use-cases/<slug>_*`)

Each use-case article has its own evidence pack: the exact data behind the chart
embedded on that page, so its specific claims are verifiable without
reverse-engineering. Present only for use cases that have been generated; one set
of files per article, named by the article slug.

**Why this is separate from File 1.** The scorecard above is the *site-wide*
view — one overall row per provider, one streaming status per service. A use-case
chart is a *different, use-case-specific* composite, scored only on the criteria
chosen for that page (e.g. Netflix US unblock rate, simultaneous connections, US
speed retention, price) and shown for a provider set curated for that page. **A
use-case score is therefore not the provider's overall VPN score** — the pack
carries its own columns and a per-provider `overall_score` so the two reconcile.

### `<slug>_scorecard.json` (primary, machine-readable)

| Field | Meaning |
|---|---|
| `use_case`, `article_url` | The page this pack backs. |
| `snapshot` | `{ id, version, generated_at, locked_at }` — the exact chart version and when it was scored (the freshness anchor for the page). |
| `scoring_template` | `{ model_id, prompt_version, umbrella, umbrella_version }` — the rubric version, so the result is reproducible. |
| `composite_basis` | Always `use_case_specific_not_overall_vpn_score` — a reminder this score ≠ the provider's overall rating. |
| `price_basis` | Basis of any price criterion — `intro_monthly_equivalent` (cheapest introductory monthly-equivalent), so an intro price is never mistaken for a renewal price. |
| `providers_scored_total`, `providers_shown` | How many providers were scored vs shown on the chart — explains an "N benchmarked, M shown" gap. |
| `dimensions[]` | Each scored criterion: `field`, `label`, `weight`, `curve_id` (+ `price_basis` on price fields). |
| `inclusion` | `{ policy, included[], excluded[ { provider_slug, reason, type } ] }`. `type` is `auto_incomplete_data` (provider missing data on a scored criterion — can't be honestly ranked here) or `editorial` (an analyst dropped it, with a reason). |
| `providers[]` | Per provider: `provider_slug`, `display_name`, `rank`, `composite_score`, `included_in_chart`, `exclusion_reason`, **`overall_score`** (the site-wide rating — the bridge to File 1 and the hub), and `dimensions{ field → { raw, display, points, sample_count, state } }`. |

`display` is the reader-facing rendering of `raw` (e.g. the `999` "unlimited"
connections sentinel shows as `Unlimited`); `points` is what that criterion
contributed to the composite; `state` is `scored` or `missing_data`.

### `<slug>_scorecard.csv` (flat)

The same per-provider rows as a spreadsheet: `provider_slug`,
`provider_display_name`, `rank`, `composite_score`, `included_in_chart`,
`exclusion_reason`, then `<field>__raw` / `<field>__display` / `<field>__points`
for each criterion, then `overall_score` and `price_basis`.

### `Comparitech_Streaming_Scorecard.csv` (global per-region streaming evidence)

One global file (at the pack root) holding the raw streaming results behind every
page's per-region / per-service claims — one row per (provider, service, target
region). Streaming-related use-case pages link this same file via
`files.streaming_scorecard_csv`; there is no per-page copy:

| Column | Meaning |
|---|---|
| `provider_slug`, `provider_display_name` | Provider. |
| `service`, `target_region` | e.g. `netflix` / `US`. |
| `runs` | Distinct test sessions (soaks) in the 14-day window. |
| `verdict_tests`, `working`, `partial`, `blocked` | Verdict-bearing tests and their outcomes. Infra/detection errors are excluded, never counted as failures. |
| `latest_status`, `last_tested_utc` | Most recent verdict and when it ran. |
| `is_proxied` | `true` for synthetic rows — currently Hulu, measured via the Disney+ US bundle rather than tested directly. |

These are **raw verdict counts**. The Streaming *score* applies STREAMING-V3.1
run-grouping + transient tolerance on top (a single transient `blocked` need not
lower a published unblock rate), so a 100% chart figure can legitimately coexist
with an occasional `blocked` row here.

---

## How the measurements are made (why the scores are credible)

- **Speed** — download, upload, latency, jitter and packet loss measured across
  multiple countries and multiple VPN protocols on real machines, repeated over
  time. Latency is corrected for the test network's own overhead, only trusted
  samples count, and a provider with too few samples is withheld rather than
  guessed.
- **Streaming** — we attempt to actually play content on major services (Netflix,
  Disney+, BBC iPlayer, and others) through each VPN, repeatedly over a rolling
  window, weighted by how in-demand each service is.
- **Security** — automated leak tests (DNS, IPv4, IPv6, WebRTC) reported with
  statistical confidence, plus researched independent-audit recency, headquarters
  jurisdiction, and anonymous-payment options. When a provider *persistently*
  misroutes (advertises country X, repeatedly exits country Y, rDNS-confirmed),
  an `advertised_location_accuracy` row records the location integrity (0.5–1.0)
  and the scorer applies a capped Security deduction (≤0.5 pt). Like the
  server-location findings it derives from, it is **withheld pending responsible
  disclosure** (note 5) — absent until that surface goes public.
- **System Performance** — the extra CPU and memory the VPN app consumes versus an
  idle baseline on the same machine, measured like-for-like.
- **Value for Money** and **Ease of Use** — researched catalogue facts (pricing,
  terms, platform coverage, support), graded against fixed criteria.

---

## Interpreting it correctly — read this before drawing conclusions

1. **Empty = not measured yet, not zero.** A blank speed or a missing criterion
   means the provider hasn't been tested for it — not that it failed. Criteria
   deliberately emit nothing when data is absent rather than a misleading default.
2. **Use the `Overall` protocol rows** for provider-level numbers; named protocols
   are a breakdown.
3. **For headline scores use the scorecard file** (`Comparitech_Raw_Telemetry.csv`);
   the feature file is the per-criterion breakdown behind them.
4. **Weight by `Provenance`** — measured/analyst-verified above researched/catalogue
   when claims conflict.
5. **Some findings are intentionally withheld** (certain leak / server-location
   results are held pending responsible disclosure) and will simply be absent.
6. **Freshness** — values are current as of download; benchmarking runs
   continuously, so figures move as new measurements land. For real per-provider
   freshness use `Last speed measurement (UTC)` (when the fleet last sampled the
   provider), not an assumption that everything is current. The **Snapshot** at
   the end of this download (or the file pack's `manifest.json`) stamps the export
   time and the Speed reference band/quarter in effect.

## Independence and ownership

Comparitech is owned by Point Wild. Point Wild also owns several VPN products. We do not score, rank or list those products on the VPN Intelligence Platform, because no disclosure makes a ranking of a parent-owned product read as independent, and they are not included in these files. Earlier versions of these files remain in this repository's history. See https://vpn.comparitech.com/methodology#independence-and-ownership
