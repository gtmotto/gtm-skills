---
name: instantly-to-linkedin
description: "Add a LinkedIn channel to an Instantly cold-email campaign from the same chat: pull the companies that opened, clicked or replied in Instantly and hand them to gtmotto as an account_list lane, so the buyers inside get visited, warmed and invited on LinkedIn. Use when the user runs Instantly (or another cold-email tool) and wants LinkedIn next to it, says 'add LinkedIn to my campaign', 'multichannel', 'the people who opened my emails', 'engaged leads to LinkedIn', or wants to sync cold-email engagement to LinkedIn outreach. Requires the Instantly MCP and the gtmotto MCP connected together."
---

# Instantly to LinkedIn

You run email from Claude. This adds LinkedIn from the same chat: the companies that
showed interest in an Instantly campaign become an `account_list` lane in gtmotto, which
finds the buyers inside each company and warms, invites and opens conversations from the
user's own LinkedIn account. Email stays in Instantly; nothing is duplicated.

Needs both servers connected:

- Instantly MCP: `https://mcp.instantly.ai/mcp` (free with an Instantly subscription that
  has API access).
- gtmotto MCP: `https://agent.gtmotto.com/mcp`; the first protected call opens a Connect
  card for browser sign-in. Scripts without OAuth can create a key at
  `app.gtmotto.com/connect`.

One honest limit up front: gtmotto's `account_list` lane takes **companies** (up to 100),
not people. It finds the right buyers inside each company from the play's ICP. A
person-level sync (the exact lead who opened) is not what this does.

## Step 1. Pick the campaign and the signal

```
gtm_whoami                                  plan + leads left this month (sizes step 3)
Instantly: list campaigns                   (tool names vary by server build:
                                             list_campaigns, get_campaign, list_leads,
                                             list_leads_in_campaign, get_campaign_analytics)
```

Ask which campaign, and which signal counts as interest. From strongest to weakest:

1. replied, and the reply was not "unsubscribe" or an auto-reply
2. clicked
3. opened more than once
4. opened once

Default: replies and clicks. Opens alone produce a list of email scanners.

## Step 2. Build the company list

```
Instantly: list leads in the campaign, filtered on the signal
```

Then, per lead: take the domain from the email address (`jane@acme.io` gives `acme.io`),
drop free-mail domains (gmail, outlook, yahoo, icloud, proton), drop the user's own
domain, dedupe. Rank companies by the strongest signal any of their leads showed, then
by how many leads engaged.

Prefer the **domain** over the company name in the list: gtmotto resolves each line to a
LinkedIn company page by title match, and a domain is unambiguous where "Acme" is not.

Cap is 100 per lane. Over 100: keep the top 100 by signal and tell the user how many
were cut, or split by segment into two plays with different ICPs.

## Step 3. Hand it to gtmotto

```
gtm_list_plays                              an existing play for this product, or:
gtm_create_play name: "Instantly <campaign> on LinkedIn", websiteUrl | productId,
                icp: { titles: [<the buyer titles the campaign targeted>], seniority: [...] }
                                            stays a draft
gtm_configure_source
  playId, key: "account_list",
  config: { companies: ["acme.io", "beta-corp.com", ...], fitGate: "standard" },
  on: true
gtm_preview_source playId, key: "account_list"
```

The ICP decides who inside each company is a buyer. Use the titles the email campaign was
written for; the seniority band matters more than the industry here (the company is
already chosen).

The preview returns `accounts[]` with `found: true | false` per line and LinkedIn's own
title for each match. A `found: false` line is a spelling or ambiguity problem: ask the
user for the company's LinkedIn page URL and re-send that line. Editing the list forgets
the misses and keeps the hits.

Then the seat and the launch follow the `launch-a-play` skill: `gtm_linkedin_status`, the
cockpit link if `no_seat`, the user's explicit go, `gtm_update_play status: "active"`.
Never activate without the ask.

## Step 4. Keep it fed

Weekly, or when the user asks:

```
Instantly: leads engaged since <last run>
gtm_configure_source playId, key: "account_list",
  config: { companies: [<new domains>] }, listMode: "append"
```

`append` dedupes case-insensitively against what is there. The lane is due again
immediately after any edit and runs on the next tick (every 5 minutes on an active play),
otherwise once per 24 hours, up to 25 new people per run. `gtm_show_sources` shows
`addedToday` and the last run's funnel per lane.

## What to say about timing

Email replies arrive in hours; LinkedIn does not. gtmotto visits day 0, reacts and
invites around day 2, and the first accepted connections land around day 3. The first
message drafts wait for approval in the cockpit. Tell the user this before the silent
days, not after.

## Do not

- Do not put people (LinkedIn profile URLs) in `companies`. The line will not resolve.
- Do not mirror the whole campaign. 2,000 leads is 300 companies is three plays of noise;
  the signal is the point.
- Do not pause Instantly. The two channels compound; the email gives the LinkedIn note
  its reason ("following up on my note about X").
