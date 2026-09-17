---
name: linkedin-limits-and-safety
description: "LinkedIn account safety for outbound: how many connection requests, messages, profile visits, reactions and comments per day are safe, invitation-note character limits (200 vs 300), pending-invite hygiene, warm-up sequencing, weekday and working-hour pacing, restriction warning signs, and a weekly health check. Use when the user asks 'how many invites per day is safe', 'is my LinkedIn account at risk', 'LinkedIn jail', 'weekly invitation limit', 'restricted', 'connection request limit', 'why was my invite not sent', or wants to pace a LinkedIn outreach account. With the gtmotto MCP connected, reads gtm_linkedin_status and writes the plain-English health report."
---

# LinkedIn limits and safety

Every number below is per LinkedIn account per day, whatever tool sends the action. The
caps are the ones gtmotto enforces as a hard floor under the user's own daily cap; they
were tuned against real LinkedIn throttling and against the provider ceiling of roughly
80 to 100 invites a day, and they are deliberately lower than that.

## The caps

| Action | Per account, per day | Why this number |
|---|---|---|
| Connection requests | 20 | LinkedIn's public guidance is about 100 a week; 20 a day on weekdays lands at 100 with no spikes |
| Direct messages | 40 | replies count; a conversation is many messages |
| Profile visits | 40 | the warm-up step; visits are the least sensitive action |
| Reactions | 30 | second warm-up step |
| Comments | 15 | comments are public and attributable; keep them rare and good |

Sum: 145 actions a day. That is the default daily cap on a gtmotto seat, and the reason
to lower it is a new or thin account, never a crowded one.

Other rules that cost more than the caps when broken:

- **Invitation notes are 200 characters on a free seat, 300 on Premium.** LinkedIn does
  not negotiate: a 242-character note from a free seat is rejected and the invite is never
  sent. Write for 200 unless the seat is known Premium.
- **Weekdays only.** A connection request on a Sunday morning is what gets you reported.
  Working hours matter less (a message read at 08:00 bothers nobody).
- **Warm up before inviting.** Visit the profile on day 0, react or comment on a post two
  days later, then invite. Skip posts older than 10 days; engaging with a two-month-old
  post looks like what it is.
- **Withdraw stale pending invites.** Over 3 weeks pending, withdraw. A pile of ignored
  requests is the strongest "this is automation" signal LinkedIn has.
- **Acceptance rate under 30 percent means stop and fix targeting**, not send more.
- **Never search-and-invite from a fresh account.** Under 3 months old or under 200
  connections: half the caps for the first month.

## Warning signs, in order

1. "You've reached the weekly invitation limit" earlier in the week than usual.
2. Search results that stop paging ("commercial use limit").
3. A CAPTCHA or a "verify your identity" prompt at login.
4. Invitation notes silently dropped (the invite arrives without the text).
5. A temporary restriction. Stop everything for a week, withdraw pending invites, log in
   from the usual device only.

At sign 1 or 2: halve the invite cap for two weeks. At sign 3: pause every tool that
touches the account for 72 hours. At sign 5: do not appeal with "I use a tool".

## Weekly health check

Run every Monday. One line per question, yes or no:

- Invites sent last week under 100? Acceptance rate above 30 percent?
- Pending invites older than 3 weeks withdrawn?
- Any action sent on a weekend?
- Any note over 200 characters on a free seat?
- Any day with visits, reactions or comments at the cap? (Cap days are fine occasionally;
  five in a row is a pattern.)
- Replies to messages within 24 hours? A conversation that is opened by a tool and
  abandoned by a human is the thing that earns the "I'm not interested in your bot"
  screenshot.

## With the gtmotto MCP

```
gtm_list_plays                        which plays share which seat
gtm_linkedin_status playId            seat state + today's usage per kind
gtm_show_play playId                  daily cap, working hours, autonomy ladder
gtm_play_stats playId                 invited vs new connections = acceptance rate
```

Read the seat `state` first:

| state | meaning | what to say |
|---|---|---|
| `no_seat` | nothing connected; the play cannot act | connect LinkedIn in the cockpit at `app.gtmotto.com/app/<playId>` |
| `seat_broken` | disconnected or deleted; the queue is parked | reconnect in the cockpit; nothing was lost |
| `pending` | connection in progress | wait |
| `classic` | connected, standard search backend | fine |
| `sales_navigator` | connected, Sales Navigator search backend, Premium note length | fine |

Then `usage` per kind (`invite`, `dm`, `visit`, `reaction`, `comment`) as used / cap, the
account's `dailyCap` and `seatUsedToday`, and this play's share. The daily cap and
working hours are account-level: every play on the seat shares them.

To throttle, when the user asks:

```
gtm_update_play playId, dailyCap: 70, workingHours: { start: "09:00", end: "18:00", tz: "Europe/Paris" }
```

Never raise the cap, resume a paused play or activate one without the user asking in
this conversation. `paused: true` on `gtm_update_play` is the stop button when a warning
sign appears.

### The report

Write it in plain English, one screen, no JSON:

```
Seat: <account name>, <state>. Today <seatUsedToday>/<dailyCap>.
Invites 12/20, messages 8/40, visits 31/40, reactions 14/30, comments 3/15.
Last 7 days: <invited> invited, <connections> accepted (<rate>%). <verdict>.
Working hours <start> to <end> <tz>, weekdays. Cap: <fine | lower it because ...>.
Plays on this seat: <names>. Steps that speak wait for approval: <comment, first_dm, ...>.
Next check: Monday.
```

Verdict rules: acceptance under 30 percent, say "fix targeting before sending more";
any kind at cap for 5 days running, say "pattern, lower the cap"; `seat_broken`, say
"reconnect, nothing sends until then".
