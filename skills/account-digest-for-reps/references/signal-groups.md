# Signal Groups And Ranking

## Allowed Signal Groups

Use these values in `signals.csv`:

```text
product_signups
product_usage
session_activity
web_intent
identified_visit
form_fill
email_reply
crm_activity
lifecycle
opportunity
hiring
funding
launch
news
web_change
tech_change
stakeholder_change
review_intent
category_intent
competitor_intent
custom_event
custom_object
company_context
other
```

## Ranking Factors

Score accounts using a simple weighted judgment, not a black box:

1. RevOps preference match: does the signal match the user's stated useful patterns?
2. Recency: did it happen inside the approved window, default 30 days?
3. GTM relevance: does it imply budget, ownership change, expansion, urgency, or a better outreach reason?
4. ICP/account fit: is the CRM account actually the matched company/domain?
5. First-party strength: product usage, form fill, reply, session activity, or identified visit beats static context.
6. Provider confidence: structured, dated provider signals beat vague public context.
7. Actionability: can the rep take a specific next step?

## Suppress By Default

- old funding/news with no current trigger
- broad hiring at large public companies with no relevant team/function
- parent-company hiring when the CRM account is a subsidiary, product, or unrelated domain
- raw firmographics, headcount, location, or industry alone
- generic tech stack lists without a sales reason
- rep-authored CRM notes/tasks/calls by themselves
- anonymous visits with no date, contact, or account confidence
- static enrichment that does not create a next move

## Buyer-Authored Vs Rep-Authored

Set `buyer_authored=true` for product events, form fills, replies, identified visits, account news, hiring, funding, launches, web changes, review intent, and other account/market actions.

Set `buyer_authored=false` for rep-created CRM notes, tasks, calls, lifecycle labels, owner changes, or stale static enrichment. These can support ranking but should not surface an account alone.

## Top 10 Rule

Render at most 10 accounts per rep/segment by default. If fewer than 10 accounts have strong signals, render fewer. Do not pad.
