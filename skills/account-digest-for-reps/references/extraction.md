# Extraction And Staging

All connectors and files must normalize into `signals.csv` before rendering. The Slack renderer reads only staged rows plus run metadata.

## Path

```text
outputs/account-digests/<list-slug>/run-<YYYY-MM-DD>/signals.csv
```

## Required Header

```text
account_name,domain,crm,crm_record_id,crm_url,owner,signal_group,summary,source_title,source_url,date,confidence,suggested_action,buyer_authored,provider
```

## Field Rules

- `account_name`: CRM/display account name.
- `domain`: normalized company domain when known.
- `crm`: `hubspot`, `salesforce`, `csv`, or other CRM/source.
- `crm_record_id`: source system record id when known.
- `crm_url`: direct CRM account/deal/company URL when available.
- `owner`: rep/owner name, email, or CRM owner id.
- `signal_group`: one allowed value from `signal-groups.md`.
- `summary`: one concise sentence explaining the signal.
- `source_title`: human-readable source label; include provider in Slack source text.
- `source_url`: direct source URL when possible. Use CRM URL if CRM is the source.
- `date`: ISO `YYYY-MM-DD` of event occurrence, not fetch date.
- `confidence`: `low`, `medium`, or `high`.
- `suggested_action`: concrete rep next move.
- `buyer_authored`: `true` or `false`.
- `provider`: connector/source name such as `hubspot`, `salesforce`, `sumble`, `exa`, `predictleads`, `clerk`, `apollo`, `csv`.

## Extraction Logic

For every account:

1. Normalize account name, domain, owner, CRM id, and CRM URL.
2. Query approved providers only.
3. Convert each usable event into one row.
4. Deduplicate on `(account_name, domain, signal_group, summary, source_url, date)`.
5. Drop rows with no date unless they are explicitly durable context approved by the user.
6. Attach suggested action before Slack rendering.
7. Mark blocked accounts separately; do not include them in Slack.

## Suggested Action Pattern

Good actions name the rep move and the reason:

```text
Reach out to the RevOps owner about tooling for the new outbound operations team.
```

Avoid generic actions:

```text
Follow up.
Check in.
Mention the signal.
```

## Run Metadata

Track:

```json
{
  "run_scope": "single_rep | selected_reps | team | all_source_owners",
  "source_label": "HubSpot active early pipeline",
  "window_label": "last 30 days",
  "providers_detected": [],
  "providers_used": [],
  "revops_signal_preferences": [],
  "excluded_accounts": [],
  "blocked_accounts": []
}
```
