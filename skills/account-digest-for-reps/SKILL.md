---
name: account-digest-for-reps
description: Build connector-first account signal digests for reps or managers from CRM lists, Salesforce reports/list views, CRM searches, HubSpot custom events/objects, product/session activity, or CSVs; rank the best accounts; render Slack-ready summaries; and gate Slack delivery plus CRM writeback behind explicit approval.
---

# Account Digest For Reps

Use this skill to help RevOps, founders, managers, or reps decide which accounts deserve attention now. The default output is a compact Slack digest of the top accounts, with cited signals and suggested next moves.

This skill is portable and connector-first. Do not assume Freckle-specific paths, local CLIs, local env vars, or a fixed CRM.

## Core Contract

- Start by discovering available connectors/tools before asking broad setup questions.
- Ask only for missing inputs after inspecting the account source and installed providers.
- Stage all account signals in a per-run `signals.csv` before rendering.
- Cap Slack output at the top 10 accounts per rep/segment unless the user asks for more.
- Use `🟢 *Signal:*` for normal surfaced signals. Do not use red alert/red-circle markers.
- Slack delivery, CRM note writeback, external domain enrichment, and optional provider extraction require explicit approval.
- Never claim a provider was used unless it produced signal rows.

## Reference Router

Load only what the run needs:

- Provider discovery and mapping: `references/providers.md`
- Signal groups, scoring, and suppression rules: `references/signal-groups.md`
- Extraction schema and CSV staging logic: `references/extraction.md`
- HubSpot CRM, custom events, and custom objects: `references/hubspot.md`
- Slack templates, formatting rules, and full example output: `references/slack-output.md`

Templates live in `templates/`:

- `templates/slack_digest.md`
- `templates/slack_account_block.md`
- `templates/slack_digest_zero_surface.md`

## Fast Workflow

1. **Discover tools first.**
   Inspect available connectors/MCP tools for CRM, Slack, HubSpot/Salesforce, signal providers, product analytics, sheets/CSVs, and search. Then summarize detected providers and likely signal groups. See `references/providers.md`.

2. **Resolve account source.**
   Accept HubSpot list/search, Salesforce report/list view, connected CRM search, CSV, or another CRM payload. Require account name plus domain or CRM record URL/id. Infer owner fields before asking the user.

3. **Resolve run scope.**
   Determine whether this is for one rep, selected reps, team/territory/segment, or all owners in the source. For multi-rep runs, render one digest per rep/segment unless the user asks for a manager summary.

4. **Ask only missing questions.**
   After inspection, ask for unresolved items: signal preferences, excluded accounts, missing owner mappings, unavailable providers, Slack destination, and whether HubSpot custom events/objects or product/session activity should be checked.

5. **Extract concrete signal rows.**
   Normalize each usable signal into `signals.csv` using the schema in `references/extraction.md`. Each row needs account, source, provider, date, confidence, signal group, buyer-authored status, and suggested action.

6. **Score and gate.**
   Rank by RevOps preference, recency, GTM relevance, ICP fit, first-party activity, confidence, and actionability. Suppress static firmographics, rep-authored CRM activity alone, weak anonymous activity, and parent-company/domain mismatches.

7. **Render preview.**
   Use the Slack templates and `references/slack-output.md`. Render per-account blocks only from `signals.csv`; do not invent missing signals.

8. **Deliver only after approval.**
   Confirm exact Slack destination and post only after user approval. CRM note writeback requires a second explicit approval and account list confirmation.

## Initial Discovery Checklist

Before asking the user which providers exist, check for:

- CRM: HubSpot, Salesforce, or other CRM tools.
- Signals: Sumble, Exa, PredictLeads, Apollo, ZoomInfo, RB2B, Vector, G2, Clay, CrustData, Trigify, Jungler, TAMRadar, AI Ark, SignalBase.
- First-party activity: HubSpot Marketing Hub, HubSpot custom events, HubSpot custom objects, Clerk/session activity, product analytics, internal warehouse exports, CSV/sheets.
- Delivery: Slack send/draft tools.

Then ask:

```text
I found these usable sources: ...
Do you also have signals stored in HubSpot custom events/objects, product analytics, Clerk/session activity, or a RevOps CSV/sheet that should be included?
Any accounts, segments, competitors, customers, or parent-company false matches to exclude?
```

## Account Shape

```json
{
  "name": "Example Co",
  "domain": "example.com",
  "owner": "Rep Name or owner id",
  "crm": "hubspot",
  "crm_record_id": "123",
  "crm_url": "https://app.hubspot.com/contacts/PORTAL/record/0-2/123"
}
```

## Signal Shape

```json
{
  "account_name": "Example Co",
  "domain": "example.com",
  "signal_group": "hiring",
  "summary": "Hiring for RevOps and outbound operations roles.",
  "source_title": "RevOps Manager role (found with Sumble)",
  "source_url": "https://example.com/jobs/revops",
  "date": "2026-04-27",
  "confidence": "high",
  "suggested_action": "Ask whether outbound operations tooling is part of the new RevOps mandate.",
  "buyer_authored": true,
  "provider": "sumble"
}
```

## Default Signal Preferences

If the user has no custom preference, prioritize:

- hiring in GTM, RevOps, sales ops, marketing ops, growth, partnerships, customer success, support, sales engineering, data, analytics, CRM, Salesforce, HubSpot, or business operations
- funding, market expansion, product launches, partner programs, pricing/packaging changes
- first-party activity: product usage, signups, form fills, replies, identified web visits, Clerk/session activity
- CRM/account context only when paired with buyer-authored or external-trigger evidence

## Output Rules

- Top 10 accounts by default; do not pad weak accounts.
- Every Slack account block needs account name, CRM link, signal, source/provider, date, and suggested action.
- Source labels must include provider attribution, e.g. `(found with Sumble)`.
- Use short dates like `Apr 27` in Slack; keep ISO dates in CSV.
- Report blocked/missing-domain/owner-review records outside the Slack digest unless the user asks to include them.

## Optional Repo Adapter

If the current repo has a documented implementation, use it only when the user intends to run that repo-specific path and outputs still conform to this skill. In the Freckle GTM repo, historical commands live behind `python3 -m outbound_ops.cli account-digest-*`; treat them as a local adapter, not the portable default.
