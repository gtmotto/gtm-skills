---
name: icp-to-linkedin-search
description: "Turn a sentence, a website URL or a rough persona into a structured ICP (titles, seniority, languages, industries, headcount, locations, exclusions) and a LinkedIn or Sales Navigator search: normalized title variants, boolean strings, exclusion list, and a market-size estimate. Use when the user describes who they sell to, asks for a LinkedIn search, a Sales Nav boolean, a people search, wants to size an audience, normalize job titles (CMO + Chief Marketing Officer + VP Marketing), or exclude noise (fractional, freelance, intern). With the gtmotto MCP connected, sizes the ICP with gtm_estimate_icp and writes it to a play's people_search lane."
---

# ICP to LinkedIn search

Input: "CFOs at Swiss fintechs", a website URL, or a paragraph about the buyer.
Output: an ICP card in seven facets, a boolean title string, an exclusion list, and, when
gtmotto is connected, a company count for the audience and a `people_search` lane that
runs every day.

## Step 1. Extract the seven facets

Write the ICP as seven plain-word arrays. Only name the facets the user gave you or that
follow from the product; leave the rest empty. An empty facet is "any", not "unknown".

| Facet | What goes in | Example |
|---|---|---|
| `titles` | every wording of the role, current job only | CFO, Chief Financial Officer, VP Finance, Head of Finance, Finance Director |
| `seniority` | owner, cxo, vp, director, manager, senior, entry | cxo, vp, director |
| `languages` | the language the buyer posts in | French, English |
| `industries` | LinkedIn industry names, plain words are fine | Financial Services, Fintech, Banking |
| `sizes` | headcount bands | 11-50, 51-200, 201-500 |
| `locations` | countries, regions, metro areas | Switzerland, Geneva |
| `exclude` | never contact, absolute | fractional, freelance, interim, intern, student, recruiter, agency, consultant |

Headcount bands: `1-10`, `11-50`, `51-200`, `201-500`, `501-1000`, `1001-5000`,
`5001-10000`, `10001+`. "SMB" is 11 to 200, "mid-market" 201 to 1000, "enterprise" 1001+.

From a website URL, read the pricing page and the customer logos first: the pricing page
tells you the size band, the logos tell you the industry, the docs or the integrations page
tell you the buyer's function.

## Step 2. Normalize titles

A title is a family, not a string. For each role the user names, list:

- the acronym and the long form (`CMO`, `Chief Marketing Officer`)
- the VP, Head, Director and Lead variants when the company size makes them the same
  person (at 11 to 50 employees, the "Head of Marketing" is the CMO)
- the local-language forms for each `languages` entry (`Directeur Marketing`,
  `Responsable Marketing`)
- the adjacent function that owns the same budget (RevOps for a sales tool, Growth for a
  marketing tool)

Then the exclusions. Always add to `exclude`: `fractional`, `freelance`, `interim`,
`intern`, `assistant to`, `student`, `former`, `ex-`, `retired`, `looking for`. Add
`recruiter`, `agency`, `consultant` and `advisor` unless those are the buyer.

## Step 3. Write the boolean string

For the current-title field of LinkedIn or Sales Navigator:

```
("CFO" OR "Chief Financial Officer" OR "VP Finance" OR "Head of Finance" OR "Finance Director")
NOT (fractional OR interim OR freelance OR intern OR assistant)
```

Rules: quotes around multi-word titles, `OR` inside a family, `NOT` for the exclusions,
parentheses around each group, at most 15 operators per field (split into two searches
past that). Keywords are a separate field: use them for the chore or the tool ("outbound",
"HubSpot"), never for the title.

Give the user the string, the facets it came from, and the two or three things LinkedIn
cannot filter on (a language, "does their own prospecting", a tech stack). Those get
judged per person, not per search.

## Step 4. Size it

Before anyone builds a play or a list, size the audience. A daily lane takes up to 25 new
people per run, so a few thousand matching companies is plenty; a few hundred is a
campaign, not a play; tens of thousands means the ICP is a market, not a buyer.

With the gtmotto MCP:

```
gtm_estimate_icp industries: [...], locations: [...], sizes: [...]
  → "≈ 1,240 companies match" (writes nothing)
```

It takes only the three company facets (titles do not change the company count). Iterate:
40 companies, widen the locations; 30,000, add an industry or narrow the size band. A
`null` estimate means the data provider has no key on that deployment, not an error.

## Step 5. Write it to a play (gtmotto MCP)

```
gtm_whoami                                   plan + leads left this month
gtm_list_plays                               existing play, or:
gtm_create_play name, websiteUrl | productId, icp: {titles, seniority, languages,
                industries, sizes, locations, exclude},
                sources: [{ key: "people_search", on: true,
                            config: { keywords: "outbound", fitGate: "standard" } }]
                                             activate defaults to false: it stays a draft
gtm_update_icp playId, <facets>              on an existing play; only named facets change
gtm_preview_source playId, key: "people_search"
                                             size, 10-person sample, the unmapped facets
```

What to know:

- gtmotto compiles the facets into the LinkedIn query itself and normalizes odd labels
  ("Swiss" to Switzerland, "Fintech" to Financial Services). Plain words are fine.
- `keywords` on `people_search` are extra free-text terms on top of the ICP. Alone they
  are a complete search; the lane runs on an empty ICP if keywords are set.
- The facets LinkedIn cannot filter on come back as `unmapped` in the preview and are
  judged per person by the fit gate. `standard` needs one match, `strict` needs all of
  them, `loose` is role and seniority only. `exclude` is absolute at every level.
- The first `gtm_update_icp` on a play forks its own ICP row; no other play's targeting
  changes.
- `gtm_estimate_icp` and `gtm_preview_source` need a write-scoped key but write nothing.

Never activate the play from this skill. Hand the user the cockpit link the create call
returns and let them press go, or follow the `launch-a-play` skill.

Connect `https://agent.gtmotto.com/mcp` with no credentials. The first protected call
opens a Connect card for browser sign-in. Scripts without OAuth can create an API key at
`app.gtmotto.com/connect` and send it as `Authorization: Bearer gtm_sk_...`.
