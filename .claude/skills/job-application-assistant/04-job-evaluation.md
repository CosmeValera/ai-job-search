---
framework_version: 1.2.2
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

Evaluate each job posting against these five dimensions:

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
| **Tier 2** | Dublin, Amsterdam, Munich, Luxembourg, Zurich/Geneva | PASS, surface but rank below Milan/Berlin. These qualify because they pay well, not because the candidate prefers them. |
| **Remote** | Fully remote within the EU | PASS |
| **FAIL** | Madrid or anywhere in Spain | The candidate is explicitly not seeking roles in Spain. |
| **FAIL** | Outside the EU/EEA/Switzerland | Ignore; not being pursued. Historical UK and Singapore rows in the tracker predate this focus. |

**Milan outranks Berlin.** At equal compensation, prefer Milan (personal reasons). Berlin wins only when Milan has no comparable offer, or when the Berlin offer is substantially higher.

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
- **Salary bands** (from the candidate's own tracker legend, confirmed during setup):

  | City | Acceptable | Target |
  |---|---|---|
  | Milan | 45-50K € | 48-50K €+ |
  | Berlin | 50-60K € | 55-60K €+ |

  Below the acceptable floor, flag the posting explicitly rather than dropping it. Do not treat an unstated salary as a failure - most postings omit it.
- **Flexibility**: *(not yet captured)*
- **Professional development**: *(not yet captured)*

### 6. Salary Benchmark (Optional)

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
| Location | PASS/FAIL | [brief note] |
| Career Alignment | XX/100 | [brief note] |

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
- Technical Skills: 30%
- Experience Match: 25%
- Behavioral Fit: 15%
- Career Alignment: 30%

(Location is pass/fail, not weighted)

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
