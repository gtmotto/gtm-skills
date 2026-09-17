---
name: weekly-play-review
description: "One-page weekly review of a gtmotto play: pipeline over 7 and 30 days with deltas, per-lane sourcing verdicts, LinkedIn seat health, a diagnosis table from symptom to cause to fix, and three changes to make. Use when the user asks 'how is the play doing', 'weekly review', 'what should I change', 'why no leads', 'why no replies', 'is it working', wants a report on a LinkedIn play, or on a Monday with the gtmotto MCP connected. Read-only unless the user asks for a change."
---

# Weekly play review

Replaces the campaign dashboard habit. Five reads, one diagnosis table, one page.
Nothing is written unless the user asks for a specific change at the end.

## The reads

```
gtm_whoami                       lead meter: used / cap / left this month
gtm_list_plays                   every play, status, seat, pipeline spread
per active play:
  gtm_play_stats playId          sourced → warmed → invited → connections → conversations,
                                 all time / 30d / 7d / 24h, each with the delta vs the window before
  gtm_show_sources playId        play state, per lane: on, blocked, last run funnel, addedToday, failures
  gtm_linkedin_status playId     seat state, today's usage per kind
  gtm_source_runs playId, key    only for a lane that looks off (10 runs, newest first)
```

"Conversations started" counts real replies from the other side, never the openers
gtmotto sent.

## Read the play state first

`gtm_show_sources` returns one verdict for the play; it explains an empty week before any
number does.

| state | what it means | fix |
|---|---|---|
| `paused` | the user pressed stop | ask if they want to resume; never resume on your own |
| `draft` | never launched | `launch-a-play` |
| `no_seat` / `seat_broken` | LinkedIn not connected; nothing can act | cockpit link |
| `icp_empty` | people_search on, no ICP | `gtm_update_icp` |
| `needs_config` | a lane is on with nothing to run on | configure or switch it off |
| `lead_quota` | the month's leads are spent | say so; more leads is a plan question |
| `never_ran` | active but the tick has not reached it | wait a day |
| `no_results` | the queries return nobody | widen the ICP or the keywords |
| `provider_error` | LinkedIn or the provider refused | `gtm_source_runs` for the status; often rate limiting |
| `added_nothing` | everyone found was already a lead | the lane is exhausted; new anchors, new list, or off |
| `ok` | sourcing works | look at the funnel below |

## Diagnose the funnel

Read the five numbers over 7 days against the 7 days before. One symptom at a time.

| Symptom | Likely cause | Check | Fix (only if asked) |
|---|---|---|---|
| sourced 0 | lane blocked, quota, provider | `gtm_show_sources` state, `gtm_source_runs` | see the table above |
| sourced fine, warmed 0 | seat broken or cap at 0 | `gtm_linkedin_status` | reconnect; `gtm_update_play dailyCap` |
| scanned high, added low | fit gate or exclusions too tight, or all duplicates | `lastRun.dropped` vs `duplicates` | loosen `fitGate`, or the lane is exhausted |
| dropped by intent high | post lane's `intentGate` too high for the intent wording | `gtm_source_runs` on competitor_posts | lower the gate or rewrite `intent`, then `gtm_forget_harvested_posts` |
| invited fine, connections under 30% | targeting, or the note | sample from `gtm_preview_source` | fix ICP `exclude`; rewrite the note with `linkedin-comment-and-invite-writer` |
| connections fine, conversations 0 | first messages waiting in approvals, or the DM asks for a call | `gtm_show_play` autonomy; the cockpit approvals | approve the drafts; rewrite `voice` and `goal` |
| everything down together | daily cap hit early, or a weekend week | `gtm_linkedin_status` usage | nothing, or raise the cap only if asked |
| one lane adds, others add 0 | the good lane eats the daily budget | `addedToday` per lane | switch the empty lanes off; budget goes to the ones that work |

A lane whose last three runs say `added_nothing` with `duplicates` high is done: the
audience is harvested. Say so instead of proposing a tweak.

## Write the page

```
<Play name> (<product>), week of <date>. Active, seat <state>, <used>/<cap> leads this month.

Pipeline, last 7 days (vs the week before)
  sourced <n> (<delta>) · warmed <n> (<delta>) · invited <n> (<delta>)
  new connections <n> (<delta>, <acceptance rate>%) · conversations <n> (<delta>)

Lanes
  people_search    ON   +<n> this week  <verdict in five words>
  competitor_posts ON   +<n>            <verdict>
  job_offers       off
  account_list     ON   +<n>            <k> companies not found on LinkedIn

Seat: <state>. Invites <used>/<cap> today; nothing at cap for 5 days running.

Three changes, in order
  1. <one line, the tool call that does it>
  2. ...
  3. ...

Leave alone: <what is working and why touching it would cost a week>.
```

Then stop and ask which changes to make. Apply only the ones named, with
`gtm_configure_source`, `gtm_update_icp` or `gtm_update_play`. Never resume a paused play,
raise the cap, activate, or run a lane unless that exact thing was asked. After a change
to a post lane's gates, offer `gtm_forget_harvested_posts` once so the next run re-judges
today's posts; it costs a fresh engager fetch per post, so it is the user's call.
