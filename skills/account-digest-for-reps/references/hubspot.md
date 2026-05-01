# HubSpot CRM, Custom Events, And Custom Objects

Use this reference when HubSpot is the account source or signal store.

## Early Question

After detecting HubSpot, ask:

```text
Are relevant account signals stored in HubSpot properties, notes/tasks, custom events, custom objects, form submissions, web activity, or lists? If yes, which objects/events should I inspect?
```

## Account Source Fields

Common fields:

- company/account name
- domain
- company id or deal id
- company owner / deal owner / `hubspot_owner_id`
- lifecycle stage
- associated deal stage
- record URL

Do not assume company and deal are the same object. If the source is a deal list, preserve the deal link while also resolving the associated company when possible.

## Custom Events

Use HubSpot custom events for buyer-authored or product/account activity such as:

- signup
- workspace created
- invited user
- logged in
- activated feature
- trial started
- pricing viewed
- form submitted
- account identified

Normalize event rows with `signal_group=custom_event`, `product_signups`, `product_usage`, `session_activity`, `form_fill`, or `web_intent` as appropriate.

## Custom Objects

Use custom objects when teams store signal entities such as:

- account signal records
- intent records
- product workspaces
- onboarding records
- trial records
- account research records

Each custom object row must map to a CRM account/company/deal before it can surface in Slack.

## Rep-Authored Activity

Notes, calls, tasks, owner changes, and last-modified timestamps are rep-authored by default. Use them for context or tie-breaking only unless paired with buyer-authored or external-trigger rows.

## Optional Logging

If the user approves CRM writeback, log a note or custom signal object only after Slack QA. Include:

- run date
- account rank
- signal summary
- source/provider
- suggested action
- Slack message link if posted

Never write CRM notes before the user approves exact accounts.
