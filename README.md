# Engineering Handbook

How to build, ship, and run backend platforms, and how to run the team that does it.

This is the handbook for leading a backend team: the default starting points, the rules the team
holds itself to, and the templates to fill in. It comes out of building platforms from zero to
production, including the backend of a real-money betting platform (wallets, bonus and wagering
logic, referral systems), moving a team from environment-named branches to trunk-based development
with immutable artifact promotion, and rolling out AI-assisted development across a team.

It is opinionated on purpose. A handbook that hedges every rule is a handbook nobody follows.

## Who this is for

- **Engineers joining the team.** Read this before your first PR. It answers most "how is X
  done here" questions.
- **Hiring managers and peers.** This is the most honest answer to "how would you run
  an engineering team?"
- **Other leads.** Fork it, delete what does not fit, keep what does.

## Principles in one screen

The full version lives in [principles.md](principles.md). The short version:

1. **Ship small, ship often.** Small changes are cheap to review, cheap to roll back, and cheap to
   reason about.
2. **You build it, you run it.** The team that writes the code carries the pager for it.
3. **Write it down.** Decisions, runbooks, and context live in the repo, not in someone's head or
   a chat thread.
4. **Boring technology by default.** Novelty is a budget, not a goal.
5. **Correctness over cleverness where money moves.** Idempotency, double-entry, and audit trails
   are not optional.
6. **Automate the second time.** Do it by hand once, script it the second time, put it in CI the
   third.
7. **Blameless, not accountable-less.** Fix systems, not people, and still own outcomes.

## Stack context

Most examples assume the stack below. The principles transfer; the examples do not
need to.

| Area | Default |
| --- | --- |
| Language / runtime | TypeScript on Node.js (LTS) |
| Framework | NestJS |
| Database | PostgreSQL |
| Packaging | Docker images in GHCR |
| CI/CD | Jenkins pipelines, trunk-based, artifact promotion |
| Work tracking | Jira |
| AI tooling | Claude, with shared rules and skills committed to repos |
| Working model | Fully remote, EU/US time zones, async-first |

## How to use this handbook

- **Defaults, not laws.** Every rule here can be broken with a written reason in the PR or an
  [ADR](templates/adr-template.md). Breaking it silently is the problem.
- **Change it by PR.** The handbook follows its own [code review](process/code-review.md) rules.
  Use the [improvement issue template](.github/ISSUE_TEMPLATE/improvement.md) if you want to
  discuss first.
- **Link, do not repeat.** When a team repo needs a rule, it links here instead of copying it.

### Adopting it in your team

1. Fork the repo and read [principles.md](principles.md) with the team. Strike anything you do
   not actually believe.
2. Adopt the process docs first: [code review](process/code-review.md),
   [definition of done](process/definition-of-done.md), and
   [branching and releases](process/branching-and-releases.md). They pay off fastest.
3. Copy [templates/](templates/) and [.github/](.github/) into your service repos.
4. Start an ADR log with [0001](docs/adr/0001-record-architecture-decisions.md).
5. Revisit the handbook in a retro every quarter. Delete rules nobody follows; that is cheaper
   than pretending.

## Index

| Area | Document | What it covers |
| --- | --- | --- |
| Core | [principles.md](principles.md) | Values and the trade-offs they imply |
| Process | [code-review.md](process/code-review.md) | Author and reviewer rules, SLAs, PR size, comment prefixes |
| Process | [branching-and-releases.md](process/branching-and-releases.md) | Trunk-based development, versioning, promotion, rollback |
| Process | [definition-of-done.md](process/definition-of-done.md) | When a ticket is actually done |
| Process | [rfc-process.md](process/rfc-process.md) | RFC vs ADR, lifecycle, who decides |
| Process | [tech-debt.md](process/tech-debt.md) | Tracking, prioritizing, and budgeting debt |
| Process | [testing-strategy.md](process/testing-strategy.md) | Test pyramid for NestJS and Postgres services |
| Process | [ai-assisted-development.md](process/ai-assisted-development.md) | Rules for Claude and other LLMs in the workflow |
| Process | [ai-agent-guardrails.md](process/ai-agent-guardrails.md) | Agent permissions, hooks, MCP servers, prompt injection, unattended agents |
| Operations | [incident-response.md](operations/incident-response.md) | Severity levels, roles, comms cadence |
| Operations | [on-call.md](operations/on-call.md) | Rotation, handoff, load, alert hygiene |
| Operations | [observability.md](operations/observability.md) | Logs, metrics, traces, SLOs, alerting |
| Operations | [security-baseline.md](operations/security-baseline.md) | Secrets, dependencies, API security, money handling |
| Backend | [api-design.md](backend/api-design.md) | REST conventions, errors, pagination, idempotency, versioning |
| Backend | [postgres-guidelines.md](backend/postgres-guidelines.md) | Safe migrations, locking, transactions, money types |
| Backend | [nestjs-conventions.md](backend/nestjs-conventions.md) | Layout, module boundaries, DTOs, config, errors |
| People | [onboarding.md](people/onboarding.md) | First day, first week, 30/60/90 |
| People | [hiring-process.md](people/hiring-process.md) | Interview loop, scorecards, candidate experience |
| People | [career-ladder.md](people/career-ladder.md) | Levels from Junior to Senior and Lead |
| People | [one-on-ones.md](people/one-on-ones.md) | Cadence, structure, what they are for |
| People | [team-rituals.md](people/team-rituals.md) | Remote-first rituals and time zone etiquette |
| Templates | [adr-template.md](templates/adr-template.md) | Architecture decision record |
| Templates | [rfc-template.md](templates/rfc-template.md) | Request for comments |
| Templates | [postmortem-template.md](templates/postmortem-template.md) | Blameless postmortem |
| Templates | [pull_request_template.md](templates/pull_request_template.md) | PR description |
| Templates | [incident-update-template.md](templates/incident-update-template.md) | Status updates during incidents |
| Templates | [onboarding-checklist.md](templates/onboarding-checklist.md) | Copyable onboarding tracker |
| Decisions | [0001-record-architecture-decisions.md](docs/adr/0001-record-architecture-decisions.md) | Why ADRs are kept |
| Decisions | [0002-trunk-based-development.md](docs/adr/0002-trunk-based-development.md) | Why the team moved off environment branches |
| Decisions | [0003-adopt-ai-assisted-development.md](docs/adr/0003-adopt-ai-assisted-development.md) | Why AI rules and guardrails are committed to repos |

## Repository layout

```text
.
├── principles.md
├── process/        how work moves from idea to production
├── operations/     how production stays healthy
├── backend/        technical conventions for services
├── people/         hiring, growth, and working together
├── templates/      copy-paste starting points
├── docs/adr/       decisions about this handbook and its defaults
└── .github/        PR and issue templates
```

## License

Text is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
Reuse, adapt, and share it, including commercially. Attribution is appreciated.

## Author

Oleksandr (Alexander) Hrachov, Engineering Lead / Senior Backend Engineer.
Node.js, NestJS, TypeScript, PostgreSQL. Builds platforms from 0 to 1 and the teams that run them.

[LinkedIn](https://www.linkedin.com/in/TODO)
