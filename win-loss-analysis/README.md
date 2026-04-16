# Win/Loss Analysis Skill

A Claude Code / Claude Agent skill that runs a comprehensive closed-won vs closed-lost analysis on your CRM and produces an actionable report — TL;DR, ICP canon vs reality finding, unified signal dashboard, deal stories, and a strategic recommendations block.

Built for B2B sales teams who want to know **why** they're winning and losing, not just the win rate number.

---

## What it does

Given access to your CRM, the agent will:

1. **Discover** your CRM connector, output destination, time range, and ICP grounding docs
2. **Inspect the schema** before pulling — surfaces missing fields, custom properties, and association coverage gaps
3. **Pull all closed deals** plus associated companies and contacts (full coverage, not a sample)
4. **Analyze** funnel, velocity, loss reasons, firmographics, buyer personas, activity signals, source, and motion
5. **Synthesize** into a unified signal dashboard with ✅ win signals and ❌ anti-signals in one ranked table
6. **Compare** the data to your stated ICP (if found in the repo) — produces a "Canon vs Reality" headline
7. **Tell deal stories** — 5 best wins and 5 instructive losses as narrative
8. **Output** to Notion (if connected) or a markdown file in your working directory

See [`examples/sample-win-loss-report.md`](examples/sample-win-loss-report.md) for a full reference output.

---

## Prerequisites

**Required:**
- A CRM with closed deal data (HubSpot, Salesforce, or a CSV export)
- At least 30 closed deals (more = better statistical significance)

**Recommended:**
- **HubSpot MCP** or **Salesforce MCP** connected — the skill detects automatically and uses native tool calls
- **Notion MCP** connected — for nicer output (falls back to markdown otherwise)
- An ICP / persona / scoring document somewhere in your repo (the skill will find and use it for the "Canon vs Reality" finding)

**Optional:**
- Enrichment MCP (Crustdata, Clearbit, etc.) — only used with explicit per-call approval
- Web research access — only used with explicit user approval

---

## Installation

### As a Claude Code skill

1. Drop the `win-loss-analysis/` folder into your skills directory:
   - Project-level: `.claude/skills/win-loss-analysis/`
   - User-level: `~/.claude/skills/win-loss-analysis/`
2. Restart Claude Code or reload skills.
3. The skill auto-triggers when you ask for "win/loss analysis," "deal pattern analysis," "why are we losing deals," etc. You can also invoke it explicitly.

### As an Agent SDK skill

Mount the folder into the skills directory of your Claude Agent SDK setup. The `SKILL.md` frontmatter handles discovery.

---

## How to use it

Just ask. Examples:

> "Run a win/loss analysis on the last 12 months of HubSpot deals."

> "Why are we losing deals? Look at the CRM and tell me."

> "Compare our stated ICP to what's actually closing in HubSpot."

The agent will walk you through Step 0 (discovery & scoping) before pulling anything. Don't skip those questions — they make the rest of the analysis 10x sharper.

---

## What the output looks like

The report is structured for skim-readability. Top of the page:

1. **TL;DR** — 3–5 bullets with the headline trends (the only thing some people will read)
2. **Strategic Recommendations** — top 3 Monday-morning actions
3. **Methodology Snapshot** — 5–6 lines on what was analyzed
4. **Unified Signal Dashboard** — ✅ win signals + ❌ anti-signals in one table with an action column
5. **Headline Finding** — ICP canon vs reality (if ICP docs were found)

Then the supporting detail: funnel, velocity, firmographics, personas, deal stories, source/motion, association coverage explanation, data quality notes, full methodology footer.

See [`examples/sample-win-loss-report.md`](examples/sample-win-loss-report.md) for the full layout.

---

## Key design choices

- **CRM-agnostic.** HubSpot and Salesforce are first-class; CSV is a fallback. The analytical framework doesn't care which CRM you use.
- **Schema discovery before pulling.** Step 1 inspects the deal object, association coverage, and custom fields, then reports back to the user before any heavy queries run. This catches data issues early instead of mid-analysis.
- **Association coverage is treated as a first-class concern.** PLG / self-serve businesses often have "orphan deals" with no associated company or contact — this biases firmographic analysis. The skill detects and explicitly explains the bias direction in a dedicated section.
- **No invented ICPs.** The skill only compares against an ICP if it finds one in the repo or the user provides one. It will not hallucinate an ICP from general knowledge.
- **No paid API calls without approval.** Enrichment is fully optional. The core analysis stands on its own with CRM data.
- **TL;DR and Top 3 Actions are computed last but presented first.** The report is structured for the executive who reads only the top.

---

## Guardrails

- Never modifies CRM data (read-only)
- Never auto-sends reports externally
- Never calls paid enrichment APIs without per-call approval
- Never invents an ICP
- Flags small-sample claims (<10 deals)
- Warns when loss reason coverage is <70% or industry coverage is <50%
- Always checks all pipelines, not just the main one

---

## File structure

```
win-loss-analysis/
├── SKILL.md                          # The skill itself (with frontmatter)
├── README.md                         # This file
└── examples/
    └── sample-win-loss-report.md     # Reference output (anonymized)
```

---

## Contributing / customizing

The skill is one Markdown file. To customize:

- **Title classification** — edit the persona buckets in Section 6
- **Velocity buckets** — edit Section 3
- **Signal strength tiers** — edit Section 10
- **Output structure** — edit Step 5
- **Guardrails** — edit the bottom of `SKILL.md`

If you want to add domain-specific signals (e.g. funding stage, tech stack, hiring intent), wire them into Section 15 (Enrichment) and add the matching rows to the unified dashboard.

---

## License

MIT. Use it, fork it, ship it.
