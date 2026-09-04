---
framework_version: 1.5.0
---

# Job Evaluation Framework

<!-- SETUP: Skill match areas and career goals are personalized by running /setup -->

## Eligibility Gate — run before scoring

If the candidate is not a citizen or permanent resident of the country they are applying in, run this first. It is a hard filter, not a scoring dimension, and it is separate from work-permit *timing*: timing asks "can they work the required hours yet?", eligibility asks "are they permitted to hold this job at all?". A candidate can pass timing and still be categorically excluded.

Read the posting's eligibility / work rights / "who can apply" section **verbatim** and classify:

| Posting wording | Verdict |
|-----------------|---------|
| Names a **citizenship or permanent-residency requirement** ("must be a citizen of X", "permanent resident", "PR required", "full working rights" where the employer means citizen/PR) | **FAIL — hard stop.** Do not score, do not draft. Quote the exact wording back to the user. |
| Requires a **security clearance** at any level | **FAIL** in most countries, since clearance is normally gated on citizenship. Verify the specific scheme rather than assuming. |
| **Explicitly names** the candidate's permit class, or says "international applicants welcome", "visa holders considered", "we sponsor" | **PASS** — verified acceptance. Worth noting as a positive in the application. |
| **Silent** on citizenship or residency | **PROCEED, but mark unverified.** Check the employer's own careers or international-applicant page before drafting. |

**Two rules that are easy to get wrong:**

1. **Silence is not permission.** Large graduate programs frequently gate eligibility on their own website rather than in the job ad. Highest-risk categories: professional-services firms, government and defence, banking, telecommunications, and anything touching critical infrastructure.
2. **A company-wide "we accept international applicants" statement is not role-level permission.** The common pattern is a general welcome followed by a *named list* of the specific programs or service lines it covers. Confirm the **specific posting or stream** appears on that list before drafting.

**Report an eligibility failure to the user with the quoted source** rather than silently dropping the role. They may know something about their own status that the profile does not record.

If the candidate's permit also constrains *hours* or *start date* (a student visa with a term-time cap, a permit that begins on graduation), record that as a second gate under this section during `/setup`, with the specific dates. Do not merge it with the eligibility question above — they fail for different reasons and need different answers.

A role that fails this gate is not scored and not drafted. Everything below applies only to roles that pass it.

## Language Gate — run before scoring

