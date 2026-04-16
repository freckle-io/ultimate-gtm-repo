---
name: win-loss-analysis
description: Run a comprehensive closed-won vs closed-lost analysis on CRM deal data (HubSpot, Salesforce, or CSV fallback). Outputs a structured report with TL;DR, ICP canon vs reality finding, unified signal dashboard, deal stories, and association coverage explanation. Use when the user asks for a win/loss analysis, deal pattern analysis, ICP validation from CRM data, sales post-mortem, or "why are we winning/losing deals."
---

# Win/Loss Analysis

Run a comprehensive closed-won vs closed-lost analysis on CRM deal data. Outputs actionable signals for sales strategy, ICP refinement, and outbound targeting.

This skill is CRM-agnostic with a strong bias toward HubSpot and Salesforce (the two systems with first-class MCP coverage). It will also accept a CSV fallback when no CRM MCP is connected.

See `examples/sample-win-loss-report.md` for a full reference output.

---

## What This Skill Produces

A structured report — to Notion if connected, otherwise to a markdown file in the working directory — containing:

- TL;DR with key trends and patterns (top of the report)
- Headline finding: ICP canon vs. reality
- Top 3 recommended actions
- Unified signal dashboard (win signals + anti-signals in one table with an action column)
- Funnel overview (total created → open → won → lost, with both creation-cohort and closed-basis rates)
- Revenue over time
- Deal velocity + stale-deal threshold
- Firmographics (employee count, industry, geography) with over-index ratios
- Buyer persona patterns with grouped title examples
- Deal stories (best wins + instructive losses as narrative)
- Activity signals, source, and motion analysis
- Association coverage explanation (why some deals don't appear in firmographic tables)
- Data quality notes
- Methodology footer

---

## Step 0 — Discovery & Scoping (do not skip)

Before touching any data, work through the following with the user. Do not assume answers. Confirm each one.

### 0.1 CRM connector discovery

Check the currently-available tools for a CRM MCP in this priority order:

1. **HubSpot MCP** — look for tools like `search_crm_objects`, `search_properties`, `get_crm_objects`. If present, use HubSpot.
2. **Salesforce MCP** — look for Salesforce-flavored tools (`query_opportunities`, SOQL-style tools, or a Salesforce-named MCP). If present, use Salesforce.
3. **Other CRM MCPs** — if a different CRM MCP is connected, adapt the approach; the analysis framework is CRM-agnostic.
4. **No CRM MCP connected** — tell the user which CRMs you support natively (HubSpot, Salesforce), ask which CRM they use, and ask them to connect the appropriate MCP before continuing. If they can't connect one, offer the CSV fallback and tell them exactly which columns you need: `deal_name, amount, close_date, create_date, stage (won/lost), loss_reason, source`, plus any company/contact association fields they can include.

Tell the user which connector you detected before continuing.

### 0.2 Output destination

Check which output connectors are available:

- **Notion MCP** — look for tools like `notion-create-pages`, `notion-update-page`, `notion-search`. If present, this is the preferred destination.
- **Other documentation MCPs** (Confluence, Google Docs, etc.) — use if present and the user prefers them.
- **Markdown fallback** — always available. Write to the user's working directory.

Ask the user where they want the report. If Notion is available, ask for the parent page or database to create under.

### 0.3 Time range

Ask which window to analyze. Offer: last 30 days, last 90 days, last 12 months, all time (default: all time with period breakdowns).

### 0.4 High-level scope

Ask only what the user already knows up front:
- Any deal types or pipelines they want to **exclude** by name (test deals, internal deals, partner pipeline, etc.)
- Any specific segment they want to focus on (a region, product line, or motion)

**Do not ask about pipeline scope here yet** — the full pipeline list isn't known until Step 1's schema discovery. Pipeline confirmation happens in Step 1.4 once you can show the user what actually exists in their CRM.

### 0.5 ICP / GTM canon grounding

Check the current repo / working directory for existing ICP, persona, scoring, or positioning docs. Look for files matching patterns like `**/icp*`, `**/persona*`, `**/scoring*`, `**/pain*`, `**/positioning*` under `playbooks/`, `docs/`, `frameworks/`, or the project root.

- **If found:** read them. Note the *stated* ICP (target company size, buyer persona, vertical, motion). You will compare this to the CRM reality at the end of the analysis and produce an "ICP Canon vs Reality" headline finding.
- **If not found:** ask the user if they have an ICP document anywhere. If not, skip the comparison and note in the output that grounding was unavailable. **Do not invent an ICP** from general knowledge or hardcoded examples.

### 0.6 Data boundary Q&A

Walk through these questions with the user before pulling anything. This prevents wasted work later.

1. **Which data sources should I use?** (CRM only? CRM + enrichment? CRM + web research?)
2. **Any sources I should NOT use?** (e.g. competitor data, internal Slack, anything behind paywalls)
3. **Am I allowed to call paid enrichment APIs?** (Crustdata, Clearbit, ZoomInfo, etc.) If yes, get explicit per-call approval — do not assume blanket approval. If no, rely on CRM data alone.
4. **Am I allowed to do web research on the top won/lost companies?** (e.g. checking their funding stage, recent news, tech stack)
5. **Are there special segments to analyze separately?** (e.g. a specific product line, a specific region, a partnership channel)
6. **Are there deals I should exclude entirely?** (test deals, internal deals, historical migrated data, specific deal types)

### 0.7 Depth and audience

- **Depth:** quick (executive summary only), standard (full analysis), or deep (full analysis + enrichment + deal stories + strategic recommendations)?
- **Audience:** founder, GTM leader, sales team, or board? Tunes how aggressive the recommendations are and how much context to include.

Do not proceed until the user has confirmed scope.

---

## Step 1 — Data Structure Discovery

Before running the full pull, inspect the CRM schema so you know what's available. Report back to the user before continuing — this surfaces missing fields, custom properties, and naming quirks early.

### 1.1 Deal schema

Inspect the deal object and report:
- Total deal count in the CRM
- All available pipelines and their stage definitions (which stages map to "closed-won" vs "closed-lost")
- Core deal fields: name, amount, close date, create date, stage, owner
- Loss reason field(s): is it a picklist or free text? Sample the distinct values.
- Source / channel / campaign fields
- Deal type, motion, or segment fields (often custom)
- Activity / engagement fields: notes count, contacted count, associated contacts count
- Days-to-close or equivalent velocity field
- Any custom properties that look relevant to win/loss

For HubSpot, use `search_properties(objectType: "DEAL", ...)` with keyword queries. For Salesforce, describe the `Opportunity` object.

### 1.2 Association schema

Check what associated objects exist on deals and the coverage:
- Companies: how many deals have an associated company record?
- Contacts: how many deals have an associated contact record?
- Any custom association types (partners, parent accounts, etc.)

**Flag association coverage gaps up front.** In many CRMs — especially those with a PLG / self-serve signup flow — a large fraction of deals have no associated company or contact record (orphan deals). This biases firmographic analysis toward whichever segment has better association hygiene. If you detect coverage below ~80% on either side, explicitly surface it to the user and note that the final report will include an "Association Coverage" section explaining the bias.

### 1.3 Company and contact schema

Inspect the associated objects you'll query:
- Company: name, domain, industry, employee count, revenue, country, any custom firmographic fields
- Contact: first/last name, job title, seniority, job function, email

Note the coverage rate of `jobtitle`, `numberofemployees`, and `industry` as data-quality flags — these drive the persona and firmographic analysis.

### 1.4 Report back to the user (and confirm pipeline scope)

Summarize what you found in 5–10 lines before running the full pull:
- Total deals + breakdown by closed status
- **Pipelines detected — list each one and ask which to include** (default: all)
- Loss reason field type + top values
- Association coverage estimate
- Any custom fields that will be useful
- Any obvious data-quality issues

Get confirmation before running Step 2.

---

## Step 2 — Pull Deal Data

### HubSpot

Pull all closed deals in paginated batches:

```
search_crm_objects(objectType: "deals", filterGroups: [
  {filters: [{propertyName: "hs_is_closed_won", operator: "EQ", value: "true"}]}
])
search_crm_objects(objectType: "deals", filterGroups: [
  {filters: [{propertyName: "hs_is_closed", operator: "EQ", value: "true"},
             {propertyName: "hs_is_closed_won", operator: "EQ", value: "false"}]}
])
```

Paginate with `limit: 200` and `after` cursors until the `total` count is fully retrieved. Verify the count matches before moving on — truncated pulls are a common silent failure.

Check for deals in **all** pipelines, not just the main one. Filter by pipeline only if the user scoped it down in Step 1.4.

### Salesforce

Query the `Opportunity` object filtering on `IsClosed = true`. Pull `Name, Amount, CloseDate, CreatedDate, StageName, IsWon, LossReason (or custom), LeadSource, Type`, plus any custom fields discovered in Step 1.

### CSV fallback

Read the provided CSV, map columns to the standard fields, warn about any missing columns.

---

## Step 3 — Pull Associations

Pull associated companies and contacts for **all** closed deal IDs (not a sample). Full coverage dramatically improves signal strength.

Batch deal IDs in groups of ~100 to avoid API errors. Run won and lost batches in parallel.

**Companies:** name, domain, industry, employee count, annual revenue, country, any custom firmographic fields.
**Contacts:** first/last name, job title, seniority, job function, email.

Record the coverage rate (associated companies / total closed deals, and same for contacts) — you will need this for the Association Coverage section of the report.

---

## Step 4 — Run the Analysis

> **Note on ordering:** The sections below are listed in *compute order* (the order to calculate them in), not *output order*. The TL;DR and Top 3 Actions are computed last because they synthesize the rest, but they appear at the **top** of the final report. The canonical output layout is in Step 5.

Execute each section below. For each section, compute the metrics, identify the patterns, and note statistical significance.

**Minimum sample sizes:**
- Strong claim: 20+ deals in the subset
- Moderate claim: 10–19 deals
- Weak/directional: 5–9 deals
- Do not make claims from <5 deals — note as "insufficient data"

### Section 0 — Funnel Overview

Establish the full funnel before win/loss analysis:
- Total deals created (all stages)
- Still open
- Closed-won count and % of created
- Closed-lost count and % of created

Report both:
- **Creation-to-close conversion rate:** won / total created
- **Closed-basis win rate:** won / (won + lost)

A "65% win rate" means something very different when only 22% of deals ever close. Always show both.

### Section 1 — Executive Summary

For each period (30d, 90d, 12mo, all time): won count, lost count, revenue won, revenue lost, average deal size, closed-basis win rate, net revenue.

### Section 2 — Revenue Over Time

Monthly breakdown: won #, won $, lost #, lost $, win rate, trend direction. Flag anomalous months (batch close-outs).

### Section 3 — Deal Velocity

- Avg and median days-to-close: won vs lost
- Win rate by velocity bucket: 0–7, 8–14, 15–30, 31–60, 61–90, 90+
- Identify the **stale-deal threshold** — the point where win rate drops below 50%

### Section 4 — Loss Reasons

- Categorized breakdown (normalize free text: "ghosted"→"No Response", competitor mentions, pricing, priority, etc.)
- % with no loss reason (data quality flag — warn if >30%)
- Loss reasons segmented by deal size and source

### Section 5 — Company Signal Patterns

- Employee count distribution won vs lost (buckets: 1–10, 11–50, 51–200, 201–1000, 1000+)
- Industry won vs lost
- Geography (normalize country names)
- Revenue range if available

**Skip any industry, geography, or size row with fewer than 3 total deals** — sample is too small to be meaningful and clutters the table.

Compute **over-index ratio** for each attribute: `(% of wins with attribute) / (% of losses with attribute)`. Ratios >1.5 = win signal, <0.67 = anti-signal.

### Section 6 — Buyer Persona Patterns

Classify contact titles into persona buckets using generic business-title patterns:

```
Founder/CEO: founder, co-founder, ceo, owner, president, managing director
C-Suite (non-CEO): cmo, coo, cfo, cto, chief (not ceo)
VP/Director/Head: vp, vice president, head of, director
Manager/Lead: manager, senior manager, lead, team lead
RevOps/GTM Ops: revops, revenue operations, gtm, sales ops, marketing ops
BDR/SDR: bdr, sdr, sales development
IC/Specialist: analyst, specialist, coordinator, associate
Consultant/Advisor: consultant, advisor, freelance
```

For each bucket: count won vs lost, win rate, flag strong predictors.

**After the bucket table, show grouped title examples.** For each bucket with meaningful volume, list 3–5 representative exact titles from the data. This makes personas concrete without dumping raw title lists.

Example format:
> **Founder/CEO (67 won, 28 lost — 71% WR):** Founder, Co-Founder, CEO, Founder & CEO, Managing Partner

### Section 7 — Activity Signals

Pull activity fields for all closed deals (names vary by CRM): total notes, outreach touches, associated contacts count. Compare averages won vs lost. Total notes often anti-correlate with winning (problem deals accumulate documentation); outreach touches often positively correlate.

### Section 8 — Source & Motion

For each source / motion / deal-type value: won count, lost count, win rate, average deal size.

### Section 9 — Deal Stories

Select 5 best wins and 5 most instructive losses. Pull the full deal record plus associated company and contact. Write 2–3 sentence narratives.

**Best wins:** highest value, fastest close, or strongest ICP fit.
> "[Company], [X emp], [contact title] — closed $[amount] in [days] days via [motion]. [One sentence on what made this work.]"

**Instructive losses:** longest cycle, highest lost value, or clearest anti-pattern.
> "[Company], [X emp], [contact title] — lost after [days] days. [Loss reason]. [One sentence on what this teaches.]"

Deal stories make the analysis memorable and shareable. They turn abstract signals into specific examples.

### Section 10 — Unified Signal Dashboard

Synthesize findings into ONE table with win signals and anti-signals together, ranked by strength, with an action column. Strength tiers:

- **Very Strong win:** >80% WR with 20+ deals
- **Strong win:** >65% WR with 10+ deals
- **Moderate win:** >60% WR with 5+ deals
- **Very Strong anti:** <30% WR with 10+ deals
- **Strong anti:** <40% WR with 5+ deals

Each row: direction (✅/❌), signal, evidence (WR + n), strength, recommended action.

List anti-signals first when the sales team is the audience — they double as routing rules, not just observations.

### Section 11 — ICP Canon vs. Reality

If ICP/persona/scoring files were found in Step 0.5, write a 2–4 sentence headline comparing stated ICP to actual win/loss data. This belongs near the top of the final report, not buried in the appendix — if the stated ICP contradicts the data, that IS the headline.

If no ICP docs were found, skip and note the gap.

### Section 12 — TL;DR

After completing the full analysis, extract the 3–5 most important trends/insights/patterns as a bulleted TL;DR. Place at the very top of the output. This is often the only part leadership reads — make it count.

### Section 13 — Top 3 Recommended Actions

The three specific, actionable changes the team should make on Monday. Ranked by impact. Include file paths or system changes where concrete (e.g. "add a workflow to require company association at deal creation"). Place near the top of the output.

### Section 14 — Association Coverage Explanation

If Step 1.2 detected association coverage below ~80% on either won or lost deals, include a dedicated section explaining:
- The actual coverage rates in a small table
- Why the gap exists (orphan deals from self-serve flows, missing association workflow automations, etc.)
- **Which direction the gap biases the analysis.** In PLG / self-serve businesses the won-side gap is usually larger, which means small-company / founder signals are *understated*. Name this explicitly.
- What to fix in the CRM to close the gap going forward

### Section 15 — Enrichment (Optional)

If enrichment MCP tools are connected AND the user gave explicit approval in Step 0.6, enrich the top 25 won and 25 lost companies with funding stage, headcount growth, tech stack, and recent job postings. Compare won vs lost and add new signals to the dashboard.

**Do not call paid enrichment APIs without explicit approval.** If no enrichment is available or approved, note the gap in Data Quality and move on. The core analysis stands on its own.

---

## Step 5 — Output

### Canonical structure

This is the order things appear in the final report (different from the compute order in Step 4).

```
# Win/Loss Analysis — [Company] — [Date]

Owner / Next Review / Confidence footer

## TL;DR — What The Data Actually Says
[3–5 bullets]

## Strategic Recommendations (Top 3 Actions)
[Ranked, specific, actionable — Monday-morning changes]

## Methodology Snapshot
[5–6 lines: source, deals analyzed, time range, coverage caveat, signal framework]

## Unified Signal Dashboard
[Win + anti-signals in one table with action column]

## Headline Finding — ICP Canon vs Reality
[Callout, if ICP docs found]

## Funnel & Revenue Snapshot
[Both creation-cohort and closed-basis rates]

## Deal Velocity
[Velocity table + stale threshold callout]

## Firmographics
[Employee count, industry, geography with over-index ratios]

## Buyer Personas
[Bucket table + grouped title examples for won and lost]

## Deal Stories
[5 best wins + 5 instructive losses, narrative format]

## Loss Reasons & Activity
[Loss reason breakdown + activity averages]

## Source & Motion
[Source and motion win rates]

## Association Coverage Explanation
[If coverage gap detected — why, which direction it biases, how to fix]

## Data Quality Notes
[Low-coverage fields, anomalous periods, small sample warnings]

## Methodology
[Full footer: deal counts, time range, pipelines analyzed, sample sizes, signal framework, analysis date, version]
```

### Destination

- **If Notion (or other doc MCP) is connected** and the user chose it: create the page under the agreed parent.
- When writing to an existing Notion page, do **not** repeat the page title as an H1 or opening heading in the body. Notion already renders the page title in the page chrome, so start the content with the first real section heading instead. This avoids the "double header" effect.
- **Otherwise:** write a markdown file to the working directory (default: `notes/win-loss-analysis-[date].md`).

---

## Step 6 — Offer Recurring Setup

After delivering the one-shot analysis, ask:

> "Would you like to set this up as a monthly recurring analysis? I can create a scheduled task that re-runs this on the 1st of each month and updates the report with fresh data."

If yes, create a scheduled task that re-runs Steps 2–5 with the same parameters, appends a new dated section to the existing page (or creates a new page per month), and highlights deltas from the previous period.

---

## Guardrails

- **Never modify CRM data.** Read-only.
- **Never auto-send reports externally.** Always present to the user first.
- **Never call paid enrichment APIs without explicit user approval.** Not even once.
- **Never invent an ICP** from general knowledge. Either pull from the repo or leave the comparison out.
- **Flag small-sample claims** (<10 deals) in the report.
- **Normalize country names** (US / USA / United States → single value) before reporting geography.
- **Warn if >30% of losses have no reason recorded** — this is a data quality issue, not an analytical finding.
- **Warn if >50% of company records lack industry** — industry analysis becomes directional only.
- **Check all pipelines** — deals often hide in secondary pipelines.
- **Check association coverage** — orphan deals are the #1 source of silent bias in PLG businesses.
