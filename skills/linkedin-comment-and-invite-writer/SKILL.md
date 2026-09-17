---
name: linkedin-comment-and-invite-writer
description: "Write LinkedIn comments, connection-request notes and first messages that get replies: rules, character limits (200 or 300 for invitation notes), anti-patterns, and examples in English and French. Use when the user asks for a LinkedIn comment, an invite note, a connection request message, a first DM after connecting, a follow-up, wants to rewrite an outreach draft, or asks 'what should I say to this person'. With the gtmotto MCP connected, this is how to draft and review the words that wait for approval in the cockpit, and how to set a play's voice."
---

# LinkedIn comment and invite writer

Three things get written on LinkedIn during outreach, and each has one job:

| Piece | Job | Length |
|---|---|---|
| Comment on their post | be noticed by someone who does not know you | 1 to 3 sentences |
| Connection-request note | give one reason to accept | under 200 characters |
| First message after they accept | start a conversation, not a sale | under 60 words |

None of them pitches. The pitch happens after they answer, if it happens at all.

## The comment

Rules:

1. **Add one specific thing.** A number, a counter-example, a step they skipped, a
   question only they can answer. "Great post" and "so true" are noise.
2. **Disagree politely when you can.** A courteous "I'd push back on one point" gets three
   times the replies of agreement.
3. **No link, no product name, no "we".**
4. **Match the language and the register of the post.** A French rant gets a French
   comment. A post with emoji gets none back; a comment is not a reply.
5. **Under three sentences.** The best comments are one.
6. **Reply to a comment rather than the post** when the thread is long. The author sees
   both; the commenter sees only yours.

Examples:

> Le vrai coût n'est pas les 100 euros, c'est les 4h par semaine à nettoyer la liste
> après. Vous avez mesuré ce temps-là ?

> The 30 percent acceptance rate is the number I'd watch before the reply rate. Below
> that it is a targeting problem, not a copy problem.

## The connection-request note

Rules:

1. **200 characters, hard.** LinkedIn gives a free seat 200 and Premium 300, and a note
   over the limit means the invite is never sent, silently. Count.
2. **Name the reason.** The post, the comment, the hire, the mutual context. Something
   that proves this note was written for them.
3. **No pitch, no ask, no link.** The ask is the invite itself.
4. **One sentence, sometimes two.** First name, the reason, done.
5. **Send it two days after the visit and the reaction**, not the same hour.

Examples (each under 200 characters):

> Pierre, votre post sur Sales Nav m'a fait rire jaune, on fait la même chose à la
> main. Ravi de vous suivre ici.

> Sarah, your comment on the SDR ramp thread was the only one with a number in it.
> Would like to keep reading you.

> Marc, saw you're opening a first SDR role. Been through that stage twice, happy to
> connect.

## The first message

They accepted. Now the mistake is to pitch within the hour. Rules:

1. **Under 60 words.**
2. **Ask about the thing they said**, not about their needs. "How did the spreadsheet
   thing end up?" beats "What are your biggest challenges?".
3. **One question.** Two questions get zero answers.
4. **Nothing to click.** No calendar link, no deck, no case study.
5. **Wait at least a day after the accept.**

Example:

> Merci pour l'accept. Je repensais à votre post : vous avez fini par garder Sales Nav
> ou vous avez trouvé autre chose pour les listes ?

The follow-up, if there is no answer after 5 to 7 days, is one line and gives them an
out: "Pas de souci si ce n'est pas le moment, je voulais juste savoir comment ça s'était
fini."

## Anti-patterns

Rewrite any draft that contains one of these:

- "I hope this message finds you well", "Je me permets de vous contacter"
- "Quick question" followed by a long one
- "I noticed you..." followed by something on their profile (everyone notices)
- the product name, a feature, a pricing word
- "would love to", "jump on a call", "15 minutes"
- more than one question mark
- an exclamation mark in a note to a stranger
- a note that would work for anyone else on the list

## Voice

Before writing, ask for or infer the voice in one line: "direct, French, no emoji, tutoiement
only after they do". Keep it for the whole sequence. A comment in one voice and a note in
another reads as two people, which it often is.

## With the gtmotto MCP

gtmotto warms each lead (visit, like) and invites on its own by default; the four steps
that speak (`comment`, `first_dm`, `follow_up`, `reply`) wait for a human unless the play
says otherwise. Those drafts sit in the cockpit's approvals at
`app.gtmotto.com/app/<playId>`. This skill is how to write or rewrite them.

```
gtm_show_play playId                 goal, voice, the autonomy ladder (what runs alone)
gtm_update_play playId, voice: "direct, French, no emoji, one question per message"
gtm_update_play playId, goal: "a 20-minute call about how they build lists today"
```

`voice` is how every message should sound; `goal` is what a conversation should reach.
Both are read by the drafts. Never switch a speaking step to automatic from here; that is
the user's decision, made in the cockpit.