No dimension or gate anywhere in this framework currently checks a posting's language requirements against what the candidate actually speaks - it is not one of the five Scoring Dimensions below, not a field `/scrape` or `/rank` track, and not something `/apply`'s language detection (Step 1, which already extracts a posting's required language generically) has anywhere to report to. This gate adds that check, structured the same way as the Eligibility Gate above: read the posting, classify against profile data, and treat a hard mismatch as FAIL before scoring.

Read the posting's language requirements as stated for **the role itself** — not the language the ad happens to be written in. A posting written in a language you don't work in, for a role that only needs languages you do work in on the job, passes fine; only an explicit job-condition requirement ("fluent X required," "must communicate with the Y team in Z") triggers this check. For each language the posting requires as a job condition, compare it against your Languages table in CLAUDE.md / `01-candidate-profile.md`:

| Posting requirement vs. your Languages table | Verdict |
|---|---|
| Requires a language **not on your table at all** (e.g. "fluent Polish required," "must communicate with the Warsaw team in Russian," and you list no Polish/Russian row) | **FAIL — hard stop.** Do not score, do not draft. Quote the exact requirement line. |
| Requires a language you **do** list, but the posting's stated bar (as written — "fluent," "native," "C1+," "business-level") reads as plausibly **higher** than your declared level | **FLAG, then proceed.** Not a fail. Score and draft normally, but surface the gap explicitly in your report to the user (quote both the posting's requirement and your declared level) so they can judge it themselves — bars like "fluent" vary a lot by company and geography, and a recruiter may be flexible. Never silently drop the posting and never silently treat it as a clean pass. |
| Requires a language you list, at or below your declared level (or the posting doesn't specify a level at all — just names the language) | **PASS.** No note needed. |

Judge the level comparison the same way you judge everything else in this framework: read both sides as written and reason about it, don't force either into a rigid scale — CEFR letters, LinkedIn-style buckets ("professional working proficiency"), and plain-English words ("conversational," "fluent," "native") all appear in the wild and don't map onto each other precisely. When genuinely unsure whether a stated bar exceeds the candidate's level, prefer FLAG over a silent PASS — the human is meant to be the tiebreaker, not the gate.

**Worked example:** a candidate whose Languages table lists Spanish (Native) and English (B1/B2). A posting requiring "fluent Russian" → **FAIL**, Russian isn't declared at all. A posting requiring "fluent English" → **FLAG**, English is declared but "fluent" plausibly exceeds B1/B2 — score and draft the application, but tell the candidate this posting's bar may be a stretch and let them decide. A posting requiring "conversational English" or unspecified English → **PASS**, B1/B2 clears a "conversational" bar cleanly.

## Scoring Dimensions

Evaluate each job posting against these six dimensions:

### 1. Technical Skills Match (0-100)
How well do the required/preferred skills align with the candidate's capabilities?

| Score | Meaning |
|-------|---------|
| 80-100 | Core requirements are primary skills |
| 60-79 | Most requirements match, 1-2 gaps that are learnable |
| 40-59 | Partial match, significant upskilling needed |
| 0-39 | Fundamental mismatch |

**Strong match areas:** React, TypeScript, JavaScript, CSS, Angular, responsive web development, Git
**Moderate match areas:** Node, Java (Spring), PostgreSQL/MySQL, Docker, Kubernetes, Helm, Kustomize, Jenkins, GitHub Actions, AWS (EC2, RDS, S3, Lambda), Figma, Scrum
**Weak match areas:** Vue, Svelte, Next.js/SSR frameworks, React Native/mobile, Python at production scale, Go/Rust/Kotlin, data engineering, ML, native cloud architecture beyond the AWS services listed above, formal leadership or line management

### 2. Experience Match (0-100)
Does work history align with what they're looking for?

| Score | Meaning |
|-------|---------|
| 80-100 | Direct experience in the same domain and role type |
| 60-79 | Related experience, transferable skills clear |
| 40-59 | Adjacent experience, would need to make the case |
| 0-39 | Unrelated experience |

**Strong:** Frontend development in React/TypeScript for enterprise and institutional clients; Angular full-stack delivery; working inside Agile teams against external stakeholders (ESA, ADIF/Renfe)
**Moderate:** Full-stack roles where the backend is Node or Java/Spring; DevOps-adjacent roles touching Docker, Kubernetes, Helm, and Jenkins; roles in space, aerospace, transport, or critical-infrastructure domains
**Entry-level:** Senior or lead titles (approximately 4 years total experience as of 2026, no line management); backend-primary roles; anything requiring a language beyond Spanish/English/French

### 3. Behavioral/Culture Fit (0-100)
Does the role and company culture match the behavioral profile?

| Score | Meaning |
|-------|---------|
| 80-100 | Culture strongly matches behavioral preferences |
| 60-79 | Mixed signals but mostly compatible |
| 40-59 | Some friction areas |
| 0-39 | Significant culture mismatch |

**Red flags to research:** Department disorganization, work dominated by maintenance over development, poor chemistry with leadership, culture mismatches. Check reviews, media coverage, LinkedIn connections, and network contacts for insider perspective.

### 4. Location & Logistics (Pass/Fail + Notes)

Relocation is the **goal**, not a deal-breaker. The candidate currently lives in Madrid and is actively seeking to leave Spain. Score location in tiers:

| Tier | Locations | Verdict |
|---|---|---|
| **Ideal** | Milan, Italy | PASS, top priority |
| **Ideal** | Berlin, Germany | PASS, top priority |
| **Ring 1** | Milan commuter belt: Monza/Brianza, Bergamo, Como, Lecco, Varese, Novara, Pavia | PASS, **treat exactly as Milan** — same floor, same priority. See Ring 1 below. |
| **Ring 2** | Northern Italy, separate labour markets: Turin, Genoa, Bologna, Verona, Trento, Bolzano/Bozen | **Conditional PASS at 48K € or above**, or on an exceptional sector match. See Ring 2 below. |
| **Tier 2** | Dublin, Amsterdam, Munich, Luxembourg, Zurich/Geneva | PASS, surface but rank below Milan/Berlin. These qualify because they pay well, not because the candidate prefers them. |
| **Remote** | Fully remote within the EU | PASS |
| **FAIL** | Madrid or anywhere in Spain | The candidate is explicitly not seeking roles in Spain. |
| **FAIL** | Outside the EU/EEA/Switzerland | Ignore; not being pursued. Historical UK and Singapore rows in the tracker predate this focus. |

#### Ring 1 — the Milan commuter belt (added 2026-09-04)

Monza/Brianza, Bergamo, Como, Lecco, Varese, Novara, Pavia. All roughly 40-60 minutes from Milano Centrale by regional rail. The candidate can live in Milan and commute, so **the Milan personal tie is fully preserved** and so is the Milan fallback job market if the role sours.

Score these as Milan in every respect: same 45K € anchor, same 41-45K € stretch-down, same priority ranking against Berlin. **The point of Ring 1 is funnel volume, not money** — see the pay-geography note below.

Employers here that hit the space / aerospace / rail lane: D-Orbit (Fino Mornasco, Como), Leonardo Helicopters (Cascina Costa, Vergiate, Sesto Calende — Varese), Alstom (Sesto San Giovanni), ST Microelectronics (Agrate Brianza), Brembo (Stezzano, Bergamo), Scame (Bergamo province) and Daze (Vimercate) in EV charging. **Treat this list as leads to verify, never as established fact** — confirm every company claim independently before it reaches a CV or cover letter.

Defence caveat: Italian defence clearance is normally gated on Italian citizenship. The candidate is a Spanish national, so run the Eligibility Gate carefully on Leonardo and similar postings rather than assuming EU citizenship is enough.

#### Ring 2 — northern Italy, standalone cities (added 2026-09-04)

Turin, Genoa, Bologna, Verona, Trento, Bolzano/Bozen. Moving to any of these means **committing to that city's job market**: no Milan fallback, no personal tie, and a second relocation if it goes wrong. That forfeited optionality is what the salary premium has to buy.

| Situation | Verdict |
|---|---|
| 48K € or above | **PASS.** Surface it, rank alongside Milan. |
| Below 48K €, but exceptional sector match (space, aerospace, defence, rail, geospatial) | **FLAG to the user with the trade-off stated.** Do not auto-pass and do not silently drop. |
| Below 48K €, ordinary role | **FLAG as below floor**, consistent with the rest of this framework — never drop a posting silently. |

Floor lowered from 50K to **48K on 2026-09-04**, at the candidate's direction, to keep the tier realistic against what Italian postings actually publish.

**Variable pay counts, at half.** When a posting states a total-compensation figure that bundles a bonus, do not test the base alone and do not test the headline alone. Test **base + 50% of the variable** — the realistic expected cash — and show the arithmetic. Alpitronic's UI role is the worked example: €64,000 total including a 30% variable puts base near €44.8K and variable near €19.2K, so the expected figure is roughly **€54K**, clearing the floor. Testing the base alone would have wrongly rejected it.

Do not treat Frecciarossa journey times as commutability. Bologna and Turin are roughly an hour from Milan by high-speed rail — faster than Bergamo by regional train — but daily high-speed fares make that a non-option. Ring membership is about labour markets, not minutes.

**Turin is the standing exception worth understanding.** It holds Italy's densest aerospace cluster outside Rome (Thales Alenia Space, Argotec, Altec, Avio Aero), so the domain match on the ESA Galileo background is the strongest in the country. Turin tech pay typically sits *below* Milan, so the Turin **consultancies** will keep failing the 48K rule that makes Ring 2 worth it — verified 2026-09-04: aizoOn €30-35K, Teoresi €28.5-35K, Akkodis €27-35K.

But the exception has an exception, confirmed on the same run: **international product employers and funded startups in Turin do clear the floor outright.** Qualcomm/Arduino posted €55,200-81,600 and Nebuly €50,000-72,000 plus stock, both in English, both asking 3+ years. This is the same rule as Milan — employer type buys the salary, geography only buys funnel volume. Screen Turin by employer type before assuming the pay band.

**Bolzano / South Tyrol carries a language risk the other Ring 2 cities do not.** The province is bilingual Italian/German and many local employers use German as a working language, which trips the Language Gate above — German is not on the candidate's Languages table. Alpitronic is applicable because it runs English in engineering, not because Bolzano is generally accessible. Read the working-language requirement on every South Tyrol posting; do not infer it from the region.

**Alpitronic posts several roles at once — check which one before quoting a number.** Two were live in September 2026 and they are not interchangeable:

| Posting | Compensation | Stack | Verdict |
|---|---|---|---|
| **Software Developer - User Interfaces (m/f/d)**, Charger Interfaces | "Total compensation starting from € 64,000 gross per year, including a 30% variable bonus" — base ≈ €44.8K, expected ≈ €54K | HTML5/CSS3/modern JavaScript, "at least one modern JavaScript framework React, Vue, Angular", C++ backend familiarity, Qt a plus. "Fluent in English" | **The Ring 2 exemplar.** Clears the floor. Language gate passes. This is the one the candidate applied to. |
| **Full Stack Developer - Cybersecurity Hardening (m/f/d)** | "Total compensation starting from € 52,000 gross per year, including a 20% variable bonus" — base ≈ €43.3K, expected ≈ €48K | "front-end (Vue.js/Nuxt.js) and back-end (C#/.NET and PHP/Laravel)" | **Drop on stack**, not on pay. Vue + .NET + Laravel is a genuine mismatch against React/TypeScript/Angular + Node/Java. |

The UI role is the one that justifies Ring 2's existence. Never let the weaker sibling posting be mistaken for it.

#### Pay geography in northern Italy — do not re-derive this wrong

Two intuitions that look right and are not:

1. **"Further north pays more" does not hold against Milan.** Milan is the pay peak for tech in Italy. Bergamo, Brescia, Varese and Como pay *below* Milan — lower cost of living, thinner employer competition. The real north-south gap in Italian wages is Milan-vs-the-south, not north-of-Milan-vs-Milan.
2. **Alpitronic's 64K is an Alpitronic effect, not a Bolzano effect.** It is a hypergrowth HPC-charger hardware company hiring against German and Austrian employers, so it pays German-market rates. South Tyrol's median tech pay is above the Italian average (autonomous province, high cost of living, Austrian border) but nowhere near that figure. Never generalise one outlier employer into a regional band.
3. **A single employer's own postings do not share a band either.** Alpitronic ran 64K and 52K roles simultaneously — see the two-posting table above. Quote the specific requisition, never "Alpitronic pays 64K".

The consequence for search strategy: **geography buys funnel volume, employer type buys salary.** The reliable route to 55-65K is not a different city — it is a different kind of employer in the same cities: foreign-HQ companies paying to a non-Italian band, funded scaleups, and deeptech or hardware firms competing for engineers internationally. Weight employer type at least as heavily as location when prioritising a shortlist.

**Milan outranks Berlin, but not at any price.** The Milan preference is personal (a close personal tie in the city) and it is real, but the candidate weighs it against money rather than above money. Apply this order:

| Situation | Choice |
|---|---|
| Milan and Berlin offers within roughly the same band | **Milan** |
| Milan moderately below Berlin | **Milan**, unless something else is wrong with it |
| Milan far below Berlin | **Berlin.** Take the money, visit Milan. |
| Only one city has an offer | That city |

There is **no fixed threshold** where the preference flips, and none should be invented. The candidate's "40K Milan vs 60K Berlin" was an illustration of the principle, not a rule to apply literally — asked about a narrower 40K/55K gap, their own answer was "case by case". Present the comparison and let the candidate decide; both the money and the Milan tie carry real weight, and the trade-off is theirs to make each time.

**Remote policy is a first-class factor in this trade-off, not a perk.** The Milan preference is about being able to spend time with a specific person there, so anything that buys physical flexibility partly substitutes for the city itself. When comparing offers, always surface:
- Remote or hybrid days per week, and whether they are policy or manager's discretion
- Work-from-abroad / "work from anywhere" allowances (Doctolib's 10 days a year, Flix's 60 days, SumUp's sabbatical — these vary enormously and are usually buried in the benefits list)
- Fully remote EU roles, which sidestep the choice entirely

A well-paid Berlin role with generous work-from-abroad days can beat a poorly-paid Milan role *on the Milan criterion itself*. Say so explicitly when it applies.

Compare **net, not gross**, before calling any gap large - see the tax note under Salary bands.

**Work eligibility:** Spanish national, so an EU citizen. Every Ideal and Tier 2 location except Switzerland requires no sponsorship. Do not spend evaluation effort on visa questions for EU roles, and never treat a "no sponsorship available" line as a blocker for them.

- Frequent international travel: FLAG (discuss with user)

### 5. Career Alignment & Motivation (0-100)
Does this role advance career goals and contain tasks that energize?

| Score | Meaning |
|-------|---------|
| 80-100 | Strongly aligned with career direction, clear growth path |
| 60-79 | Good role but only partially aligned with long-term goals |
| 40-59 | Decent job but doesn't build toward career goals |
| 0-39 | Dead end or backwards step |

**Career goals:**
- Relocate out of Spain to Milan (first choice) or Berlin
- Grow from mid-level toward senior frontend / full-stack, on React and TypeScript
- Work in an English-speaking international team
<!-- Confirm and extend these during /setup --section goals. They are inferred from the
     application history and the relocation targets, not stated by the candidate directly. -->

**Motivation filter:** Evaluate not just whether you *can* do the tasks, but whether the tasks will *energize* you. Consider:
- Tasks that energize: *(not yet captured - ask the candidate)*
- Tasks that drain: *(not yet captured - ask the candidate)*
- Non-task factors: leadership style, department culture, company values, degree of autonomy

**Life situation alignment:**
- **Security**: Currently employed at GMV, so there is no urgency. This is a search for a better role, not a search under pressure. A weak offer can be declined.
- **Current compensation: 37.5K € gross at GMV (Madrid).** This is the number every offer is measured against, and it is the reason a 40K Milan offer is not attractive: it barely clears the status quo while adding relocation cost. **Never write this figure into a CV, cover letter, or any outbound message** - it is an internal evaluation input only.
- **Salary bands** (revised by the candidate 2026-08-16):

  | City | Stated floor (the anchor) | Target | Stretch-down |
  |---|---|---|---|
  | Milan | 45K € | 45-50K €, ideally 48-50K €+ | 41-45K € occasionally, to widen the funnel |
  | Ring 1 (Milan commuter belt) | 45K € — same as Milan | same as Milan | same as Milan |
  | Ring 2 (Turin, Genoa, Bologna, Verona, Trento, Bolzano) | 48K € | 48K €+ | none — below 48K, flag rather than pass, exceptional sector match included. Total-comp figures tested as base + 50% variable |
  | Berlin | 50K € | 50-60K €, ideally 55-60K €+ | none |

  **The 45K Milan anchor is a negotiating position, not a filter.** Always ask for 45K in Milan even when the posting's stated maximum is below it - the candidate did exactly this at AS Watson (stated max 41K) as a deliberate stretch, on the reasoning that no offer is lost by asking. Applying to a 41-45K Milan posting is allowed and sometimes right; quoting a number below 45K first is not.

  Below the acceptable floor, flag the posting explicitly rather than dropping it. Do not treat an unstated salary as a failure - most postings omit it.

- **Compare net, not gross, across the two cities.** A Milan gross figure and a Berlin gross figure are not comparable numbers. Two structural reasons:
  - Italian contracts commonly pay **13 or 14 monthly instalments**, so an Italian "annual" figure quoted per-month differs from the German convention.
  - Italy's **inbound-worker tax regime** (*regime impatriati*) exempts a large share of employment income from tax for several years for workers who move their tax residence to Italy and meet the conditions. The 2024 reform reduced the exemption and tightened eligibility, so the current rules must be checked rather than assumed. If it applies, it can move a Milan net figure much closer to a higher Berlin gross than the raw numbers suggest.

  **Full detail, eligibility analysis, risks and the pre-signing checklist live in `10-italy-tax-regime.md`.** The candidate believes they qualify via the ICT-experience route, but that assessment came from an AI chatbot and has not been checked by an Italian accountant, so it is recorded there as unverified. Three things carry into every Milan evaluation:
  - The regime is **self-declared**, so an error surfaces years later as a repayment demand, not as a rejected application.
  - It carries a **4-year Italian tax residence lock-in**. Leaving early means repaying the whole benefit with interest. Since the reason Milan ranks first is one personal tie, always surface this as a two-sided factor: it improves a good Milan offer and it makes a bad Milan situation expensive to exit.
  - Tax residence needs >183 days in a calendar year, so a second-half move usually pushes the benefit and the clock to the following January. **Start date is negotiable and worth negotiating** when an offer lands late in the year.

  **Treat this as a question to verify with the employer or an Italian accountant before comparing offers - never as a settled fact in the candidate's favour, and never as a reason to accept a low Milan gross on its own.** The gross is what survives if the regime does not apply, what the next employer anchors on, and what the pension is built from. Never quote a net-with-regime figure to an employer or write one into any document.
- **Flexibility**: *(not yet captured)*
- **Professional development**: *(not yet captured)*

### 6. Compensation (0-100) — added 2026-09-04

Scored, not just gated. Added at the candidate's direction after a `/rank` run put three postings with **no stated salary** in the top three positions: the four original dimensions contain no money term at all, so an unpriced posting could not lose a single point for being unpriced. That is backwards. A job whose pay cannot be checked against the floor is not more applyable than one that publishes a band clearing it by 30K.

Score against **the floor for that posting's tier** (Milan / Ring 1 45K, Ring 2 48K, Berlin 50K, Tier 2 52K):

| Posting's stated compensation | Score |
|---|---|
| Clears the tier floor by 20K € or more at the **bottom** of the band | 100 |
| Clears the floor by 10K € or more at the bottom of the band | 85 |
| Clears the floor across the **whole** band | 75 |
| Straddles the floor — clears it only in the upper part of the band | 55 |
| **Not stated** | 40 |
| Entirely below the floor | 25 |

**Why `not stated` scores 40 and not 50 or 60.** It sits below every band that actually clears the floor, and above one that is verifiably too low. An unpriced posting is not neutral — it is an unpaid research task handed to the candidate, and most of the historical tracker's 173 applications were exactly this. It should still be applyable when the fit is exceptional, which 40 permits: a posting scoring 90/90/85/90 on the other dimensions still lands at Strong Fit with a 40 here. It just cannot outrank a comparable posting that published a good number.

Rules:

| Rule | Detail |
|---|---|
| **Never estimate the band** | Score from the posting's own figure. A market estimate, a Glassdoor range or a guess from company stage is not a stated salary — it is `not stated`, 40. |
| **Variable pay at half** | Same rule as everywhere else in this framework: test base + 50% of the variable, and show the arithmetic. |
| **Equity is not salary** | "Competitive salary plus stock options" with no number is `not stated`. Note the equity separately; never let it lift the score. |
| **Italian instalments** | An Italian RAL quoted over 13 or 14 instalments is still that RAL. Do not multiply it up. |
| **The floor is the tier's, not Milan's** | A 49K Turin posting fails Ring 2's 48K... barely (55, straddling). The same 49K in Berlin is entirely below floor (25). Read the location first. |

### 7. Salary Benchmark (Optional)

If the salary lookup tool is configured (`salary_data.json` exists), look up the company:
```
python salary_lookup.py "<Company Name>" --json
```

If a city is known from the posting, add `--city "<City>"` to narrow results.

Present findings as:
```
### Salary Benchmark
| Metric | Value |
|--------|-------|
| [Category] index | XX.X (+/-X.X% vs baseline) |
| Overall index | XX.X (+/-X.X% vs baseline) |
```

Interpret results relative to the baseline defined in the data file's metadata. For index-based data, higher typically means above-market compensation.

If the salary tool is not configured, skip this section.

## Output Format

Present the evaluation as:

```
## Job Fit Evaluation: [Role] at [Company]

| Dimension | Score | Notes |
|-----------|-------|-------|
| Technical Skills | XX/100 | [brief note] |
| Experience Match | XX/100 | [brief note] |
| Behavioral Fit | XX/100 | [brief note] |
| Career Alignment | XX/100 | [brief note] |
| Compensation | XX/100 | [stated band vs tier floor, or "not stated"] |
| Location | PASS/FAIL | [brief note] |

**Overall Score: XX/100** (weighted average of scored dimensions)

### Verdict: [Strong Fit / Good Fit / Moderate Fit / Weak Fit / Poor Fit]

### Key Strengths for This Role
- [bullet points]

### Gaps to Address
- [bullet points]

### Recommendation
[1-2 sentences: apply/skip/apply with caveats]

### Company Research Checklist
- [ ] Checked company website (mission, values, recent news)
- [ ] Checked review sites (Glassdoor, Jobindex, etc.)
- [ ] Checked LinkedIn for team size, recent hires, connections
- [ ] Checked media for restructuring, growth, or workplace issues
- [ ] Identified network contacts who may know the team/manager
```

## Weighting

Revised 2026-09-04 when Compensation was added. The four original weights were rescaled proportionally so their ratios to each other are unchanged — only the new dimension's 20% is new money.

| Dimension | Weight | Was |
|---|---|---|
| Technical Skills | 24% | 30% |
| Experience Match | 20% | 25% |
| Behavioral Fit | 12% | 15% |
| Career Alignment | 24% | 30% |
| **Compensation** | **20%** | — |

(Location is pass/fail, not weighted — but it orders the shortlist, see below)

## Shortlist ordering — fit, pay and location together

The weighted score answers "how good a match is this?". It does not answer "which should I open first?", and those are different questions. **Every shortlist this repo produces — `/rank`, `/milan`, `/berlin`, `/milan-berlin`, `/milan-berlin-wide` — is ordered top-to-bottom by how eager the candidate should be to apply**, never by raw score alone and never grouped so that a weak preferred-city role sits above a strong one elsewhere.

Order by, in sequence:

| # | Key | Detail |
|---|---|---|
| 1 | **Actionability** | A posting whose pay clears its tier floor outranks one whose pay is unknown, which outranks one below floor. This is already inside the Compensation dimension; it is restated here because it must survive into the ordering. |
| 2 | **Weighted score** | The six-dimension total. |
| 3 | **Location preference** | Milan / Ring 1 > Berlin > Ring 2 > Tier 2 > remote EU. Applies as a **tiebreak within a band**, not as a grouping — it reorders near-equal rows, it never lifts a Moderate Milan role above a Strong Berlin one. |
| 4 | **Language gate clean** | A PASS outranks a ⚠ FLAG at equal standing. |

**Do not emit tier-grouped tables as the primary output.** One ranked list, best first. Tier belongs in a Location column so the reader can see it without the list being fragmented by it. The candidate reads from the top and stops when they lose interest — that only works if row 1 is genuinely the row to apply to first.

The shortlist length is whatever the run justifies. 10, 15 and 20 are illustrations, not fixed sizes; 13 or 18 is equally fine. Never pad to reach a round number and never truncate a genuinely qualifying posting to stay under one.

## Thresholds
- **Strong Fit** (75+): Definitely apply, tailor everything
- **Good Fit** (60-74): Apply, address gaps in cover letter
- **Moderate Fit** (45-59): Consider carefully, discuss with user
- **Weak Fit** (30-44): Probably skip unless strategic reasons
- **Poor Fit** (<30): Skip

## Calibration from Past Applications

Drawn from `job_search_tracker.csv` (173 rows, June 2025 - August 2026), imported during `/setup` Path A. There are no `documents/applications/` folders yet, so there is no cover letter, CV draft, or interview feedback to learn from - only the outcome column.

**The headline number:** 173 applications, 1 recorded response that progressed (Amplemarket, Milan, technical quiz). That is a response rate near 0.6%. Whatever the cause, it is not a shortage of applications sent.

What this implies for evaluation:

- **Volume is not the bottleneck; targeting and materials are.** Prefer a smaller number of properly tailored applications over another wide sweep. The `/apply` workflow exists for exactly this, and none of the 173 rows went through it.
- **The historical spread was extremely wide** - London, Dublin, Berlin, Milan, Amsterdam, Munich, Luxembourg, Zurich, Prague, Singapore - and much of it needed visa sponsorship the candidate did not need for EU roles. Effort was spread across markets where a non-resident applicant is filtered out early. The Milan/Berlin focus set during this setup is a correction; hold it.
- **Big-tech and highly competitive listings are heavily represented** (Meta, Apple, Microsoft, Amazon, Stripe, Shopify, Pinterest, Zalando, PayPal, Klarna) with no recorded progression. Score these honestly against the experience dimension rather than optimistically: roughly 4 years, no senior scope, no line management.
- **No sector concentration is visible.** Fintech, crypto, mobility, energy, HR-tech, and travel all appear once or twice. The candidate's actual differentiator - space/satellite systems via ESA Galileo, and rail infrastructure via ADIF - was not being used to target adjacent employers. Aerospace, space, defence, transport, and geospatial companies in Milan and Berlin are an underexploited lane where the GMV experience is directly relevant rather than incidental.

**Do not over-read the outcome column.** The statuses were reconstructed during import from the candidate's rule of thumb (dead unless recent), not recorded per-application at the time. Treat `no_response` here as "no reply received", never as evidence of a specific rejection reason.

## Pre-Application: Call the Employer (Best Practice)

Before writing the application, consider whether the candidate should call the contact person listed in the posting. **Only call if there are substantive questions** - never call just to "be remembered."

### When to Suggest Calling
- The posting has unclear or ambiguous requirements
- It's unclear which competencies are essential vs. nice-to-have
- The role description is vague about day-to-day tasks
- There's a named contact person who invites questions

### Good Questions to Ask
- "What are the primary challenges in this role?"
- "How is time typically divided across the listed responsibilities?"
- "Which competencies are most critical for success in this position?"
- "What does success look like in the first 6-12 months?"

### Rules for the Call
- Prepare a 30-second "elevator pitch" about your background in case they ask
- The call's purpose is **gathering information**, not delivering a pitch
- Take notes - use what you learn to tailor the application
- Reference the conversation naturally in the cover letter ("After speaking with [name], I was especially drawn to...")
