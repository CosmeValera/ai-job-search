---
name: milan
description: >
  Runs a job search scoped to Milan and the Ring 1 commuter belt only (Monza/Brianza,
  Bergamo, Como, Lecco, Varese, Novara, Pavia). Excludes Berlin, Ring 2 and Tier 2
  cities. Triggers on: milan jobs, jobs in milan, search milan, milano, /milan
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(bun --version), Bash(bun run .agents/skills/*/cli/src/cli.ts *), WebFetch, WebSearch, Agent, AskUserQuestion
---

# Milan-Only Job Search

Scoped variant of `/scrape`. Same pipeline, one geography.

## Scope

**In scope**
- Milan / Milano (primary)
- **Ring 1 commuter belt**: Monza/Brianza, Bergamo, Como, Lecco, Varese, Novara, Pavia — scored as Milan per `04-job-evaluation.md`
- Fully remote within Italy, or fully remote EU where the company has a Milan presence

**Out of scope for this run** — drop silently, do not present, do not write to `seen_jobs.json` as new:
- Berlin and everything German
- Ring 2 (Turin, Genoa, Bologna, Verona, Trento, Bolzano) — use `/scrape` or `/milan-berlin` if the wider net is wanted
- Tier 2 (Dublin, Amsterdam, Munich, Luxembourg, Zurich, Geneva)
- Spain — hard exclude, always

## Re-application cooldown — apply and restate every single run

**Never surface a position already applied to within the last 12 months.** This is not optional and not a judgement call. State the rule and the cutoff date in the output header on every run, even when nothing was filtered by it.

Build the exclusion set from `job_search_tracker.csv` (columns: `date,company,sector,role,role_type,channel,status,contact_person,fit_rating,notes,cv_file,cover_letter_file,source`):

| Tracker row | Handling |
|---|---|
| Same company + role, `date` **within the last 12 months** | **Exclude.** Do not present, do not count toward the top 10. |
| Same company + role, `date` **older than 12 months** | **Eligible again.** Surface it, and mark `re-apply, last <YYYY-MM> <status>` in the Notes cell. |
| Same company + role, `date` missing or unparseable | **Exclude.** Fail toward not re-pitching a company too soon. |
| Same company, **different** role | Not excluded. The cooldown is per position, not per employer. |

Compute the cutoff as today minus 12 months and use the actual date — never a hardcoded one.

**Matching is fuzzy by necessity.** The tracker's 173 rows were hand-imported from a spreadsheet, so compare case-insensitively, trim whitespace, and ignore legal suffixes (`S.r.l.`, `S.p.A.`, `GmbH`, `Ltd`, `Inc`, `AG`). Match roles on meaning, not string equality — "Frontend Developer" and "Front-end Engineer" at the same company are the same position for this rule. When genuinely unsure whether two rows are the same position, **exclude** and note it.

`/rank`, `/berlin`, `/milan-berlin` and `/milan-berlin-wide` apply this identical rule. All five share one cooldown; keep them in sync if it ever changes.

Dedup against `seen_jobs.json` still applies on top of this, per Step 4.

## Execution

Follow `.claude/skills/job-scraper/SKILL.md` Steps 0-6 unchanged, with the overrides below. Do not reimplement the pipeline — dedup, health checks, referral links and tracker rules all still apply exactly as written there.

### Override 1 — how to search Ring 1

**Never search Ring 1 cities by name.** `search-queries.md` records this as a dead lane, verified 2026-08-16: Bergamo, Como, Monza, Novara and Piacenza returned zero locally-posted roles, and LinkedIn silently falls back to Milan results when a location has no matches, so a full-looking result set is an artefact rather than a finding. Re-running those queries burns calls and manufactures false positives.

Reach Ring 1 two other ways instead:

1. **Wide Milan location terms.** `Milan`, `Milano`, `Milan metropolitan area`, `Lombardy`, `Lombardia`. Belt employers post under these, which is exactly why the city-name searches came back empty.
2. **Named-employer careers pages, direct.** Check these by hand each run — they are the reason Ring 1 exists as a tier:

   | Employer | Site | Lane |
   |---|---|---|
   | D-Orbit | Fino Mornasco (Como) | space, ESA contracts — strongest single match on the Galileo background |
   | Alstom | Sesto San Giovanni | rail — direct ADIF/Renfe adjacency |
   | ST Microelectronics | Agrate Brianza | semiconductors |
   | Leonardo Helicopters | Cascina Costa, Vergiate, Sesto Calende (Varese) | aerospace/defence |
   | Brembo | Stezzano (Bergamo) | automotive, active software/digital push |
   | Scame | Bergamo province | EV charging |
   | Daze | Vimercate | EV charging |

   Treat every row as a **lead to verify, not a fact**. Confirm the site, the opening and any company claim independently before it reaches a CV or cover letter, per the CLAUDE.md verification checklist.

**Leonardo and defence:** Italian defence clearance is normally gated on Italian citizenship. Cosme is a Spanish national, so run the Eligibility Gate carefully on Leonardo and similar postings — EU citizenship is not automatically sufficient. Quote the posting's own wording rather than assuming either way.

### Override 2 — query emphasis

Run Priority 1-4 from `search-queries.md` with Milan terms. Two Italy-specific weightings, both evidence-backed in that file:

- **Query full-stack titles at least as hard as frontend.** TeamSystem posted `Full-stack Engineer — Node.js & React` at 53-66K € and `Frontend Developer` at 33-37K € in the same week, same cities, same English bar — a ~20K gap tracking title and scope, not seniority. A pure "Frontend Developer" title in Italy is structurally underpaid. Cosme's Node, Java/Spring and Docker/Kubernetes/Helm/Jenkins work genuinely supports the full-stack framing, so this is reframing real experience, not a stretch. Include `full stack`, `fullstack`, `software engineer`, `product engineer`.
- **Weight toward product companies and international employers.** Milan's band is bimodal by *employer type*, not role title: Italian consultancies and staffing agencies (Fincons, TXT, Teoresi, Akkodis, aizoOn, ADENTIS, KeyBiz, Softlab, Hays, ACTION ICT) cluster at 30-45K and top out at or below the floor, while product/international employers clear it comfortably. The consultancy lane is high-volume, low-yield, and also where Italian-language requirements concentrate.

Prioritise **Priority 4 (space, aerospace, transport, geospatial)** results even when the raw keyword match looks weaker. That is the one lane where this profile is differentiated rather than competing against every other mid-level React CV.

### Override 3 — salary

| Floor (the anchor to ask for) | Target | Surface anyway | Do not surface |
|---|---|---|---|
| 45K € | 45-50K €, ideally 48-50K €+ | 41-45K €, labelled below floor | below ~40K € — current comp is 37.5K €, so it is not a move |

45K is **mid-band for Milan, not the ceiling.** Never quote a number below 45K first, even when the posting's stated maximum is lower — asking costs nothing.

### Override 4 — language gate

Apply `04-job-evaluation.md`'s Language Gate strictly. Italian is **not** on Cosme's Languages table, so any posting requiring Italian as a job condition is a hard FAIL — excluded before scoring, quoted back to the user. A posting merely *written* in Italian for a role that operates in English is fine. Search in English only; Italian-language queries surface results that are already disqualified.

### Override 5 — tax regime

If a Milan posting names a number or reaches offer discussion, surface the *regime impatriati* as a **two-sided** factor per `10-italy-tax-regime.md`: it can move a Milan net much closer to a higher Berlin gross, and it carries a 4-year Italian tax-residence lock-in that makes a bad Milan situation expensive to exit. It is self-declared and unverified by an accountant. Never treat it as settled in the candidate's favour, never quote a net-with-regime figure to an employer, and never let it justify accepting a low gross.

## Search dense, present sparse

**Search wide.** Run every priority category in `search-queries.md` against every enabled portal — the `/scrape broad` behaviour, not the default top-3. Density is the point: cast for volume so the shortlist is drawn from a real pool.

**Present narrow.** Show only the **top 10** by fit. Do not pad, do not present a long table.

Everything fetched still goes into `seen_jobs.json` per Step 4 — the results that did not make the top 10 are stored with `status: "skipped"` so they do not resurface every run. Report the discard count in the header so the cut stays visible and the user can ask to see more.

## Output

**One ranked table, best row first.** Exactly these six columns, in this order, every run. **URL is always last.**

| Company | Position | Salary | Location | Fit | URL |
|---|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`45-55K €`, `53-66K € + benefits`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Most postings name no number; that is normal, but per `04-job-evaluation.md` dimension 6 it is no longer free — `not stated` scores 40.
- **Location** — `Milan`, or the belt city with its ring for Ring 1 results: `Como (Ring 1)`, `Bergamo (Ring 1)`. This replaces the old `— Ring 1 (<city>)` note suffix.
- **Fit** — how well the posting matches *this* profile, on the `04-job-evaluation.md` bands: `Strong` (75+), `Good` (60-74), `Moderate` (45-59), `Weak` (30-44), `Poor` (<30). See the shared rules below.

### Keep every cell short — the terminal reflows wide tables

A markdown table wider than the user's terminal is re-rendered as `Key: value` blocks, one stacked paragraph per row. It stops being a table.

| Rule | Detail |
|---|---|
| **Hard cap: 40 characters per cell** | Shorten titles (`Senior Frontend Developer` → `Senior Frontend`), drop legal suffixes (`S.r.l.`, `S.p.A.`). |
| **No quoted evidence in the ranked table** | Italian posting quotes belong in the Gate failures table. Long quotes are what blows the width. |
| **URL cell is the literal word `link`** | `[link](url)`, never the full URL as visible text. |
| **Split before you widen** | A seventh field means a second narrow table keyed on Company — never an extra column. |

### Ordering — fit, salary and location together

Per `04-job-evaluation.md`'s **Shortlist ordering** section, which is authoritative. Row 1 is the posting to apply to first, not the highest raw score.

| # | Key | Detail |
|---|---|---|
| 1 | **Actionability** | Clears the 45K Milan floor > not stated > below floor. |
| 2 | **Weighted score** | The six dimensions, Compensation included at 20%. |
| 3 | **Location preference** | Milan proper > Ring 1, as a tiebreak only. Ring 1 scores as Milan on money and priority; this key just breaks near-equal rows toward the city itself. |
| 4 | **Language gate clean** | PASS outranks ⚠ FLAG at equal standing. Italian-language postings with no stated requirement are FLAG, and there are many. |

Number rows `1.`, `2.`, `3.` … in the Company cell. **Length is whatever the run justifies** — the old "top 10" was a cap, not a target. Never pad to a round number, never truncate a qualifying posting to stay under one.

### The Fit column

| Rule | Meaning |
|---|---|
| **Same bands as `/rank`** | Score on `04-job-evaluation.md`'s six weighted dimensions — Technical 24%, Experience 20%, Behavioral 12%, Career Alignment 24%, **Compensation 20%** — and map to its verdict bands. A search run and a `/rank` run must never disagree about the same posting. |
| **Word, not number** | Write `Strong`, `Good`, `Moderate`, `Weak`, `Poor`. The underlying number is a triage estimate from posting text alone; publishing it implies a precision this pass does not have. `/rank` is where numbers belong. |
| **Fetched text only** | A posting whose detail was never retrieved gets `unscored`. Never infer fit from a job title. |
| **Gates override the band** | Location or language gate FAIL → `Poor` regardless of stack match, with the gate quoted in the Gate failures table. FLAG → keep the band, append `⚠`. |
| **Fit includes pay** | Added 2026-09-04. Compensation is dimension 6 at 20%, so a below-floor or unpriced posting genuinely scores lower. It is no longer a Notes-only annotation. |
| **Sorts the table** | One table, ordered by the four keys in the Ordering section above. The reader should be able to stop after row 3. |

Header line:

```
## New Milan Matches - YYYY-MM-DD
K of N new positions, best first. Scope: Milan + Ring 1.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown).
(N-K lower-fit results stored, not shown.)
```

Keep the `skipped (disabled):` and `health:` lines from Step 5 when they apply. High-match highlights and the Step 4.5 contact links follow the table for the top results only.

If the run genuinely yields little, say so plainly and suggest `/milan-berlin` or `/scrape` — never pad the table with out-of-scope results to reach ten rows.

### Everything is a table — no prose blocks

The position table is not the only table. **Every section of the output is a table**, the analysis included. Prose is reserved for a single closing caveat where tabulating would misrepresent the finding — never for findings themselves, never for comparison, never for reasoning about a specific posting.

Emit these after the position table, in this order, and only when they have rows:

| Section | Columns | Emit when |
|---|---|---|
| **Gate failures and drops** | `Company \| Position \| Dropped on \| Evidence (quoted)` | Any posting failed a screen. `Dropped on` is one of: `Language gate`, `Below floor`, `Stack gap`, `Discipline gap`, `Seniority gap`, `Mass posting`. |
| **Ring 1 findings** | `Company \| Position \| Belt city \| Outcome` | Any result landed outside Milan proper. Records that the belt is reachable through wide Milan terms even when every belt result fails on merit — that is the Override 1 evidence trail. |
| **Trade-off** | `Factor \| <option A> \| <option B>` | Two or more results are genuinely competitive. One row per factor: top pay stated, best stack fit, language risk, salary transparency, seniority gap, employer type. |
| **Framework corrections** | `File \| Current entry \| Evidence this run \| Change needed` | The run contradicted `04-job-evaluation.md` or `search-queries.md`. Do **not** edit those files from inside a search run — table the correction and let the user approve it. |
| **Already surfaced, still unapplied** | `Company \| Position \| First seen \| Status \| Why still relevant` | `seen_jobs.json` dedup suppressed a posting still marked `new` that is still a strong match. |
| **Run stats** | `Metric \| Count` | Always. Raw results, unique URLs, cooldown drops, seen_jobs drops, intra-run dupes, kept, detail-fetched, shortlisted, stored as skipped. |

Rules holding across all of them:

| Rule | Meaning |
|---|---|
| **Quote, do not summarise** | Any cell asserting what a posting requires carries the posting's own words in quotes. A paraphrased gate failure is not a gate failure. |
| **One fact per row** | Two gates failed at one company is two rows, not one cell holding both. |
| **No width cap** | The 40-character cell cap is the ranked position table only. These tables carry the detail that cap forced out — quoted Italian posting text belongs here. Still keep them under terminal width; split into two keyed tables rather than running one wide. |
| **Empty means omitted** | A section with no rows disappears. Never emit an empty table or a "none found" placeholder row. |
