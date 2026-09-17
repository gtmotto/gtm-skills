---
name: find-buyers-in-linkedin-comments
description: "Find B2B buyers in the comments and reactions of LinkedIn posts where people complain about a problem your product solves. Use when the user wants to find prospects on LinkedIn from posts, comments, reactions or engagers, wants intent signals from LinkedIn, asks 'who is complaining about X on LinkedIn', wants to mine a competitor's posts or a viral post for leads, or mentions post discovery, social listening for sales, engager scraping or comment mining. Works by hand (search recipe + scoring rubric) and, when the gtmotto MCP is connected, as a daily competitor_posts sourcing lane."
---

# Find buyers in LinkedIn comments

A person who writes "I pay 100 a month for Sales Navigator and still build my lists by hand"
under a post has told you three things a people search never will: they have the problem
today, they own it, and they said it in public. This skill finds those posts every day,
takes the people who engaged, and keeps the ones who fit.

Two modes:

1. **By hand.** Search recipe, scoring rubric, who to take. Works with nothing but LinkedIn.
2. **Daily, with gtmotto.** The same recipe as a `competitor_posts` lane ("Post discovery"
   in the cockpit) that runs every day and warms the people it finds. Needs the gtmotto
   MCP (`https://agent.gtmotto.com/mcp`). See "Make it daily" below.

## What you need from the user

- **The intent**, in their own words and language: "people struggling with manual LinkedIn
  prospecting", "founders doing their own sales who hate cold email". One sentence.
- **Anchors**: 5 to 15 short words for tools, brands and chores the buyer would name.
  `Sales Nav`, `Waalaxy`, `Lemlist`, `prospects froids`, `liste de prospects`.
- **Who is a buyer and who is not**: "anyone doing their own B2B prospecting; exclude
  vendors, agencies, consultants, recruiters".

If they give you a post URL instead, skip the search and start at "Score the post".

## Step 1. Write the searches (short, many, in every language)

LinkedIn post search is an AND over every word. A natural pain phrase like
"tired of prospecting by hand" finds almost nothing. What works, measured on a real day:

| Query style | Queries | Posts found in one day | Target post found |
|---|---|---|---|
| Pain phrases, 4 to 6 words | 16 | 18 | no |
| Topic keywords, 2 to 3 words | 20 | 638 | yes, ranked first by engagement |

Rules:

- 2 to 3 words per query. Tools, brands, chores. `Sales Nav`, not `Sales Navigator` (the
  short form found the target post, the full name did not: never rely on one query).
- 15 to 20 queries. Write them in each language the buyer speaks.
- Sort by **date**, filter **past 24 hours** (or since your last pass). Relevance sort
  surfaces old high-engagement posts you already saw.
- Recall comes from the searches. Precision comes from the next two steps, never from
  a longer query.

## Step 2. Filter on engagement first (it is free)

Median reactions on a raw result page is 0 to 4. Rank every found post by
`reactions + 2 × comments` and keep the ones above 20. On a real day 638 posts became 45,
and 11 of those had over 100 reactions. Nobody engages with a post nobody saw, and the
people you want are in the engagement, not in the post.

## Step 3. Score the post

Read the post body and put it in one of four buckets:

| Bucket | Keep? | Tell |
|---|---|---|
| Rant or ask from someone who has the problem | yes | first person, present tense, a number, a question |
| Vendor pitch or launch | no | "we built", "excited to announce", a link to a product |
| Job post or hiring | no | "we're hiring", "join us" |
| Thought leadership about the market | maybe | third person, no personal stake; take only the commenters who disagree or ask |

Score 0 to 100 against the intent. Keep 70 and above. The bucket matters more than the
number: a vendor pitch with 400 reactions is 400 people who like the vendor, not buyers.

## Step 4. Take the right people

From each kept post, in this order:

1. **Commenters.** They wrote something. Read the comment: "same here, we do it in a
   spreadsheet" is intent; "great post!" is nothing. Score each comment 0 to 100 against
   the intent, keep 70 and above.
2. **Reactors.** Silent, so no intent score. Keep them only if they fit the buyer
   description; expect 85 percent to be 2nd-degree connections.
3. **The author.** Off by default because the author of a post about your market is as
   often a vendor as a buyer. Read the profile: a founder venting about their own sales
   process is the best lead on the thread, and free.

Then apply the buyer description **before** any model scoring, as hard exclusions:
vendor, agency, consultant, recruiter, SDR at a tool company. Models rate an outbound
consultant 80 on "fit" if you let them.

What a real post yields: 96 reactions and 80 comments gave 150 unique people. Against a
strict people-search ICP (Head of Sales, 11 to 200 employees, SaaS, France): 1 fit.
Against the buyer description above: 22 fit, 5 with intent above 70, plus the author.
Read engagers with a loose fit, not the ICP you use for search.

## Step 5. What to do with them

Do not pitch in the thread. Visit the profile, react to the post, reply to their comment
with one specific sentence (see the `linkedin-comment-and-invite-writer` skill), and
invite two days later with a note that names the thread. The conversation opens itself.

## Worked example

Post: "Help, I pay 100 a month for Sales Navigator and still build my lists by hand"
(French, founder, 96 reactions, 80 comments). Found by the query `Sales Nav`, ranked first
on engagement, scored 92 against "people struggling with manual LinkedIn prospecting".
Yield after gates: 22 fitting people, 5 with explicit intent, plus the author.

## Make it daily (gtmotto MCP)

The lane is `competitor_posts`. Read `gtm_sources_reference` once, then:

```
gtm_whoami                                   plan + leads left this month
gtm_list_plays                               pick the play, or gtm_create_play (draft)
gtm_configure_source
  playId, key: "competitor_posts",
  config: {
    intent: "<the sentence>",
    competitors: ["Sales Nav", "Waalaxy", "prospects froids", ...],   # max 25
    postUrls: ["https://www.linkedin.com/posts/..."],                  # optional, max 25
    minReactions: 25,
    postGate: 70,
    commenters: true, reactors: true, authors: false,
    intentGate: 70,
    fitGate: "loose"
  },
  on: true
gtm_preview_source playId, key             free: size + 10-person sample, nothing inserted
```

Field notes:

- `intent` does three jobs: the query writer writes the daily searches from it, the post
  scorer judges each post against it, the comment scorer judges each engager against it.
- `competitors` are the anchors. They are the fallback searches when no model can write
  queries. Short words, every language.
- `postUrls` are posts the user named. Harvested as they are, no post gate (they picked
  them).
- `postGate: null` means rank by engagement only. `0` is not off; it still demands a
  verdict.
- `fitGate: "loose"` is the right setting for engager lanes: role and seniority only,
  intent counts more than firmographics. The ICP's `exclude` list is absolute at every
  level.
- The lane remembers every post it already paid to harvest. After changing a gate, call
  `gtm_forget_harvested_posts` once (it costs a fresh engager fetch per post) so the next
  run re-judges today's posts. Only when the user asked to re-test.

Cadence: an ON lane on an ACTIVE play runs once per 24 hours, up to 25 new people per run.
`gtm_run_source` runs it now, spends real quota, and works on a draft play; call it only
when the user asked for a run in this conversation. Nothing here invites or messages
anyone: gtmotto warms each person (visit, like, comment), invites, and waits for a human
before anything is said.

Connect `https://agent.gtmotto.com/mcp` with no credentials. The first protected call
opens a Connect card for browser sign-in. Scripts without OAuth can create an API key at
`app.gtmotto.com/connect` and send it as `Authorization: Bearer gtm_sk_...`.
