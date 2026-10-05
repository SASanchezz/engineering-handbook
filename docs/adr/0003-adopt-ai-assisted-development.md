# ADR 0003: Adopt AI-assisted development with committed rules, skills, and guardrails

- **Status:** Accepted
- **Date:** 2026-10-05
- **Deciders:** Engineering lead
- **Related:** [AI-assisted development](../../process/ai-assisted-development.md),
  [AI agent guardrails](../../process/ai-agent-guardrails.md),
  [ADR 0001](0001-record-architecture-decisions.md)

## Context

Engineers were already using AI assistants, each in their own way. Personal tools, personal
prompts, and no shared rules led to predictable problems:

- **Inconsistent output.** The same kind of task (a migration, a new endpoint) came out
  differently depending on who prompted it. Assistants did not know the local rules: integer
  minor units, ledger-only balance changes, expand/contract migrations.
- **Data risk.** There was no written rule about what may go into a prompt. Stack traces with
  player identifiers and `.env` contents were one paste away.
- **Review load.** Large, confident-looking diffs arrived without a plan, and reviewers could not
  tell which parts had been thought through.
- **Knowledge stayed private.** Prompts that worked lived in personal chat history.
- **Agents act, not just suggest.** Assistants now run commands, edit files, and call external
  systems through MCP. A prompt alone is not enough to limit what they do.

The platform moves real money, so mistakes in generated code are financial events, not UX bugs
(see [principles](../../principles.md#5-correctness-first-where-money-moves)).

## Decision

Adopt Claude as the team's standard assistant, and keep everything that shapes its behavior in the
repo, reviewed like code.

- **One approved tool set** with a business agreement and no-training terms. Other tools need an
  evaluation before use on company code.
- **Context in the repo:** every service has a short `CLAUDE.md`; it links to this handbook rather
  than copying it.
- **Shared skills** for repeated workflows (`jira-to-plan`, `write-migration`, `review-pr`),
  with eval sets for those on critical paths.
- **Enforcement below the prompt:** committed permission rules, hooks for rules that must never
  be broken, an approved MCP server list, and no production credentials in any agent
  environment.
- **Humans decide and merge:** agents open PRs; people approve plans, review diffs, merge, and
  deploy.
- **Measured at team level:** lead time, change failure rate, and rework on `ai-assisted` PRs,
  never per person.

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| Ban AI tools | No new risk | Unenforceable in practice; pushes use into personal, unmanaged tools; loses real speed on tests, docs, and boilerplate |
| Allow any tool, individual choice | No process overhead | No shared rules, no data agreement, inconsistent output, nothing to measure |
| Standard tool, guidelines in a wiki only | Light to set up | Rules the assistant cannot see and nothing enforces; drifts out of date |
| **Standard tool, rules and guardrails committed to repos (chosen)** | Same rules for every engineer and every agent; changes go through review; enforcement does not depend on memory | Upfront work on `CLAUDE.md`, skills, and hooks; another set of files to keep current |

## Consequences

- **Positive:**
  - Assistants follow local conventions much more consistently, because they see them.
  - The riskiest rules (secrets, ledger, migrations, tests) are enforced by tooling, in line with
    [principle 6](../../principles.md#6-make-the-right-thing-the-easy-thing).
  - Good workflows spread through skills instead of word of mouth.
  - The plan-first workflow keeps PRs reviewable.
- **Negative:**
  - `CLAUDE.md` files, skills, and hooks are code that has to be maintained. Stale rules produce
    stale output.
  - Review has to resist the volume agents can produce; the 400-line limit applies to them too.
  - Spend is a new running cost that needs a budget and an owner.
  - Juniors can skip the struggle that builds fundamentals; the
    [career ladder](../../people/career-ladder.md#working-with-ai) has to account for it.
- **Follow-ups:**
  - Commit `.claude/settings.json` and the core hooks to every service repo.
  - Add eval sets for `write-migration` and `review-pr`.
  - Add AI-assisted rework and spend to the monthly metrics review.
  - Review this decision after one quarter against lead time, change failure rate, and the team
    survey.
