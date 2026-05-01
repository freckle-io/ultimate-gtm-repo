# Slack Output

Use the templates in `../templates/`. Do not hand-roll a different format unless the user asks.

## Formatting Rules

- Use Slack `mrkdwn` single asterisks for bold labels.
- Use `🟢 *Signal:*` for normal surfaced signals.
- Render divider lines as inline code: `` `-----` ``.
- Put CRM record links on the account line.
- Link sources with Slack syntax: `<url|source title>`.
- Include provider in source label, e.g. `(found with Sumble)`.
- Include short dates like `Apr 27`.
- Include suggested action under each signal.
- Omit `why_it_matters` by default.

## Template Inputs

- `rep_or_segment_name`
- `source_label`
- `window_label`
- `connectors_used`
- `surfaced_count`
- `suppressed_count`
- rendered `account_blocks`

## Full Example Output

```text
📣 *Signal Digest — Aaron Trent*

🗂️ *Source:* CRM

📊 *Accounts tracked:* 50

✅ *Accounts with signals:* 10

🔌 *Providers used:* HubSpot CRM, PredictLeads, Sumble, Apollo


`-----`


🏢 *Account:* Northstar Kinetic Labs (<https://fake.com/hubspot/deals/demo-1001|HubSpot Deal>)

🟢 *Signal:* Sumble found a hiring spike in RevOps and outbound ops, with 4 new roles in 9 days. PredictLeads also detected homepage copy changing from "book a demo" to "see the platform in action."

📍 *Source:* <https://fake.com/sumble/jobs/demo-1001|Sumble hiring feed (found with Sumble)> · <https://fake.com/predictleads/web/demo-1001|PredictLeads web change (found with PredictLeads)> · *Date:* Apr 27

➡️ *Suggested action:* Reach out to Rachel Zhang, RevOps primary contact, about tooling for her new outbound operations team.


`-----`


🏢 *Account:* Bramble Forge Systems (<https://fake.com/hubspot/deals/demo-1002|HubSpot Deal>)

🟢 *Signal:* PredictLeads detected a new customer logo strip on the pricing page, while Apollo enrichment shows new operations and security stakeholders added this month.

📍 *Source:* <https://fake.com/predictleads/site/demo-1002|PredictLeads site diff (found with PredictLeads)> · <https://fake.com/apollo/accounts/demo-1002|Apollo enrichment (found with Apollo)> · *Date:* Apr 27

➡️ *Suggested action:* Reach out to Marcus Lee, launch operations lead, with a rollout-readiness note tied to the new customer logo expansion.
```
