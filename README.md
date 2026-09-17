# gtmotto GTM skills

Free LinkedIn prospecting playbooks for Claude, Claude Code and any agent that reads a
`SKILL.md`. Five of them work with nothing but LinkedIn. All eight gain live workflows
when the [gtmotto](https://gtmotto.com) MCP is connected: gtmotto is self-driving LinkedIn
prospecting. You describe who you want to meet; it finds them daily, warms them up from
your own account, invites them, and opens the conversation. Nothing is said without you.

## Install

Claude Code:

```
/plugin marketplace add gtmotto/gtm-skills
/plugin install gtm-skills@gtmotto
```

That also registers the gtmotto MCP server. The first protected call opens a **Connect**
card; sign in or sign up in the browser, and Claude retries the call.

Or add the server by hand:

```bash
claude mcp add --transport http gtmotto https://agent.gtmotto.com/mcp
```

Scripts and clients without OAuth can use an API key created at
[app.gtmotto.com/connect](https://app.gtmotto.com/connect):

```bash
claude mcp add gtmotto --transport http https://agent.gtmotto.com/mcp \
  --header "Authorization: Bearer gtm_sk_..."
```

In Claude web or desktop, add `https://agent.gtmotto.com/mcp` as a custom connector.
The same Connect card handles authentication.

Any other agent: copy a `skills/<name>/SKILL.md` into wherever your agent reads skills.
They are plain Markdown.

## The skills

Work by hand, and daily with gtmotto:

| Skill | What it does | gtmotto lane |
|---|---|---|
| [find-buyers-in-linkedin-comments](skills/find-buyers-in-linkedin-comments/SKILL.md) | Find posts where your buyers complain, take the people who engaged, keep the ones who fit. Search recipe, scoring rubric, real yields. | `competitor_posts` |
| [icp-to-linkedin-search](skills/icp-to-linkedin-search/SKILL.md) | A sentence or a URL into seven ICP facets, a boolean title string, an exclusion list, and a company count. | `people_search` |
| [hiring-signal-prospecting](skills/hiring-signal-prospecting/SKILL.md) | Which open roles mean "about to buy", how to read the posting, who to contact instead of the recruiter. | `job_offers` |
| [linkedin-limits-and-safety](skills/linkedin-limits-and-safety/SKILL.md) | The daily caps, the 200-character note rule, warm-up order, warning signs, a Monday health check. | seat status |
| [linkedin-comment-and-invite-writer](skills/linkedin-comment-and-invite-writer/SKILL.md) | Comments, invite notes and first messages that get replies. Rules, anti-patterns, examples in English and French. | approvals |

Need the gtmotto MCP:

| Skill | What it does |
|---|---|
| [launch-a-play](skills/launch-a-play/SKILL.md) | From a sentence to an active play: size the ICP, draft, connect LinkedIn, preview every lane, the human's go, the first 48 hours. |
| [instantly-to-linkedin](skills/instantly-to-linkedin/SKILL.md) | Companies that replied or clicked in an Instantly campaign become an `account_list` lane. Email stays in Instantly. Needs both MCPs. |
| [weekly-play-review](skills/weekly-play-review/SKILL.md) | Pipeline over 7 and 30 days, per-lane verdicts, seat health, a diagnosis table, three changes. Read-only until you ask. |

## What the MCP will and will not do

21 tools. Read: who am I, plays, one play in full, stats, LinkedIn seat, sourcing lanes,
run history, the lane reference, and drafted messages awaiting approval. Write: estimate
an ICP, create a play, configure and switch lanes, preview or run a lane, forget harvested
posts, update the ICP, update the play (pause, resume, launch, cap, hours), connect
LinkedIn, and approve or reject one drafted message.

There is no send verb. No invite, message, comment or like can be triggered from the
MCP. A play created from a chat stays a draft until a LinkedIn seat is connected and
the human asks to launch it. The four steps that speak (comment, first message,
follow-up, reply) wait for approval in the cockpit by default.

## Contributing

A skill is one folder under `skills/` with a `SKILL.md`. Frontmatter: `name` and a
`description` written as the sentence someone would type when they need it. Body: what
to do by hand first, then the gtmotto calls. Numbers only when measured. Open a pull
request.

## License

MIT.
