# Account Digest For Reps

Portable skill for building Slack-ready account intelligence digests from CRM lists, CRM searches, Salesforce reports, or CSV account lists.

The skill is connector-first and repo-agnostic. It does not depend on Freckle-specific code, local CLIs, environment variables, or a particular CRM setup.

## Contents

- `SKILL.md` - the compact operating guide and reference router.
- `templates/slack_digest.md` - top-level Slack digest template.
- `templates/slack_account_block.md` - per-account Slack block template.
- `templates/slack_digest_zero_surface.md` - zero-surfaced-account template.
- `references/providers.md` - provider discovery and provider-to-signal mapping.
- `references/signal-groups.md` - signal groups, scoring, and suppression rules.
- `references/extraction.md` - normalized `signals.csv` extraction schema.
- `references/hubspot.md` - HubSpot custom events, custom objects, and writeback notes.
- `references/slack-output.md` - Slack formatting rules and full example output.
- `agents/openai.yaml` - optional UI metadata for skill lists and default invocation.

## What It Does

The skill guides an agent through:

1. Resolving an account list from HubSpot, Salesforce, another CRM connector, or CSV.
2. Identifying the rep, team, segment, or owner scope.
3. Asking RevOps what signals matter for the current run.
4. Discovering available signal providers before extraction.
5. Normalizing all signals into a per-run `signals.csv` staging layer.
6. Applying an actionability gate that suppresses CRM noise, broad enterprise hiring, parent-company mismatches, and weak signals.
7. Rendering a concise Slack `mrkdwn` digest with account, CRM link, signal, provider-backed source, date, and suggested action.
8. Holding Slack delivery and CRM note writeback behind explicit user approval.

## Signal Strategy

Before extraction, the agent should ask what signal patterns the RevOps user or team actually finds useful. Those preferences should guide provider queries, ranking, and suppression.

If the user does not provide custom signal preferences, the default useful GTM patterns are:

- hiring in relevant GTM, RevOps, sales ops, marketing ops, growth, partnerships, business development, customer success, support, sales engineering, data, analytics, CRM, Salesforce, HubSpot, or business operations functions
- hiring for a specific role inside a relevant team
- new funding or expansion capital
- product launches, market launches, regional expansion, partner programs, or pricing/packaging changes
- buyer-authored product, web, form, reply, or intent signals from approved sources

Broad hiring volume at very large companies should not surface by default. It is only useful when the role or team is specific enough to create a clear GTM action and the CRM account identity is not a parent-company or domain mismatch.

## Digest Size

Slack digests are capped at the top 10 accounts per rep or segment by default. If more than 10 accounts have qualifying signals, rank by:

- relevance to RevOps signal preferences
- recency
- specificity
- confidence
- actionability

Do not pad the digest. If only a few accounts have genuinely useful recent signals, render only those accounts.

## Output Contract

Signals are staged in a CSV with this header:

```text
account_name,domain,crm,crm_record_id,crm_url,owner,signal_group,summary,source_title,source_url,date,confidence,suggested_action,buyer_authored,provider
```

The Slack digest must be Slack-native `mrkdwn`, emoji-led, and rendered per account. Each surfaced account should include:

- account name with CRM link on the same line
- concise signal summary
- source and date
- provider attribution in the source label, for example `(found with Sumble)`
- concrete suggested action

Example account line:

```text
🏢 *Account:* Atlassian (<https://app.hubspot.com/...|CRM Deal Here>)
```

Example source line:

```text
📍 *Source:* <https://sumble.com/l/job/...|Senior Principal Analyst, Sales Strategy & Operations (found with Sumble)> · *Date:* Apr 25
```

## Requirements

At least one account source is required:

- HubSpot list/search connector
- Salesforce report/list-view connector
- another CRM connector with account identity, owner, domain, and record URL fields
- CSV with account name plus domain or CRM record ID/URL

Useful signal providers include:

- Sumble, Exa, PredictLeads, Trigify, Jungler, TAMRadar, CrustData, AI Ark, or SignalBase for public triggers
- HubSpot Marketing Hub, product analytics, or internal warehouse data for buyer-authored product/form/visit signals
- Apollo, ZoomInfo, Vector, RB2B, G2, or RevOps-owned CSV/sheet exports for supplemental account signals

The skill can run with only CRM data, but it will usually produce an empty digest if the CRM only contains rep-authored activity.

## Safety And Approval Gates

The skill requires explicit approval before:

- extracting from optional signal providers not already approved by the user
- sending a Slack message
- writing CRM notes
- enriching missing account domains externally

It also prevents misleading attribution: a connector is only listed as used if that connector produced rows in the run artifact.
