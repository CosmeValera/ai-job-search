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
| **Tier 3 (Italy)** | **Turin** | The only Italian secondary market with real depth (Qualcomm/Arduino, TeamSystem, Accessiway, Zucchetti, Reply). Surface it. |
| **Remote** | Fully remote within the EU | Surface, treat as acceptable. |

**Milan satellite cities are a dead lane — checked 2026-08-16, do not re-run.** Searches on Bergamo, Como,
Monza, Novara and Piacenza returned *zero* locally-posted roles; LinkedIn silently falls back to Milan
results when a location has no matches, so a full-looking result set there is an artefact, not a finding.
Brescia and the Emilia belt (Parma, Reggio Emilia, Modena) do have local postings, but they are Italian SMEs
and consultancies: Italian-language postings, .NET/C# stacks, 28-50K € with most under 45K €. One exception
worth remembering — eGlue (Reggio Emilia) posted up to 50K € for a React role. Reggio Emilia is ~1h40 from
Milan, so treat the Emilia belt as its own location, never as a Milan commute.
| **Exclude** | Madrid and all of Spain | Hard exclude. |
| **Exclude** | UK, US, Singapore, anywhere outside the EU/EEA/Switzerland | Hard exclude. The UK rows in the historical tracker predate this focus and needed sponsorship. |

## Salary Filter

| City | Floor (the anchor to ask for) | Target | Surface anyway |
|---|---|---|---|
| Milan | 45K € | 45-50K €, ideally 48-50K €+ | 41-45K € — worth surfacing, marked as below floor |
| Berlin | 50K € | 50-60K €, ideally 55-60K €+ | — |

Milan's market reality, corrected 2026-08-16 against postings that actually name a number: the band is
**bimodal, and which mode a posting sits in is predicted by the employer type, not by the role title.**

- **Italian consultancies and staffing agencies** (Fincons, TXT, Teoresi, Akkodis, aizoOn, ADENTIS, KeyBiz,
  Softlab, Hays, ACTION ICT) cluster at **30-45K €** and top out at or just under the floor. This is the
  lane that produced the old "28-47K €" read.
- **Product companies and international employers** clear the floor comfortably: TeamSystem 53-66K €,
  Alpitronic from 64K €, Facile.it 45-60K €, Callimacus 45-55K €, plus the candidate's own tracker
  evidence (Satispay 50K €, Lexroom ~45K €).

### The full-stack title premium (found 2026-08-16, act on this)

TeamSystem posted two roles in the same week, both open to Milan, both hybrid, both English C1:

| Posting | Experience asked | Total compensation | Wellbeing wallet |
|---|---|---|---|
| Full-stack Engineer — Node.js & React (4376718721) | 5 yrs | **53-66K €** | 3.850 €/yr |
| Frontend Developer (4439991999) | 3 yrs | **33-37K €** | 350 €/yr |

Same employer, same cities, same language bar. The ~20K € gap tracks the **title and scope**, not location,
not seniority — the cheaper role is the one asking *fewer* years. Corroborated across the market: Accessiway's
*Fullstack* role is 30-45K €, while pure-frontend consultancy postings sit at 30-40K €.

**Consequence for searching:** query `full stack`, `fullstack`, `software engineer` and `product engineer`
at least as hard as `frontend`. A pure "Frontend Developer" title in Italy is structurally underpaid
regardless of employer quality. The candidate's profile genuinely supports the full-stack framing — Node and
Java/Spring backend contributions at GMV, plus Docker/Kubernetes/Helm/Jenkins — so this is a reframing of
real experience, not a stretch. Do not narrow Italian searches to frontend titles.

So **45K € is mid-band for Milan, not the ceiling** — an earlier read of this file put the Milan top near
47K € and that was too pessimistic, drawn from a consultancy-heavy sample. Weight searches toward product
companies and international employers with Milan offices; the consultancy lane is high-volume, low-yield and
also where the Italian-language requirements concentrate. Still **surface 41-45K € postings and label them**
rather than dropping them. Below ~40K € is not worth surfacing: current comp is 37.5K € gross, so those are
not a move.

### Berlin's floor is set too low — corrected 2026-08-16

The 50K € Berlin floor above was set before any Berlin posting with a published number had been seen. Two
independent ones have now turned up, both well-funded product startups, both Berlin:

| Posting | Band | Note |
|---|---|---|
| Almedia — Frontend | **80-140K €** | declined by candidate: 7+ yrs required |
| Buena — Senior Software Engineer | **80-120K € base** + virtual stock options | **no years requirement stated at all** |

So Berlin is bimodal the same way Milan is, but the split runs **funded-product-startup vs everything else**, and
the upper mode is far higher than the Milan equivalent: Berlin tops around **80-140K**, Milan around **53-66K**
(and the Milan figure includes variable, while Buena's is base). Treat 50K as the *exclusion* threshold, not the
target — surfacing a 50-60K Berlin role as a good outcome understates what the top of that market pays.

**This matters for the city decision.** `04-job-evaluation.md` ranks Milan above Berlin at equal compensation and
deliberately fixes no flip-threshold. Nothing here changes that rule — but the candidate's own stated condition
for choosing Berlin was a *substantially* higher offer, and an 80K+ base against a 53-66K Milan ceiling is that
condition being met on paper. Surface the gap as evidence when both cities produce offers; never treat it as
having decided anything on its own.

Flag postings below the floor rather than dropping them. Most postings state no salary at all — that is not a reason to exclude.

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
