# AI-Assisted Development

Claude and other LLM tools are part of daily engineering work. This document is the
guideline written while rolling Claude out across a development team: what is allowed, what is
not, how to keep data safe, and how to share what works.

The short version: **AI is a tool you are responsible for, like any other.** It speeds up the
work; it does not move accountability.

## Principles

1. **The author owns the code.** If you open the PR, you can explain every line in it, whoever or
   whatever typed it.
2. **Review is not optional, and not lighter.** AI-written code gets the same
   [code review](code-review.md) as human-written code. Often more, because it looks confident.
3. **No secrets, no customer data in prompts.** Ever. See data rules below.
4. **Share what works.** Good prompts, rules, and skills live in the repo, not in personal chat
   history.
5. **Measure, do not assume.** Track whether it helps, and adjust.

## What is allowed

| Use | Allowed | Notes |
| --- | --- | --- |
| Generating code, tests, migrations | Yes | Reviewed by the author first, then by a peer |
| Explaining unfamiliar code | Yes | Verify claims against the code |
| Writing PR descriptions, ADR drafts, docs | Yes | The author edits for accuracy |
| Debugging with stack traces and logs | Yes | Redact identifiers and PII first |
| Turning a Jira ticket into an implementation plan and code | Yes | Using the shared workflow below |
| Code review assistance | Yes, as a second pass | A human reviewer still approves |
| Pasting production data, customer records, or secrets | **No** | |
| Generating security-critical crypto, auth, or ledger logic unreviewed | **No** | Two human approvals as usual |
| Using tools not on the approved list for company code | **No** | Ask; the tool will be evaluated |
| Letting an agent push to `main` or deploy | **No** | Agents open PRs; humans merge |

## Data and secret hygiene

Treat every prompt as if it may be logged.

**Never include:**

- API keys, tokens, passwords, connection strings, private keys, `.env` contents.
- Customer data: names, emails, phone numbers, wallet IDs tied to people, transaction records,
  KYC documents.
- Production database dumps or query results with real rows.

**Safe to include:**

- Source code of company services (on approved tools with a business agreement and no-training
  terms).
- Schemas, migrations, OpenAPI specs.
- Synthetic or anonymized data.
- Stack traces after redacting identifiers.

**Enforced guardrails:**

- Secret scanning in pre-commit and CI catches leaks in generated code as well as human code.
- `.env*` files and secret directories are listed in the assistant's ignore configuration.
- Agents run with least privilege: no production credentials in their environment, ever.

Permissions, hooks, MCP servers, prompt injection, and unattended agents are covered in
[AI agent guardrails](ai-agent-guardrails.md).

## Repository conventions

Every service repo has a `CLAUDE.md` at the root. It is the assistant's onboarding doc, kept short
and factual.

```markdown
# CLAUDE.md

## Project
Wallet service. NestJS, TypeScript (strict), PostgreSQL.

## Commands
- `npm test` unit tests; `pnpm test:int` integration (needs Docker)
- `npm lint --fix` before committing

## Rules
- Money is integer minor units (`bigint`). Never use floats or `number` for amounts.
- All balance changes go through `LedgerService.post()`. Never update balances directly.
- Every mutating endpoint accepts `Idempotency-Key`. See src/common/idempotency.
- Migrations: expand/contract only. Read docs/postgres-guidelines.md before writing one.
- Errors: throw domain errors; the global filter maps them to RFC 9457 problem details.

## Layout
src/<domain>/{controller,service,dto}, repository optional. snake_case file names.
No cross-module repository imports.
```

Rules for `CLAUDE.md`:

- Keep it under about 100 lines. Link to the handbook for detail rather than copying it.
- Update it in the same PR when a convention changes. Stale rules produce stale code.
- Directory-level `CLAUDE.md` files are fine for modules with special rules (ledger, migrations).
- A rule the assistant keeps breaking does not belong in `CLAUDE.md` any more; turn it into a
  hook or a permission rule (see [where a rule belongs](ai-agent-guardrails.md#where-a-rule-belongs)).

## Shared skills and prompts

Reusable workflows live in the repo (for example under `.claude/skills/` or a shared tooling repo)
and are reviewed like code.

| Skill | What it does |
| --- | --- |
| `jira-to-plan` | Reads a ticket, inspects the relevant modules, writes an implementation plan for the human to approve |
| `implement-plan` | Implements an approved plan in small commits, with tests first |
| `write-migration` | Produces an expand/contract migration pair and the rollout steps |
| `review-pr` | Second-pass review against the team checklist: money, Postgres, NestJS, security |
| `postmortem-draft` | Builds a timeline draft from the incident channel export, for the scribe to edit |

### The Jira-to-code workflow

```text
1. Engineer picks a ticket, runs jira-to-plan.
2. Assistant proposes a plan: files to touch, tests to add, risks, open questions.
3. Engineer edits and approves the plan. This is the key human step.
4. Assistant implements in small commits; engineer reviews each step locally.
5. Engineer opens the PR with the plan in the description, marks AI-assisted.
6. Normal peer review and CI.
```

The plan step is where most of the value and most of the safety come from. Skipping it produces
large, plausible-looking diffs that are expensive to review.

### Testing the skills themselves

A skill is a program whose runtime is a model, and it can regress when the prompt, the model, or
the codebase changes. Skills on critical paths keep a small eval set next to them:

- `write-migration`: schema changes that must come out as expand/contract, plus a few that must
  be refused (a `NOT NULL` column without a default on a hot table).
- `review-pr`: diffs with planted bugs (float money math, missing idempotency key, missing
  `await`) that the review must flag.

Run the evals before merging a change to the skill and when switching models. A skill that stops
catching its planted bugs does not ship.

## Review responsibility

- Mark AI-assisted PRs with the `ai-assisted` label. This is not a warning; it is how impact is measured.
- Reviewers watch for typical failure modes:
  - invented APIs, config keys, or library functions,
  - tests that assert the implementation instead of the requirement,
  - silent changes outside the requested scope,
  - plausible but wrong money math or concurrency handling,
  - copy-pasted patterns that violate a local rule `CLAUDE.md` should have caught.
- If the assistant repeatedly gets a convention wrong, fix `CLAUDE.md` or the skill, not only the
  PR.

## Measuring impact

Review these monthly at team level, never per person, and compare with the period before
rollout:

| Measure | Why |
| --- | --- |
| Lead time from ticket start to production | The outcome that actually matters |
| PR size and time to first review | AI can inflate PR size; watch for it |
| Change failure rate and escaped defects on `ai-assisted` PRs | Quality must not drop |
| Rework: PRs reverted or substantially rewritten within 30 days, `ai-assisted` vs not | Plausible code that does not hold up shows here first |
| Share of PRs labeled `ai-assisted` | Adoption |
| Spend per month against budget | Cost has to stay proportional to the gain |
| Short quarterly survey: where it helps, where it wastes time | Catches what metrics miss |

If a metric gets worse, change the rules or the skills. The goal is a better team, not a
higher adoption number.

## Rollout approach that worked

1. Start with volunteers on low-risk work (tests, docs, internal tools).
2. Write the first `CLAUDE.md` and data rules with them, from real mistakes.
3. Package repeated prompts as skills once three people use the same one.
4. Open it to the whole team with a short session and pairing.
5. Review the guidelines in retro after a month, then quarterly.
