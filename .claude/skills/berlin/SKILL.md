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

Exactly these five columns, in this order, every run:

| Company | Position | Salary | URL | Notes |
|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`80-120K € base + VSOP`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Most postings name no number; that is normal, not a gap to fill.
- **Notes** — **20 words maximum**, hard cap. Spend them on the single highest-value signal, in this priority order: hard gate failure → below-floor salary → **remote / work-from-abroad terms** → the one concrete reason this is a strong match. Not a summary of the posting.

The flexibility finding from Override 4 lives in the Notes cell (`3 days remote, 60 days work-from-abroad`) — it is a decision input here, not a perk, so it outranks generic match commentary for the word budget. It does not get its own column.

Header line:

```
## New Berlin Matches - YYYY-MM-DD
Top 10 of N new positions. Scope: Berlin.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown).
(N-10 lower-fit results stored, not shown.)
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
| **No word cap** | The 20-word Notes cap is the position table only. These tables exist to carry the detail that cap forced out. |
| **Empty means omitted** | A section with no rows disappears. Never emit an empty table or a "none found" placeholder row. |
