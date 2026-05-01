# Provider Discovery And Mapping

Use this reference after the skill triggers and before extraction.

## Discovery Order

1. Inspect the active tool registry for available connectors/MCP tools.
2. Classify each source as CRM, external signal provider, first-party activity, enrichment, delivery, or file/sheet input.
3. Summarize what was found before asking the user broad provider questions.
4. Ask only about sources that cannot be detected: exports, sheets, internal warehouses, HubSpot custom objects/events, Clerk/session activity, product analytics, or paid providers not exposed as tools.

## User Confirmation Prompt

```text
I found these available sources:
- CRM: ...
- Signal providers: ...
- First-party activity: ...
- Delivery: ...

Do you also have HubSpot custom events/objects, Clerk/session activity, product analytics, or RevOps CSVs/sheets that should be included?
Which signals should be prioritized for this rep/team?
Any accounts or account types to exclude?
```

## Provider To Signal Group Map

| Source | Useful signal groups | Notes |
| --- | --- | --- |
| HubSpot CRM | `crm_activity`, `lifecycle`, `form_fill`, `web_intent`, `product_signups`, `custom_event`, `custom_object` | Separate buyer-authored events from rep-authored notes/tasks. |
| Salesforce | `crm_activity`, `lifecycle`, `opportunity`, `custom_object` | Use reports/list views for account scope and owner mapping. |
| Clerk/session activity | `product_usage`, `product_signups`, `session_activity` | Useful for target-domain user activity, workspace creation, login frequency, and activation. |
| Product analytics / warehouse | `product_usage`, `product_signups`, `session_activity` | Prefer account-level aggregations with dates. |
| Sumble | `hiring`, `company_context` | Best for role/team hiring signals and org enrichment. Avoid broad parent-company hiring. |
| Exa | `news`, `launch`, `funding`, `hiring`, `web_change`, `company_context` | Use for current public web research and source citations. |
| PredictLeads | `hiring`, `funding`, `launch`, `news`, `tech_change`, `web_change` | Good for structured company events. |
| Apollo | `stakeholder_change`, `hiring`, `contact_change`, `enrichment` | Use when exported or connected. |
| ZoomInfo | `stakeholder_change`, `intent`, `enrichment`, `hiring` | Validate dates and source labels. |
| RB2B | `web_intent`, `identified_visit` | Treat anonymous/weak visits carefully; prioritize known account/contact context. |
| Vector | `web_intent`, `identified_visit`, `intent` | Same caution as RB2B. |
| G2 / Capterra | `review_intent`, `category_intent`, `competitor_intent` | Buyer-intent CSVs should include account, date, category/page, and source. |
| Clay / CSV / Sheets | any mapped group | Require provider/source attribution per row. |
| CrustData / AI Ark / SignalBase / Trigify / Jungler / TAMRadar | `funding`, `hiring`, `news`, `intent`, `company_context` | Use only if connected and approved. |

## Provider Rules

- "Connected" is not the same as "used." List a provider as used only if it produced rows.
- Prefer dated, account-specific signals.
- Preserve provider name in `provider` and in Slack source text.
- If a provider can only produce static enrichment, use it for context but do not let it surface an account by itself.
