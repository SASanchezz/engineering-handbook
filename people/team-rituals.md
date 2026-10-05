# Team Rituals

The team is remote-first, spread across EU and US time zones. Rituals exist to keep everyone aligned
with as few meetings as possible. Every meeting has a purpose, an owner, an agenda, and written
output; otherwise it becomes a doc or a message.

## Rituals at a glance

| Ritual | Format | Cadence | Length | Output |
| --- | --- | --- | --- | --- |
| Standup | Async, written | Daily | 5 min to write | Thread in team channel |
| Sync check-in | Optional call | 2x per week | 15 min | Blockers resolved |
| Planning | Call with pre-read | Every 2 weeks | 60 min | Sprint goal and committed tickets |
| Backlog refinement | Async + short call | Weekly | 30 min | Tickets ready for planning |
| Demo | Call, recorded | Every 2 weeks | 30 min | Recording + notes |
| Retro | Call with async input | Every 2 weeks | 45 min | 1 to 3 actions with owners |
| On-call review | Call | Monthly | 30 min | Alert fixes, load check (see [on-call](../operations/on-call.md)) |
| 1:1s | Call | Weekly or bi-weekly | 30 min | See [one-on-ones](one-on-ones.md) |

## Async standup

Posted by 11:00 in your local time, in a thread the bot opens each morning.

```text
Yesterday: PAY-412 wagering progress endpoint - PR up, needs review
Today:     PAY-430 duplicate settlement fix; on-call secondary
Blockers:  need a staging payment provider key (asked in #platform)
FYI:       out Friday afternoon
```

Rules:

- Three lines, not an essay. Link tickets and PRs.
- Blockers get a reply within your working day from someone who can help, usually the lead.
- Reading the thread is part of the ritual. Comment on things you can help with.

Why async: a daily call across 7 or more hours of time difference means someone always joins at a
bad hour, and most of the content is status nobody needs to hear live.

## Planning

- Pre-read shared 24 hours before: sprint goal draft, top of the backlog, capacity (on-call,
  holidays, the 20% [tech debt](../process/tech-debt.md) budget).
- In the call: agree on the goal, pull tickets until capacity is reached, call out risks and
  dependencies.
- Tickets entering a sprint have acceptance criteria and a rough size. Unclear tickets go back to
  refinement, not into the sprint.
- Commit to the goal, not to every ticket. Scope moves; the goal should not.

## Demo

- Show working software, in staging or production. No slides unless it is a design.
- Each person demos what they shipped; 3 to 5 minutes each.
- Product and support are invited. It is how they learn what changed.
- Recorded for people in other time zones.

## Retro

- Async input in a shared board during the 2 days before: what went well, what did not, ideas.
- Call: group, vote, discuss the top 2 or 3, agree on actions.
- **1 to 3 actions, each with one owner and a date.** More than three and none get done.
- Start each retro by reviewing the previous retro's actions.
- The facilitator rotates. The lead does not always run it.

Formats rotate every few retros to keep it from going stale: start/stop/continue, timeline of the
sprint, 4Ls (liked, learned, lacked, longed for).

## Docs-first

- **Proposal before meeting.** Any meeting to decide something has a written pre-read. No pre-read,
  no meeting.
- **Decisions get written down** in the ticket, RFC, or ADR. "It was discussed on a call" does not
  count (see [principles](../principles.md)).
- **Public over private.** Ask in the team channel rather than a DM unless it is personal. Someone
  else has the same question.
- **Link, do not paste.** One source of truth, linked from everywhere.

## Time zone etiquette

| Situation | Expectation |
| --- | --- |
| Core overlap | Agree on 2 to 3 hours where everyone is online (e.g., 15:00 to 17:00 CET). Meetings go there |
| Recurring meetings | Inside core overlap. If impossible, alternate times so the same people do not always take the bad slot |
| Messages outside someone's hours | Fine to send; no expectation of a reply until their working day. Use scheduled send for non-urgent DMs |
| Urgent issues | Page through on-call, not a DM at 2 a.m. |
| Response time on chat | Same working day; for PR reviews see [code review](../process/code-review.md) |
| Handoffs | End your day with a short note on anything someone in a later time zone needs to pick up |
| Status | Keep calendar and chat status accurate: working hours, focus time, out of office |

## Focus time

- Protect at least 2 meeting-free half-days per week per person.
- Notifications off during focus time is fine; on-call is the exception.
- If your week has more than 6 hours of meetings as an IC, tell your lead. That is a process bug.

## Meeting hygiene

- Agenda in the invite, or the meeting is declined.
- Start on time, end early if done.
- Someone takes notes; notes are posted within the hour.
- Everyone can decline a meeting where they are not needed, without explanation.

## Reviewing the rituals

Every quarter, one retro is about the rituals themselves: is each one still worth its time? Drop
or change what is not, and update this document.
