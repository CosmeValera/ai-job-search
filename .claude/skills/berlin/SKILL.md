---
name: berlin
description: >
  Runs a job search scoped to Berlin only. Excludes Milan, the Ring 1 commuter belt,
  Ring 2 and Tier 2 cities. Triggers on: berlin jobs, jobs in berlin, search berlin,
  /berlin
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(bun --version), Bash(bun run .agents/skills/*/cli/src/cli.ts *), WebFetch, WebSearch, Agent, AskUserQuestion
---

# Berlin-Only Job Search

Scoped variant of `/scrape`. Same pipeline, one geography.

## Scope

**In scope**
- Berlin (primary)
- Fully remote within the EU where the company is Berlin-headquartered

**Out of scope for this run** — drop silently, do not present, do not write to `seen_jobs.json` as new:
- Milan, Ring 1 and Ring 2 — everything Italian
- Other German cities. **Munich is Tier 2, not Berlin** — it belongs to `/scrape`, not here
- Tier 2 generally (Dublin, Amsterdam, Munich, Luxembourg, Zurich, Geneva)
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

**Matching is fuzzy by necessity.** The tracker's 173 rows were hand-imported from a spreadsheet, so compare case-insensitively, trim whitespace, and ignore legal suffixes (`GmbH`, `SE`, `AG`, `Ltd`, `Inc`, `S.r.l.`). Match roles on meaning, not string equality — "Frontend Developer" and "Front-end Engineer" at the same company are the same position for this rule. When genuinely unsure whether two rows are the same position, **exclude** and note it.

`/rank`, `/milan`, `/milan-berlin` and `/milan-berlin-wide` apply this identical rule. All five share one cooldown; keep them in sync if it ever changes.

Dedup against `seen_jobs.json` still applies on top of this, per Step 4.

## Execution

Follow `.claude/skills/job-scraper/SKILL.md` Steps 0-6 unchanged, with the overrides below. Do not reimplement the pipeline — dedup, health checks, referral links and tracker rules all still apply exactly as written there.

### Override 1 — query emphasis

Run Priority 1-5 from `search-queries.md` with Berlin terms. Berlin's tech market is large and English-speaking, so the generic frontend/TypeScript categories work here in a way they do not in Milan.

Berlin-specific sources worth hitting beyond the standard portal CLIs:
- **berlinstartupjobs.com** — Berlin-only, high signal
- **Ashby, Greenhouse, Lever, Personio, Workable, SmartRecruiters** — where most Berlin startup postings actually live; historically the highest-volume sources in the tracker
- **welcometothejungle.com**, **Otta / Cord**

Prioritise **Priority 4 (space, aerospace, transport, geospatial)** even when raw keyword match looks weaker — it is the one lane where this profile is differentiated. Berlin has real depth here: OHB, ConstellR, LiveEO (already applied), Isar Aerospace, Planetary Transportation Systems. Verify each independently before any claim reaches a document.

### Override 2 — salary

| Exclusion threshold | Realistic upper mode |
|---|---|
| 50K € | 80-140K € at funded product startups |

**Treat 50K as the exclusion threshold, not the target.** Two Berlin postings with published numbers, both well-funded product startups: Almedia Frontend at 80-140K € (declined — 7+ years required), Buena Senior Software Engineer at 80-120K € base plus virtual stock options, **with no years requirement stated at all**.

Berlin is bimodal like Milan, but the split is funded-product-startup vs everything else, and the upper mode runs far higher than Milan's 53-66K ceiling. Surfacing a 50-60K Berlin role as a good outcome understates the market. **Weight the search hard toward funded product startups** and check the years requirement rather than self-selecting out — Buena stated none.

No stretch-down band for Berlin. Below 50K, flag rather than drop.

### Override 3 — language gate

Apply `04-job-evaluation.md`'s Language Gate strictly. German is **not** on Cosme's Languages table, so any posting requiring German as a job condition is a hard FAIL — excluded before scoring, quoted back to the user. A posting merely *written* in German for a role that operates in English is fine. Search in English only.

English requirements at "fluent" or "professional" level pass cleanly against a Cambridge C1 plus a full-time English placement in Ireland.

### Override 4 — remote and work-from-abroad are first-class

Milan ranks above Berlin because of one personal tie in that city, so **anything buying physical flexibility partly substitutes for Milan itself.** For every Berlin result, capture and surface:
- Remote or hybrid days per week, and whether that is written policy or manager's discretion
- Work-from-abroad allowances — these vary enormously and hide in the benefits list (Doctolib 10 days/yr, Flix 60 days, SumUp sabbatical)
- Fully remote EU, which sidesteps the city choice entirely

A well-paid Berlin role with generous work-from-abroad days can beat a poorly-paid Milan role **on the Milan criterion itself**. Say so explicitly when it applies.

## Search dense, present sparse

**Search wide.** Run every priority category in `search-queries.md` against every enabled portal plus the Berlin-specific sources above — the `/scrape broad` behaviour, not the default top-3. Density is the point: cast for volume so the shortlist is drawn from a real pool.

**Present narrow.** Show only the **top 10** by fit. Do not pad, do not present a long table.

Everything fetched still goes into `seen_jobs.json` per Step 4 — the results that did not make the top 10 are stored with `status: "skipped"` so they do not resurface every run. Report the discard count in the header so the cut stays visible and the user can ask to see more.

## Output

**One ranked table, best row first.** Exactly these six columns, in this order, every run. **URL is always last.**

| Company | Position | Salary | Location | Fit | URL |
|---|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`80-120K € base + VSOP`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Most postings name no number; that is normal, but per `04-job-evaluation.md` dimension 6 it is no longer free — `not stated` scores 40.
- **Location** — `Berlin`, `Berlin hybrid`, `remote DE`, `remote EU`. The Override 4 flexibility finding gets its own **Flexibility** table below; do not try to cram days-per-week into this cell.
- **Fit** — how well the posting matches *this* profile, on the `04-job-evaluation.md` bands: `Strong` (75+), `Good` (60-74), `Moderate` (45-59), `Weak` (30-44), `Poor` (<30). See the shared rules below.

### Keep every cell short — the terminal reflows wide tables

A markdown table wider than the user's terminal is re-rendered as `Key: value` blocks, one stacked paragraph per row. It stops being a table.

| Rule | Detail |
|---|---|
| **Hard cap: 40 characters per cell** | Shorten titles (`Software Engineer - Frontend` → `Frontend Engineer`), drop legal suffixes (`GmbH`, `SE`). |
| **No quoted evidence in the ranked table** | Posting quotes belong in the Gate failures table. Long quotes are what blows the width. |
| **URL cell is the literal word `link`** | `[link](url)`, never the full URL as visible text. |
| **Split before you widen** | A seventh field means a second narrow table keyed on Company — never an extra column. |

### Ordering — fit, salary and location together

Per `04-job-evaluation.md`'s **Shortlist ordering** section, which is authoritative. Row 1 is the posting to apply to first, not the highest raw score.

| # | Key | Detail |
|---|---|---|
| 1 | **Actionability** | Clears the 50K Berlin floor > not stated > below floor. Berlin publishes salary rarely, so this key does real work here: it stops four unpriced postings owning the top of every run. |
| 2 | **Weighted score** | The six dimensions, Compensation included at 20%. |
| 3 | **Location preference** | Berlin office > hybrid > remote DE > remote EU, as a tiebreak only. Note this runs opposite to Override 4's logic — flexibility substitutes for *Milan*, not for Berlin, so in a Berlin-only run the physical role is the more valuable one. |
| 4 | **Language gate clean** | PASS outranks ⚠ FLAG at equal standing. |

Number rows `1.`, `2.`, `3.` … in the Company cell. **Length is whatever the run justifies** — the old "top 10" was a cap, not a target. Never pad to a round number, never truncate a qualifying posting to stay under one.

### The Fit column

| Rule | Meaning |
|---|---|
| **Same bands as `/rank`** | Score on `04-job-evaluation.md`'s six weighted dimensions — Technical 24%, Experience 20%, Behavioral 12%, Career Alignment 24%, **Compensation 20%** — and map to its verdict bands. A search run and a `/rank` run must never disagree about the same posting. |
| **Word, not number** | Write `Strong`, `Good`, `Moderate`, `Weak`, `Poor`. The underlying number is a triage estimate from posting text alone; publishing it implies a precision this pass does not have. `/rank` is where numbers belong. |
| **Fetched text only** | A posting whose detail was never retrieved gets `unscored`. Never infer fit from a job title. |
| **Gates override the band** | The German gate is the common case in Berlin: a job-condition German requirement is `Poor` regardless of stack match, with the German quoted in Notes. FLAG → keep the band, append `⚠`. |
| **Seniority is scored, not vetoed** | A Senior or Staff title with a years bar above the profile's ~4 lowers the Experience dimension — it does not zero the row. Buena stated no years requirement at 80-120K; self-selecting out on title alone is how that lane gets missed. |
| **Fit includes pay** | Added 2026-09-04. Compensation is dimension 6 at 20%, so an unpriced Berlin posting genuinely scores lower than an equivalent one publishing 80-120K. It is no longer a Notes-only annotation. |
| **Sorts the table** | One table, ordered by the four keys in the Ordering section above. The reader should be able to stop after row 3. |

Header line:

```
## New Berlin Matches - YYYY-MM-DD
K of N new positions, best first. Scope: Berlin.
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
| **Gate failures and drops** | `Company \| Position \| Dropped on \| Evidence (quoted)` | Any posting failed a screen. `Dropped on` is one of: `Language gate`, `Below floor`, `Stack gap`, `Discipline gap`, `Seniority gap`, `Mass posting`. The German gate is the highest-volume row source in Berlin — quote the German text itself, untranslated, so the bar is auditable. |
| **Flexibility** | `Company \| Position \| Office days \| Work-from-abroad \| Relocation support` | Two or more results stated remote or work-from-abroad terms. Per Override 4 this is a decision input, not a perk, so it gets its own table once the Notes cell runs out of room. |
| **Trade-off** | `Factor \| <option A> \| <option B>` | Two or more results are genuinely competitive. One row per factor: top pay stated, best stack fit, language risk, salary transparency, seniority gap, employer type. |
| **Framework corrections** | `File \| Current entry \| Evidence this run \| Change needed` | The run contradicted `04-job-evaluation.md` or `search-queries.md`. Do **not** edit those files from inside a search run — table the correction and let the user approve it. |
| **Already surfaced, still unapplied** | `Company \| Position \| First seen \| Status \| Why still relevant` | `seen_jobs.json` dedup suppressed a posting still marked `new` that is still a strong match. |
| **Run stats** | `Metric \| Count` | Always. Raw results, unique URLs, cooldown drops, seen_jobs drops, intra-run dupes, kept, detail-fetched, shortlisted, stored as skipped. |

Rules holding across all of them:

| Rule | Meaning |
|---|---|
| **Quote, do not summarise** | Any cell asserting what a posting requires carries the posting's own words in quotes. A paraphrased gate failure is not a gate failure. |
| **One fact per row** | Two gates failed at one company is two rows, not one cell holding both. |
| **No width cap** | The 40-character cell cap is the ranked position table only. These tables carry the detail that cap forced out — quoted posting text belongs here. Still keep them under terminal width; split into two keyed tables rather than running one wide. |
| **Empty means omitted** | A section with no rows disappears. Never emit an empty table or a "none found" placeholder row. |
