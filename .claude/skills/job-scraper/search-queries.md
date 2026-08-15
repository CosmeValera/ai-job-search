# Search Queries for Job Scraper

## Installed portal CLIs (primary for `/scrape`)

`/scrape` discovers every portal skill under `.agents/skills/*/SKILL.md` and runs its CLI first. Shipped country-agnostic CLIs include `linkedin-search` and `freehire-search`; Danish demos and any skill you add with `/add-portal` are included the same way. You do **not** need a matching `site:` line below for those CLIs to run.

The `site:` query templates in this file are the **WebSearch fallback** — for portals without a CLI, company career pages, or when a CLI fails.

**Language scope:** all queries are written in **English**. Cosme's Languages table declares Spanish (Native), English (C1) and French (B1) — **not** Italian or German. Under `04-job-evaluation.md`'s Language Gate, a Milan or Berlin posting that requires Italian or German as a job condition is excluded before scoring, so searching in those languages surfaces results that are already disqualified. Both cities have large English-language tech markets; search there. Spanish-language queries are pointless because Spain is a deal-breaker location.

## Search Sites

Primary:
- **linkedin.com/jobs** — main source. Filter by Milan, Italy / Berlin, Germany. Also covered by the `linkedin-search` CLI
- **Ashby, Greenhouse, Lever, Personio, Workable, SmartRecruiters** — the ATS platforms that host most Milan/Berlin startup postings. Historically the highest-volume sources in the tracker
- **welcometothejungle.com** — strong coverage of Milan and Berlin tech
- **berlinstartupjobs.com** — Berlin-specific
- **Otta / Cord** — curated European tech roles

Secondary (company career pages via Google):
- Direct Google searches with `site:` filters for target companies

## Query Categories

Queries are grouped by priority. Combine each with `Milan` or `Berlin`.

### Priority 1: Frontend / React

The core direction. React and TypeScript are the strongest match areas.

```
site:linkedin.com/jobs "Frontend Developer" Milan
site:linkedin.com/jobs "Frontend Engineer" Berlin
site:linkedin.com/jobs "React Developer" Milan OR Berlin
site:jobs.ashbyhq.com "Frontend Engineer" React Berlin
site:job-boards.greenhouse.io "Frontend Engineer" Milan
site:jobs.lever.co React TypeScript Berlin
"frontend developer" React TypeScript Milano English
"frontend engineer" React Berlin "English speaking"
```

### Priority 2: JavaScript / TypeScript (generic)

Same work, different posting title. Catches roles that don't say "frontend".

```
site:linkedin.com/jobs "TypeScript Developer" Milan OR Berlin
site:linkedin.com/jobs "JavaScript Engineer" Berlin
site:jobs.ashbyhq.com TypeScript engineer Milan
"TypeScript developer" Milano
"JavaScript developer" Berlin English
```

### Priority 3: Full Stack

Backed by Node, Java (Spring), PostgreSQL. Heavily represented in past applications.

```
site:linkedin.com/jobs "Full Stack Developer" Milan
site:linkedin.com/jobs "Full Stack Engineer" Berlin
site:job-boards.greenhouse.io "Fullstack Engineer" React Node Berlin
"full stack developer" React Node Milano
"fullstack engineer" React Java Berlin
```

### Priority 4: Domain lane — space, aerospace, transport, geospatial

**Underexploited and the highest-leverage category.** The ESA Galileo work at GMV and the ADIF rail work at Capgemini are directly relevant here rather than incidental, and this is the one place the profile is genuinely differentiated rather than competing against every other mid-level React CV. Prioritise these results even when the raw keyword match looks weaker.

```
site:linkedin.com/jobs "software engineer" space Milan OR Berlin
site:linkedin.com/jobs frontend satellite OR aerospace Berlin
site:linkedin.com/jobs developer "earth observation" OR geospatial Berlin
"frontend developer" space OR satellite OR aerospace Milano
"software engineer" mobility OR railway OR transport Berlin React
site:linkedin.com/jobs developer GNSS OR navigation Milan OR Berlin
```

Named employers worth checking directly: OHB, ConstellR, LiveEO (already applied), Thales Alenia Space (Milan/Turin), Leonardo (Milan), D-Orbit (Milan), Isar Aerospace, PTScientists/Planetary Transportation, Telespazio.

### Priority 5: Broader net

Wider sweep. Use when the categories above run dry.

```
site:linkedin.com/jobs "Software Engineer" React Milan
site:linkedin.com/jobs "Software Engineer" frontend Berlin
site:berlinstartupjobs.com React
site:app.welcometothejungle.com frontend Milan OR Berlin
```

## Location Filter

Relocation is the goal, not an obstacle. Cosme is an EU citizen, so no sponsorship is required anywhere in this list.

| Tier | Locations | Handling |
|---|---|---|
| **Ideal** | **Milan** (first choice), **Berlin** | Full score. Milan outranks Berlin at equal compensation. |
| **Tier 2** | Dublin, Amsterdam, Munich, Luxembourg, Zurich, Geneva | Surface, rank below the Ideal tier. Included because they pay well, not because they are preferred. Switzerland needs a permit; the rest do not. |
| **Remote** | Fully remote within the EU | Surface, treat as acceptable. |
| **Exclude** | Madrid and all of Spain | Hard exclude. |
| **Exclude** | UK, US, Singapore, anywhere outside the EU/EEA/Switzerland | Hard exclude. The UK rows in the historical tracker predate this focus and needed sponsorship. |

## Salary Filter

| City | Acceptable | Target |
|---|---|---|
| Milan | 45-50K € | 48-50K €+ |
| Berlin | 50-60K € | 55-60K €+ |

Flag postings below the acceptable floor rather than dropping them. Most postings state no salary at all — that is not a reason to exclude.

## Language Filter

Apply `04-job-evaluation.md`'s Language Gate. For this profile that means: **any posting requiring Italian or German as a job condition is excluded.** A posting merely *written* in Italian or German, for a role that operates in English, is fine. English requirements at "fluent" or "professional" level pass cleanly against a Cambridge C1 plus a full-time English work placement in Ireland.

## Date Filter

Only include jobs posted within the last 14 days, or with an application deadline that has not yet passed. If a posting date cannot be determined, include it but flag as "date unknown".

## Dedup

`/rank` builds its exclusion set from company+role in `job_search_tracker.csv`. That file holds 173 historical applications imported from Cosme's own spreadsheet, so previously-applied companies are already excluded automatically.

## Adapting Queries

If the user specifies a focus area, select queries from the matching category and also generate 2-3 custom queries for that focus. For example:
- `/scrape space` -> Priority 4 queries plus custom space-sector queries
- `/scrape milan` -> every category, restricted to Milan
