---
name: milan-berlin-wide
description: >
  Widest job sweep. Milan plus the Ring 1 commuter belt, Berlin, plus Ring 2 northern
  Italy (Turin, Genoa, Bologna, Verona, Trento, Bolzano) and Tier 2 Europe (Dublin,
  Amsterdam, Munich, Luxembourg, Zurich, Geneva) and remote EU. Triggers on: wide
  search, everything, all cities, widen the net, /milan-berlin-wide
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(bun --version), Bash(bun run .agents/skills/*/cli/src/cli.ts *), WebFetch, WebSearch, Agent, AskUserQuestion
---

# Milan + Berlin + Wider Net

The widest scoped sweep. Use when the two-city run comes back thin, or for a periodic check on markets that are acceptable but not preferred. `/milan-berlin` remains the default; this one trades precision for reach.

## Scope

**In scope, in preference order**

| Tier | Locations | Why it is here |
|---|---|---|
| **Ideal** | Milan / Milano + **Ring 1** (Monza/Brianza, Bergamo, Como, Lecco, Varese, Novara, Pavia) | First choice. Personal tie in the city. |
| **Ideal** | Berlin | First choice. Large English-speaking market, high upper mode. |
| **Ring 2** | Turin, Genoa, Bologna, Verona, Trento, Bolzano/Bozen | Separate Italian labour markets. Conditional — see floors. |
| **Tier 2** | Dublin, Amsterdam, Munich, Luxembourg, Zurich, Geneva | Included because they pay well, **not** because they are preferred. |
| **Remote** | Fully remote within the EU | Sidesteps the city choice entirely. |

**Out of scope — hard exclude, always**
- Madrid and anywhere in Spain
- Anywhere outside the EU/EEA/Switzerland. UK, US and Singapore rows in the historical tracker predate this focus and needed sponsorship.

Cosme is an EU citizen, so no sponsorship is required anywhere in scope. Switzerland needs a permit but it is near-automatic for EU nationals under free movement — never treat a "no sponsorship available" line as a blocker for any location in this list.

## Re-application cooldown — apply and restate every single run

**Never surface a position already applied to within the last 12 months.** This is not optional and not a judgement call. State the rule and the cutoff date in the output header on every run, even when nothing was filtered by it. Apply it to every tier.

Build the exclusion set from `job_search_tracker.csv` (columns: `date,company,sector,role,role_type,channel,status,contact_person,fit_rating,notes,cv_file,cover_letter_file,source`):

| Tracker row | Handling |
|---|---|
| Same company + role, `date` **within the last 12 months** | **Exclude.** Do not present, do not count toward the shortlist. |
| Same company + role, `date` **older than 12 months** | **Eligible again.** Surface it, and mark `re-apply, last <YYYY-MM> <status>` in the Notes cell. |
| Same company + role, `date` missing or unparseable | **Exclude.** Fail toward not re-pitching a company too soon. |
| Same company, **different** role | Not excluded. The cooldown is per position, not per employer. |

Compute the cutoff as today minus 12 months and use the actual date — never a hardcoded one.

**Matching is fuzzy by necessity.** The tracker's 173 rows were hand-imported from a spreadsheet, so compare case-insensitively, trim whitespace, and ignore legal suffixes (`S.r.l.`, `S.p.A.`, `GmbH`, `SE`, `AG`, `B.V.`, `Ltd`, `Inc`). Match roles on meaning, not string equality — "Frontend Developer" and "Front-end Engineer" at the same company are the same position for this rule. When genuinely unsure whether two rows are the same position, **exclude** and note it.

This sweep reaches markets the historical tracker covered heavily (Dublin, Amsterdam, Munich, Luxembourg, Zurich), so **expect the cooldown to bite hardest here.** Report the filtered count rather than letting it vanish silently.

`/rank`, `/milan`, `/berlin` and `/milan-berlin` apply this identical rule. All five share one cooldown; keep them in sync if it ever changes.

Dedup against `seen_jobs.json` still applies on top of this, per Step 4.

## Execution

Follow `.claude/skills/job-scraper/SKILL.md` Steps 0-6 unchanged. Run the tiers as **parallel search passes**, then merge into one dedup/store/present cycle — do not run the whole pipeline per tier, or the same remote-EU posting gets written to `seen_jobs.json` under several keys.

**Per-city rules are not duplicated here.** Apply them from the scoped skills:
- Milan and Ring 1: all five overrides in `.claude/skills/milan/SKILL.md` — especially **Override 1**, which forbids searching Ring 1 cities by name (verified dead lane; LinkedIn falls back to Milan results and manufactures false positives) and routes the belt through wide Milan location terms plus named-employer careers pages.
- Berlin: all four overrides in `.claude/skills/berlin/SKILL.md`.

### Ring 2 and Tier 2 — the conditional tiers

These exist to widen the funnel, not to lower the bar. Both forfeit something the Ideal tier has: Ring 2 gives up the Milan fallback market and the personal tie, Tier 2 gives up both cities entirely. The salary is what buys that back.

| Tier | Floor | Below floor |
|---|---|---|
| Ring 2 | 48K € | **Flag, never auto-pass.** Exception: an exceptional sector match (space, aerospace, defence, rail, geospatial) is worth surfacing below floor **with the trade-off stated** — never silently. |
| Tier 2 | 52K € | Flag. Zurich and Geneva need substantially more than 52K to be real, given cost of living — do not treat a nominal 52K there as clearing anything. |

Both floors were lowered on 2026-09-04 at the candidate's direction (Ring 2 50K → 48K, Tier 2 55K → 52K) to keep the tiers realistic against published numbers.

**Total-compensation figures are tested as base + 50% of the variable.** Never test the base alone and never test the headline alone; show the arithmetic in the Notes cell. Worked example in `04-job-evaluation.md`: Alpitronic's UI role states €64,000 total including a 30% variable, so base ≈ €44.8K and expected ≈ €54K — a clear pass that a base-only test would have wrongly rejected.

**Turin is the standing exception.** It holds Italy's densest aerospace cluster outside Rome (Thales Alenia Space, Argotec, Altec, Avio Aero), so the domain match on the ESA Galileo background is the strongest in the country — and the Turin **consultancies** will keep failing the 48K rule (verified 2026-09-04: aizoOn €30-35K, Teoresi €28.5-35K, Akkodis €27-35K). But international product employers and funded startups there clear it outright: Qualcomm/Arduino €55,200-81,600 and Nebuly €50,000-72,000, both English, both 3+ years. Screen Turin by employer type before assuming the band.

### Language gates are live across three languages

Italian, German and French all matter in this sweep, and only French is declared (B1).

- **Italian** — not declared. Job-condition requirement in Milan, Ring 1 or Ring 2 → hard FAIL, quoted back.
- **German** — not declared. Job-condition requirement in Berlin, Munich, Zurich or Bolzano → hard FAIL, quoted back. **Bolzano/South Tyrol is bilingual Italian/German and many local employers work in German** — read the working-language requirement on every South Tyrol posting, never infer it from the region. Alpitronic is applicable because it runs English in engineering, not because Bolzano is generally accessible.
- **French** — declared at B1. A Geneva or Luxembourg posting requiring French is a **FLAG, not a fail**: score and draft normally, but quote the posting's bar next to the B1 so the candidate judges it.

Search in English only. Postings merely *written* in another language, for roles that operate in English, are fine.

### Italy tax regime

If any Italian posting (Milan, Ring 1 or Ring 2) reaches offer discussion, surface the *regime impatriati* as a **two-sided** factor per `10-italy-tax-regime.md`: it can move an Italian net much closer to a higher Berlin or Tier 2 gross, and it carries a 4-year Italian tax-residence lock-in that makes a bad situation expensive to exit. Self-declared, unverified by an accountant. Never treat it as settled in the candidate's favour, never quote a net-with-regime figure to an employer, and never let it justify accepting a low gross.

## Search dense, present sparse

**Search wide.** Every priority category in `search-queries.md` against every enabled portal, across all tier passes — the `/scrape broad` behaviour, not the default top-3.

**Present narrow.** Show only the **top 15 across all tiers**, ranked by fit. This is higher than the 10 the other city skills use because the pool is wider; it is not licence to pad. Allocation follows quality, with one guarantee: **if Milan+Ring 1 or Berlin has at least 3 qualifying results, that tier gets at least 3 rows** — the wider net must never crowd out the preferred cities.

Everything fetched still goes into `seen_jobs.json` per Step 4 — results outside the top 15 are stored with `status: "skipped"` so they do not resurface every run. Report the discard count in the header.

## Output

Three tables in preference order. All use **exactly these five columns, in this order**, every run:

| Company | Position | Salary | URL | Notes |
|---|---|---|---|---|

- **Salary** — verbatim from the posting when stated (`45-55K €`, `80-120K € base + VSOP`, `95K CHF`). When the posting states nothing, write `not stated`. **Never estimate, infer, or carry a market range into this column.** Keep the posting's own currency; do not convert. Do not normalise across countries here — net-vs-gross comparison belongs in the trade-off block, not a table cell.
- **Notes** — **20 words maximum**, hard cap. Spend them on the single highest-value signal, in this priority order: hard gate failure → `re-apply, last <YYYY-MM>` → below-floor salary → tier-specific signal (`Ring 1 (<city>)`, remote / work-from-abroad terms, Ring 2 sector-match exception) → the one concrete reason this is a strong match. Not a summary of the posting.

Ring, remote and tier findings live inside the Notes budget. None gets its own column — the five-column schema is fixed across all four city skills.

```
## New Matches (wide) - YYYY-MM-DD
Top 15 of N new positions. Scope: Milan + Ring 1, Berlin, Ring 2, Tier 2, remote EU.
Excluded: positions applied to since <cutoff YYYY-MM-DD> (12-month cooldown) - M filtered.
(N-15 lower-fit results stored, not shown.)

### Milan + Ring 1 (N found)
| Company | Position | Salary | URL | Notes |

### Berlin (N found)
| Company | Position | Salary | URL | Notes |

### Wider net — Ring 2 / Tier 2 / remote EU (N found)
| Company | Position | Salary | URL | Notes |
(add the tier to each Notes cell: "Ring 2 Turin", "Tier 2 Amsterdam", "remote EU")

### Trade-off
(only when the wider net produced something genuinely competitive against a preferred
city — otherwise omit. Never present the wider tiers as equivalent to Milan or Berlin.)
```

Number the rows in a **single continuous sequence across all three tables** — prefix the Company cell (`1. Satispay`) rather than adding a sixth column — so the follow-up prompt ("give me the number(s)") stays unambiguous.

Keep the `skipped (disabled):` and `health:` lines from Step 5 when they apply. High-match highlights and the Step 4.5 contact links follow the tables, for the shortlisted results only.

### Everything is a table — no prose blocks

The three position tables are not the only tables. **Every section of the output is a table**, the analysis included. The Trade-off block is a table, not paragraphs. Prose is reserved for a single closing caveat where tabulating would misrepresent the finding — never for findings themselves, never for comparison, never for reasoning about a specific posting.

This matters most here: the wide sweep produces the most analysis of any city skill, so it is the one most likely to drift back into paragraphs. It must not.

Emit these after the position tables, in this order, and only when they have rows:

| Section | Columns | Emit when |
|---|---|---|
| **Gate failures and drops** | `Company \| Position \| Tier \| Dropped on \| Evidence (quoted)` | Any posting failed a screen. `Dropped on` is one of: `Language gate`, `Below floor`, `Stack gap`, `Discipline gap`, `Seniority gap`, `Mass posting`. Quote Italian, German and French text untranslated so the bar is auditable. |
| **Ring 1 findings** | `Company \| Position \| Belt city \| Outcome` | Any Milan-pass result landed outside Milan proper. Records that the belt is reachable through wide Milan terms even when every belt result fails on merit — the Override 1 evidence trail. |
| **Trade-off** | `Factor \| Milan + Ring 1 \| Berlin \| Wider net` | Two or more tiers produced competitive results. One row per factor: top pay stated, best stack fit, domain match, language risk, salary transparency, relocation support, remote / work-from-abroad. |
| **Framework corrections** | `File \| Current entry \| Evidence this run \| Change needed` | The run contradicted `04-job-evaluation.md` or `search-queries.md`. Do **not** edit those files from inside a search run — table the correction and let the user approve it. Ring 2 assumptions are the likeliest to break, since the tier is newest and thinnest on evidence. |
| **Already surfaced, still unapplied** | `Company \| Position \| First seen \| Status \| Why still relevant` | `seen_jobs.json` dedup suppressed a posting still marked `new` that is still a strong match. |
| **Run stats** | `Metric \| Count` | Always. Raw results, unique URLs, cooldown drops, seen_jobs drops, intra-run dupes, kept, detail-fetched, shortlisted, stored as skipped. |

Rules holding across all of them:

| Rule | Meaning |
|---|---|
| **Quote, do not summarise** | Any cell asserting what a posting requires carries the posting's own words in quotes. A paraphrased gate failure is not a gate failure. |
| **One fact per row** | Two gates failed at one company is two rows, not one cell holding both. |
| **No word cap** | The 20-word Notes cap is the position tables only. These tables exist to carry the detail that cap forced out. |
| **Empty means omitted** | A section with no rows disappears. Never emit an empty table or a "none found" placeholder row. |
| **Never rank in the Trade-off table** | It states each tier's position on a factor. It does not declare a winner — the preference rules below do that, in the one line of prose this output allows. |

### Trade-off rules when comparing across tiers

These govern what goes in the Trade-off table's cells, and the single line of prose allowed to name a preference.

| Rule | Detail |
|---|---|
| **Milan outranks Berlin; both outrank everything else** | Same band → Milan. Milan moderately below → still Milan. Milan far below → Berlin. No fixed flip threshold exists and none should be invented — present the comparison, let the candidate decide. |
| **Compare net, not gross** | Do this before calling any gap large. Italian contracts commonly pay 13 or 14 monthly instalments. Swiss and Irish tax and cost of living differ enough that gross figures are not comparable numbers. |
| **Remote and work-from-abroad are first-class factors, not perks** | The Milan preference is about time with one specific person there, so flexibility partly substitutes for the city. A well-paid Tier 2 role with generous work-from-abroad days can beat a poorly-paid Milan role *on the Milan criterion itself*. Give it its own Trade-off row when it applies. |

If the run yields 8+ new jobs, suggest `/rank` — it batch-scores everything against the full framework, which beats eyeballing three tables.
