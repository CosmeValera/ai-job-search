---
name: milan-berlin
description: >
  Runs a job search across both target cities at once — Milan plus the Ring 1 commuter
  belt, and Berlin — and presents them side by side with the city trade-off surfaced.
  Excludes Ring 2 and Tier 2. Triggers on: both cities, milan and berlin, /milan-berlin
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(bun --version), Bash(bun run .agents/skills/*/cli/src/cli.ts *), WebFetch, WebSearch, Agent, AskUserQuestion
---

# Milan + Berlin Job Search

Both target cities in one run. This is the default sweep — `/milan` and `/berlin` exist for when only one city is wanted.

## Scope

**In scope**
- Milan / Milano and **Ring 1** (Monza/Brianza, Bergamo, Como, Lecco, Varese, Novara, Pavia)
- Berlin
- Fully remote EU

**Out of scope for this run** — drop silently:
- Ring 2 (Turin, Genoa, Bologna, Verona, Trento, Bolzano) — use `/scrape` for the wider net
- Tier 2 (Dublin, Amsterdam, Munich, Luxembourg, Zurich, Geneva). **Munich is Tier 2, not a German substitute for Berlin.**
- Spain — hard exclude, always

## Re-application cooldown — apply and restate every single run

**Never surface a position already applied to within the last 12 months.** This is not optional and not a judgement call. State the rule and the cutoff date in the output header on every run, even when nothing was filtered by it. Apply it to **both** city passes.

Build the exclusion set from `job_search_tracker.csv` (columns: `date,company,sector,role,role_type,channel,status,contact_person,fit_rating,notes,cv_file,cover_letter_file,source`):

| Tracker row | Handling |
|---|---|
| Same company + role, `date` **within the last 12 months** | **Exclude.** Do not present, do not count toward the shortlist. |
| Same company + role, `date` **older than 12 months** | **Eligible again.** Surface it, and mark `re-apply, last <YYYY-MM> <status>` in the Notes cell. |
| Same company + role, `date` missing or unparseable | **Exclude.** Fail toward not re-pitching a company too soon. |
| Same company, **different** role | Not excluded. The cooldown is per position, not per employer. |

Compute the cutoff as today minus 12 months and use the actual date — never a hardcoded one.

**Matching is fuzzy by necessity.** The tracker's 173 rows were hand-imported from a spreadsheet, so compare case-insensitively, trim whitespace, and ignore legal suffixes (`S.r.l.`, `S.p.A.`, `GmbH`, `SE`, `AG`, `Ltd`, `Inc`). Match roles on meaning, not string equality — "Frontend Developer" and "Front-end Engineer" at the same company are the same position for this rule. When genuinely unsure whether two rows are the same position, **exclude** and note it.

Watch for the same employer appearing in both passes with different roles — that is allowed, and it is not a mass-posting signal unless the descriptions match per Step 2.5.

`/rank`, `/milan`, `/berlin` and `/milan-berlin-wide` apply this identical rule. All five share one cooldown; keep them in sync if it ever changes.

Dedup against `seen_jobs.json` still applies on top of this, per Step 4.

## Execution

Follow `.claude/skills/job-scraper/SKILL.md` Steps 0-6 unchanged. Run the two city scopes as **parallel search passes**, then merge into one dedup/store/present cycle — do not run the whole pipeline twice, or the same remote-EU posting gets written to `seen_jobs.json` under two keys.

**Per-city rules are not duplicated here.** Apply them from the scoped skills:
- Milan and Ring 1: all five overrides in `.claude/skills/milan/SKILL.md` — especially **Override 1**, which forbids searching Ring 1 cities by name (verified dead lane; LinkedIn falls back to Milan results and manufactures false positives) and routes the belt through wide Milan location terms plus named-employer careers pages.
- Berlin: all four overrides in `.claude/skills/berlin/SKILL.md`.

### Salary floors differ by city — do not average them

| City | Floor | Target | Stretch-down |
|---|---|---|---|
| Milan + Ring 1 | 45K € | 45-50K €, ideally 48-50K €+ | 41-45K €, labelled below floor |
| Berlin | 50K € | 50K € is the *exclusion threshold*; the funded-startup mode runs 80-140K € | none |

Applying one blended floor across both cities is wrong in both directions — it drops viable Milan roles and flatters weak Berlin ones.

### Cross-city comparison — the reason to run this over two separate searches

Both language gates are live at once: Italian **and** German are undeclared, so a job-condition requirement for either is a hard FAIL on its side of the run.

When the run surfaces strong results in both cities, add the comparison block below the tables. Per `04-job-evaluation.md`:

- **Milan outranks Berlin, but not at any price.** Same band → Milan. Milan moderately below → still Milan. Milan far below → Berlin.
- **There is no fixed flip threshold and none should be invented.** The candidate's "40K Milan vs 60K Berlin" was an illustration of the principle, not a rule; asked about a narrower 40K/55K gap their own answer was "case by case". Present the comparison, let them decide.
- **Compare net, not gross, before calling any gap large.** Italian contracts commonly pay 13 or 14 monthly instalments, and the *regime impatriati* — self-declared, unverified by an accountant, carrying a 4-year Italian tax-residence lock-in — can move a Milan net much closer to a higher Berlin gross. Two-sided: it improves a good Milan offer and makes a bad Milan situation expensive to exit. See `10-italy-tax-regime.md`.
- **Remote and work-from-abroad are first-class factors, not perks.** The Milan preference is about time with one specific person there, so flexibility partly substitutes for the city. A well-paid Berlin role with generous work-from-abroad days can beat a poorly-paid Milan role *on the Milan criterion itself*. Say so when it applies.

Surface the trade-off as evidence. Never present it as having decided anything.

## Search dense, present sparse

**Search wide.** Run every priority category in `search-queries.md` against every enabled portal, in both city passes — the `/scrape broad` behaviour, not the default top-3. Density is the point: cast for volume so the shortlist is drawn from a real pool.

**Present narrow.** Show only the **top 10 across both cities combined**, ranked by fit. Allocation follows quality, not a fixed split — a strong Berlin week may show 7 Berlin and 3 Milan. **One guarantee:** if a city has at least 3 qualifying results, it gets at least 3 rows, so the side-by-side comparison never collapses to one city by a narrow ranking margin.

Everything fetched still goes into `seen_jobs.json` per Step 4 — results outside the top 10 are stored with `status: "skipped"` so they do not resurface every run. Report the discard count in the header so the cut stays visible and the user can ask to see more.

## Output

**One ranked table across both cities, best row first.** Not one table per city — that was the format until 2026-09-04 and it was wrong: splitting by city put weak Milan rows above strong Berlin ones and forced the reader to re-merge the lists in their head. City now lives in the Location column, and the Milan-first preference is applied as an ordering tiebreak, not as a grouping.

Exactly these six columns, in this order, every run. **URL is always last.**

| Company | Position | Salary | Location | Fit | URL |
|---|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`45-55K €`, `80-120K € base + VSOP`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Do not normalise Italian and German figures against each other here — the net-vs-gross comparison belongs in the City Comparison table, not in a table cell.
- **Location** — `Milan`, `Berlin`, `Como (Ring 1)`, `Berlin hybrid`, `remote EU`. Replaces the old `Ring 1 (<city>)` note suffix.
- **Fit** — how well the posting matches *this* profile, on the `04-job-evaluation.md` bands: `Strong` (75+), `Good` (60-74), `Moderate` (45-59), `Weak` (30-44), `Poor` (<30). See the shared rules below.

### Keep every cell short — the terminal reflows wide tables

A markdown table wider than the user's terminal is re-rendered as `Key: value` blocks, one stacked paragraph per row. It stops being a table.

| Rule | Detail |
|---|---|
| **Hard cap: 40 characters per cell** | Shorten titles (`Software Engineer - Frontend` → `Frontend Engineer`), drop legal suffixes (`GmbH`, `S.p.A.`). |
| **No quoted evidence in the ranked table** | Italian and German posting quotes belong in the Gate failures table. Long quotes are what blows the width. |
| **URL cell is the literal word `link`** | `[link](url)`, never the full URL as visible text. |
| **Split before you widen** | A seventh field means a second narrow table keyed on Company — never an extra column. |

### Ordering — fit, salary and location together

Per `04-job-evaluation.md`'s **Shortlist ordering** section, which is authoritative. Row 1 is the posting to apply to first, not the highest raw score.

| # | Key | Detail |
|---|---|---|
| 1 | **Actionability** | Clears its city floor (Milan 45K, Berlin 50K) > not stated > below floor. Berlin publishes salary far less often than Milan, so this key stops unpriced Berlin postings owning the top of every run. |
| 2 | **Weighted score** | The six dimensions, Compensation included at 20%. |
| 3 | **Location preference** | Milan / Ring 1 > Berlin, as a **tiebreak within a band only**. It reorders near-equal rows; it never lifts a Moderate Milan role above a Strong Berlin one. The full preference argument stays in the City Comparison table. |
| 4 | **Language gate clean** | PASS outranks ⚠ FLAG at equal standing. |

Number rows `1.`, `2.`, `3.` … in the Company cell. **Length is whatever the run justifies** — the old "top 10" was a cap, not a target. Never pad to a round number, never truncate a qualifying posting to stay under one. Keep the guarantee that each city gets at least 3 rows when it has 3 qualifying results.

### The Fit column

| Rule | Meaning |
|---|---|
| **Same bands as `/rank`** | Score on `04-job-evaluation.md`'s six weighted dimensions — Technical 24%, Experience 20%, Behavioral 12%, Career Alignment 24%, **Compensation 20%** — and map to its verdict bands. A search run and a `/rank` run must never disagree about the same posting. |
| **Word, not number** | Write `Strong`, `Good`, `Moderate`, `Weak`, `Poor`. The underlying number is a triage estimate from posting text alone; publishing it implies a precision this pass does not have. `/rank` is where numbers belong. |
| **Fetched text only** | A posting whose detail was never retrieved gets `unscored`. Never infer fit from a job title. |
| **Gates override the band** | An Italian requirement in Milan or a German one in Berlin is `Poor` regardless of stack match, with the untranslated quote in Notes. FLAG → keep the band, append `⚠`. |
| **One scale, both cities** | Fit measures match against the profile, nothing else. It never encodes the Milan-first preference — a Berlin role that fits better scores higher, and the city preference is applied afterwards as ordering key 3. Baking the preference into Fit would double-count it. |
| **Fit includes pay** | Added 2026-09-04. Compensation is dimension 6 at 20%, so a below-floor or unpriced posting genuinely scores lower. It is no longer a Notes-only annotation, and it applies against each city's own floor. |
| **Sorts the table** | One table, ordered by the four keys in the Ordering section above. No per-city sorting — city is a tiebreak, not a grouping. |

```
## New Matches - YYYY-MM-DD
K of N new positions, best first. Scope: Milan + Ring 1, Berlin.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown).
(N-K lower-fit results stored, not shown.)

| Company | Position | Salary | Location | Fit | URL |
|---|---|---|---|---|---|
| 1. ... | ... | ... | ... | Strong | [link](...) |

### City Comparison
(only when both cities produced strong results — the four bullets above, applied to
what this run actually found. Omit entirely when one city came back empty.)
```

Number the rows in a **single continuous sequence across both tables** — prefix the Company cell (`1. Satispay`) rather than adding a sixth column — so the follow-up prompt ("give me the number(s)") stays unambiguous.

Keep the `skipped (disabled):` and `health:` lines from Step 5 when they apply. High-match highlights and the Step 4.5 contact links follow the tables, for the shortlisted results only.

If the run yields 8+ new jobs, suggest `/rank` — it batch-scores everything against the full framework, which beats eyeballing two tables.

### Everything is a table — no prose blocks

The position tables are not the only tables. **Every section of the output is a table**, the analysis included. That covers the City Comparison block above: it is a table, not four bullets. Prose is reserved for a single closing caveat where tabulating would misrepresent the finding — never for findings themselves, never for comparison, never for reasoning about a specific posting.

Emit these after the position tables, in this order, and only when they have rows:

| Section | Columns | Emit when |
|---|---|---|
| **Gate failures and drops** | `Company \| Position \| City \| Dropped on \| Evidence (quoted)` | Any posting failed a screen. `Dropped on` is one of: `Language gate`, `Below floor`, `Stack gap`, `Discipline gap`, `Seniority gap`, `Mass posting`. Quote Italian and German text untranslated so the bar is auditable. |
| **Ring 1 findings** | `Company \| Position \| Belt city \| Outcome` | Any Milan-pass result landed outside Milan proper. Records that the belt is reachable through wide Milan terms even when every belt result fails on merit — the Override 1 evidence trail. |
| **City Comparison** | `Factor \| Milan + Ring 1 \| Berlin` | Both cities produced strong results. One row per factor: top pay stated, best stack fit, language risk, salary transparency, relocation support, remote / work-from-abroad. Omit entirely when one city came back empty. |
| **Framework corrections** | `File \| Current entry \| Evidence this run \| Change needed` | The run contradicted `04-job-evaluation.md` or `search-queries.md`. Do **not** edit those files from inside a search run — table the correction and let the user approve it. |
| **Already surfaced, still unapplied** | `Company \| Position \| First seen \| Status \| Why still relevant` | `seen_jobs.json` dedup suppressed a posting still marked `new` that is still a strong match. |
| **Run stats** | `Metric \| Count` | Always. Raw results, unique URLs, cooldown drops, seen_jobs drops, intra-run dupes, kept, detail-fetched, shortlisted, stored as skipped. |

Rules holding across all of them:

| Rule | Meaning |
|---|---|
| **Quote, do not summarise** | Any cell asserting what a posting requires carries the posting's own words in quotes. A paraphrased gate failure is not a gate failure. |
| **One fact per row** | Two gates failed at one company is two rows, not one cell holding both. |
| **No width cap** | The 40-character cell cap is the ranked position table only. These tables carry the detail that cap forced out — quoted Italian and German posting text belongs here. Still keep them under terminal width; split into two keyed tables rather than running one wide. |
| **Empty means omitted** | A section with no rows disappears. Never emit an empty table or a "none found" placeholder row. |
| **Never rank in a comparison table** | The City Comparison table states each city's position on a factor. It does not declare a winner — the Milan-first preference rules above do that, in the one line of prose this output allows. |
