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

Exactly these five columns, in this order, every run:

| Company | Position | Salary | URL | Notes |
|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`45-55K €`, `53-66K € + benefits`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Most postings name no number; that is normal, not a gap to fill.
- **Notes** — **20 words maximum**, hard cap. Spend them on the single highest-value signal, in this priority order: hard gate failure → below-floor salary → mass-posting spread → the one concrete reason this is a strong match. Not a summary of the posting.

Append `— Ring 1 (<city>)` inside the Notes cell for belt results, within the 20-word budget. It does not get its own column.

Header line:

```
## New Milan Matches - YYYY-MM-DD
Top 10 of N new positions. Scope: Milan + Ring 1.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown).
(N-10 lower-fit results stored, not shown.)
```

Keep the `skipped (disabled):` and `health:` lines from Step 5 when they apply. High-match highlights and the Step 4.5 contact links follow the table for the top results only.

If the run genuinely yields little, say so plainly and suggest `/milan-berlin` or `/scrape` — never pad the table with out-of-scope results to reach ten rows.
