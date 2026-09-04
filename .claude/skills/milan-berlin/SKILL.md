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

Two tables, Milan first (it is first choice), then Berlin. Both use **exactly these five columns, in this order**, every run:

| Company | Position | Salary | URL | Notes |
|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`45-55K €`, `80-120K € base + VSOP`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Most postings name no number; that is normal, not a gap to fill. Do not normalise Italian and German figures against each other here — the net-vs-gross comparison belongs in the City Comparison block, not in a table cell.
- **Notes** — **20 words maximum**, hard cap. Spend them on the single highest-value signal, in this priority order: hard gate failure → `re-apply, last <YYYY-MM>` → below-floor salary → city-specific signal (`Ring 1 (<city>)` for Milan, remote / work-from-abroad terms for Berlin) → the one concrete reason this is a strong match. Not a summary of the posting.

Ring and remote findings live inside the Notes budget. Neither gets its own column — the five-column schema is fixed across all three city skills.

```
## New Matches - YYYY-MM-DD
Top 10 of N new positions. Scope: Milan + Ring 1, Berlin.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown).
(N-10 lower-fit results stored, not shown.)

### Milan + Ring 1 (N found)
| Company | Position | Salary | URL | Notes |

### Berlin (N found)
| Company | Position | Salary | URL | Notes |

### City Comparison
(only when both cities produced strong results — the four bullets above, applied to
what this run actually found. Omit entirely when one city came back empty.)
```

Number the rows in a **single continuous sequence across both tables** — prefix the Company cell (`1. Satispay`) rather than adding a sixth column — so the follow-up prompt ("give me the number(s)") stays unambiguous.

Keep the `skipped (disabled):` and `health:` lines from Step 5 when they apply. High-match highlights and the Step 4.5 contact links follow the tables, for the shortlisted results only.

If the run yields 8+ new jobs, suggest `/rank` — it batch-scores everything against the full framework, which beats eyeballing two tables.
