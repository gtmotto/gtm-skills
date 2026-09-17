---
name: hiring-signal-prospecting
description: "Use job postings as a buying signal: which open roles mean a company is about to buy what you sell, how to read a job description for budget and pain, who to contact at the hiring company (not the recruiter), and how to open. Use when the user mentions job posts, job offers, hiring signals, 'companies hiring for X', open roles, headcount growth, job-based intent, or wants to prospect companies that are recruiting. With the gtmotto MCP connected, configures the job_offers sourcing lane so it runs daily."
---

# Hiring-signal prospecting

A job posting is a budget line someone already approved, with the pain written in the
job description. It is the one intent signal that names the problem, the timing and the
owner in the same document. This skill turns "companies hiring for X" into buyers inside
those companies, by hand or as a daily gtmotto lane.

## Step 1. Pick the roles that mean "about to buy"

The role you search for is the one whose existence proves the problem. It is rarely the
buyer.

| They are hiring | It usually means | So the buyer is |
|---|---|---|
| first SDR / BDR | they are starting outbound and have no tooling | the founder or the Head of Sales |
| 2nd to 5th SDR | outbound works, they scale it, they will consolidate tools | Head of Sales, RevOps |
| first Head of Sales | the founder is handing off sales | the founder (before the hire lands) |
| RevOps / Sales Ops | stack consolidation, data, attribution | the RevOps hire's boss, the VP Sales |
| Growth marketer / Demand gen | pipeline pressure from the board | CMO, founder |
| Content marketer | they will need distribution for what they write | Head of Marketing |
| Customer Success Manager | churn or expansion is the quarter's problem | VP CS, COO |
| Security / compliance engineer | a deal is blocked on a questionnaire | CTO, CISO |
| Data engineer, first | they cannot answer a question the board asked | CTO, Head of Data |

Write your own row for the user's product: "what role does a company post the week before
they would buy this?"

## Step 2. Read the posting

Three lines to extract from every job description:

1. **The pain sentence.** "You will build our outbound motion from scratch", "own our
   list-building process", "replace our current spreadsheet workflow". It is the opener.
2. **The stack.** Tools named in the requirements are the tools they use or are leaving.
3. **The boss.** "Reports to the VP Sales" names the buyer. When missing, the buyer is the
   most senior person in that function on LinkedIn.

Freshness matters more than volume. A posting under 30 days old is a live budget; over 90
days it is either filled or frozen. Reposts every two weeks mean they are struggling to
hire, which is its own opener ("while you look for the SDR, here is what the role would
do on day one").

## Step 3. Find the buyer, not the recruiter

Never contact the recruiter or the talent partner listed on the post. Go to the company
page, people tab, filter by the buyer title from the table. At companies under 200 people
the buyer is often the founder; above that it is the function head named in "reports to".

One buyer per company per signal. Two people at the same company receiving the same
"saw you're hiring an SDR" note in the same week is a screenshot on LinkedIn.

## Step 4. Open on the hire, not on the tool

The note references the posting and says one true thing about what the role will face:

> Saw you're opening a first SDR role. The first 90 days usually go into list building
> before anyone sends an email. Happy to share what we see work at that stage if useful.

No pitch, no link, no "our platform". The signal is the reason to talk; the product comes
after they answer. See `linkedin-comment-and-invite-writer` for the note and DM rules.

## Make it daily (gtmotto MCP)

The lane is `job_offers`: companies posting a role, then the buyers inside them, matched
against the play's ICP. Read `gtm_sources_reference` once, then:

```
gtm_whoami                                   plan + leads left this month
gtm_list_plays                               pick the play, or gtm_create_play (draft)
gtm_update_icp playId, titles: ["Head of Sales", "VP Sales", "Founder", "CEO"],
               sizes: ["11-50", "51-200"], locations: [...]
                                             the ICP is who to contact INSIDE the hiring company
gtm_configure_source
  playId, key: "job_offers",
  config: { keywords: "SDR", postedWithinDays: 30, fitGate: "standard" },
  on: true
gtm_preview_source playId, key: "job_offers" free: size + 10-person sample
```

Field notes:

- `keywords` is the role being hired ("Head of Sales", "SDR"). It falls back to the
  ICP's `titles` when empty, which is usually wrong: the role you search for is not the
  person you contact. Set it.
- `postedWithinDays: null` means any age. 30 is the default worth proposing.
- The play's ICP decides who inside the company is a buyer. Set `titles` and `seniority`
  to the buyer row from the table, not to the hired role.
- `fitGate: "standard"` is right here: firmographics matter (a 5,000-person company
  hiring an SDR is not the same signal as a 30-person one).

Cadence: once per 24 hours, up to 25 new people per run, on an active play.
`gtm_run_source` runs it now and spends real quota; only when the user asked. gtmotto
then warms each buyer (visit, like, comment), invites with a note, and waits for a human
before anything is said.

Connect `https://agent.gtmotto.com/mcp` with no credentials. The first protected call
opens a Connect card for browser sign-in. Scripts without OAuth can create an API key at
`app.gtmotto.com/connect` and send it as `Authorization: Bearer gtm_sk_...`.
