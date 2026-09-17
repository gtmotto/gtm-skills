---
name: launch-a-play
description: "Launch a gtmotto play from a chat: size the ICP, create the play as a draft, connect the LinkedIn seat, preview every sourcing lane, get the human's go, activate, and explain the first 48 hours. Use when the user says 'set up gtmotto', 'launch a play', 'start prospecting on LinkedIn', 'create a play for <product or URL>', 'find me <persona> at <companies>' with the gtmotto MCP connected, or is onboarding onto gtmotto for the first time. Requires the gtmotto MCP server."
---

# Launch a play

A play is an ICP, up to five sourcing lanes and an autonomy ladder. Once active it finds
people every day, warms them from the user's own LinkedIn account, invites them, and
opens the conversation, with a human gate on anything that speaks. This skill is the
first-run script, from a sentence to an active play, without the user leaving the chat
except to connect LinkedIn.

Three rules that hold for the whole script:

- **Never activate, resume or run a lane unless the user asked for it in this
  conversation.** Activation spends the month's lead quota and acts from their account.
- **Preview before run.** `gtm_preview_source` is free of side effects; `gtm_run_source`
  inserts real leads.
- **Say what happens next, every time.** The first two days are silent by design.

## 0. Where are we

```
gtm_whoami            plan, subscription, products, plays, LEAD METER (used / cap / left)
gtm_list_plays        existing plays, and whether each has a LinkedIn seat
gtm_sources_reference the lane vocabulary, once per session
```

If a play already exists for this product, ask whether to improve it or start another.
Two plays with the same ICP on the same seat compete for the same daily cap.

## 1. Gather four things

Ask only for what is missing; infer the rest from the website.

1. **The product**: a website URL (seeds a product and starts its research) or an
   existing `productId` from the cockpit.
2. **The buyer**: who, where, how big. Use the `icp-to-linkedin-search` skill to turn a
   sentence into the seven facets.
3. **The goal** of a conversation: "a 20-minute call about X", "an intro to the CTO".
4. **The voice**: "direct, French, no emoji". One line.

## 2. Size the ICP

```
gtm_estimate_icp industries, locations, sizes     → "≈ N companies match"
```

Iterate with the user until the number is a few thousand (a daily lane takes up to 25
people; tens of thousands is a market, under a hundred is a list). `null` means the
deployment has no data-provider key; move on.

## 3. Create the play, as a draft

```
gtm_create_play
  name: "<buyer> at <segment>"
  websiteUrl: "https://..."          or productId
  goal, voice
  icp: { titles, seniority, languages, industries, sizes, locations, exclude }
  sources: [
    { key: "people_search",    on: true,  config: { fitGate: "standard" } },
    { key: "job_offers",       on: true,  config: { keywords: "<role>", postedWithinDays: 30 } },
    { key: "competitor_posts", on: true,  config: { intent: "<sentence>", competitors: [...], postGate: 70, intentGate: 70, fitGate: "loose" } },
    { key: "account_list",     on: false }
  ]
  # autonomy: omit. Defaults are visit, like, invite automatic; comment, first_dm,
  #           follow_up, reply wait for a human. Only write rungs the user named.
  # dailyCap: omit. Defaults to the seat's full allowance (145). Pass it only to throttle.
  # activate: omit. Defaults to false.
```

Lanes that need seeds are refused if switched on empty: `competitor_posts` wants an
`intent`, `competitors` or `postUrls`; `account_list` wants companies; `job_offers` wants
`keywords` or ICP titles. `post_engagers` is retired; never configure it.

The call returns the cockpit link. Give it to the user.

## 4. Connect the LinkedIn seat

```
gtm_linkedin_status playId
```

`no_seat` or `seat_broken`: the play cannot act. Connecting is a browser flow. Send the
user to `app.gtmotto.com/app/<playId>` and say "connect LinkedIn there, then tell me
done". Call `gtm_linkedin_status` again when they do; `classic` or `sales_navigator`
means go.

## 5. Preview every lane that is on

```
gtm_preview_source playId, key      per lane
```

For each: the size and its precision (`exact`, `atLeast`, or `unavailable` with why), a
10-person sample, the `unmapped` facets LinkedIn could not filter on (judged per person
instead), who is `alreadyALead`, and for `account_list` which companies resolved.

Tune with the user: a sample full of consultants means add them to `exclude`; a size of
12 means widen a facet; `unavailable` means read `skipped`. Write changes with
`gtm_update_icp` or `gtm_configure_source`, preview again. Two rounds is normal.

If the user wants to see real leads before launch, `gtm_run_source` works on a draft
play, but only when they asked: it spends quota.

## 6. Ask, then activate

Show the summary: ICP in one line, lanes on, daily cap, what runs alone, what waits.
Then ask "Launch it?". On yes:

```
gtm_update_play playId, status: "active"
```

No live seat means it stays a draft whatever you pass; the note says so.

## 7. Explain the next 48 hours

Say this, filled in from `gtm_show_play`:

> It is running. Today gtmotto sources up to 25 people per lane and visits their
> profiles. Around day 2 it reacts to a recent post of each and sends the connection
> request with a note. Nothing that speaks (comments, first messages, follow-ups,
> replies) goes out until you approve it in the cockpit at app.gtmotto.com/app/<playId>.
> Daily cap on your account: <dailyCap> actions, weekdays, <workingHours>. Expect the
> first accepted connections around day 3 and the first drafts waiting for you then.
> Say "how is the play doing" any time; the `weekly-play-review` skill reads the stats.

Silence for two days is the warm-up, not a failure. Say so before it happens.

## If something refuses

| Reply | Meaning | Do |
|---|---|---|
| `blocked: icp_empty` | people_search is on with no ICP | `gtm_update_icp` |
| `blocked: needs_config` | a lane is on with nothing to run on | configure or switch off |
| `no_seat` / `seat_broken` | LinkedIn not connected | cockpit link, step 4 |
| `lead_quota` | the month's leads are used | say so; the plan page is in the cockpit |
| `PLAN_REQUIRED` | no plan on the workspace yet | send them to the cockpit to pick one |
| not found | wrong or foreign id | re-read `gtm_list_plays`, never retry blindly |
