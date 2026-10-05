# Career Ladder

This ladder describes what is expected at each level so that growth, feedback, and promotions are
based on shared, written criteria instead of who is most visible. It is a guide for conversations,
not a checklist to tick.

## How to read it

- Levels are cumulative: each one assumes the previous.
- You are performing at a level when you consistently meet most of it over several months, not
  when you did it once.
- Senior is a career level. Nobody is expected to move past it. Lead is a different job, not a
  reward: it leads mainly through people and delivery while keeping enough technical depth.

## Levels at a glance

| Level | Typical scope | Short description |
| --- | --- | --- |
| Junior Engineer | A task | Learns the codebase and delivers well-defined tasks with guidance |
| Middle Engineer | A feature | Delivers features independently within an existing design |
| Senior Engineer | A system or domain | Owns a domain end to end, designs solutions, raises the team's bar |
| Engineering Lead | A team and its systems | Responsible for team outcomes, people growth, and delivery |

## Dimensions

### Scope and impact

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Completes well-scoped tasks; impact within a ticket | Delivers features end to end; impact within a module | Owns a domain such as wallets or bonuses; impact on the team's roadmap | Impact through the team's total output and health |

### Technical skill

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Writes correct, tested code with review; learns the team's conventions ([NestJS](../backend/nestjs-conventions.md), [Postgres](../backend/postgres-guidelines.md)) | Writes clean, well-tested code with little rework; debugs production issues in their area | Designs systems with clear trade-offs; anticipates failure modes (concurrency, idempotency, migrations); reviews for design, not only correctness | Keeps enough technical depth to make sound calls and spot bad ones; ensures design quality through people and process |

### Ownership and operations

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Follows through on assigned work; asks for help early | Owns features in production: metrics, alerts, fixes; takes on-call | Owns a domain's reliability; drives postmortem actions; reduces toil proactively | Owns delivery and operational health of the team; makes trade-offs between features, debt, and reliability explicit |

### Communication

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Gives clear status; asks good questions | Writes clear PR descriptions and tickets; explains decisions in their area | Writes RFCs and ADRs others can act on; adapts message for product and business | Communicates context, priorities, and decisions up, down, and across; represents the team to the business |

### Leadership and influence

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Contributes to team rituals; gives honest feedback in retros | Reviews others' code helpfully; helps onboard new joiners | Mentors engineers; leads projects with 2 to 4 people; improves team process | Hires, grows, and retains the team; runs [1:1s](one-on-ones.md) and performance conversations; builds a team that works without them |

### Working with AI

Measured by judgment and outcomes, never by how much AI someone uses.

| Junior | Middle | Senior | Lead |
| --- | --- | --- | --- |
| Uses assistants within the [guidelines](../process/ai-assisted-development.md); can explain every line of their PRs; still builds core skills by hand (debugging, SQL, reading code) | Writes clear plans and test cases before delegating; catches invented APIs and implementation-mirroring tests in their own output; keeps PRs small when an agent writes them | Knows which work to delegate and which to keep (money paths, migrations, concurrency); turns repeated mistakes into `CLAUDE.md` rules, hooks, or skills; reviews agent-written PRs for the failures that look plausible | Sets the team's AI rules, [guardrails](../process/ai-agent-guardrails.md), and budget; measures impact on lead time and quality, not adoption; makes sure juniors still learn the fundamentals |

## What separates the levels

The most common gaps at each transition:

| Transition | Usually missing |
| --- | --- |
| Junior to Middle | Independence: finishing work without needing step-by-step direction |
| Middle to Senior | Ownership beyond your own tickets: thinking about the system, operations, and other people's work |
| Senior to Lead | Getting results through others, and being willing to spend less time coding |

## Examples of senior-level evidence

To make this concrete, things that would count as strong evidence for Senior:

- Designed the double-entry ledger for a wallet service, wrote the RFC, and led its rollout with a
  backfill and no downtime.
- Found that bonus wagering progress could be double-counted on settlement retries, fixed it with
  idempotent consumers, and added integration tests that cover it.
- Led a migration from environment branches to trunk-based delivery and taught the team the new
  release flow.
- Turned a recurring incident class into an alert, a runbook, and a kill switch.
- Noticed agents kept writing balance updates outside the ledger, replaced the `CLAUDE.md` rule
  with a blocking hook, and added a `review-pr` eval case that catches it.

## Promotions

- Promotion recognizes sustained performance at the next level for about 3 to 6 months.
- The manager and the engineer agree on a growth plan against this ladder; progress is reviewed
  quarterly in 1:1s.
- A written case with concrete evidence per dimension goes to a calibration with other leads.
- Nobody should be surprised by the outcome. If they are, the manager failed at feedback earlier.

## What the ladder is not

- Not a way to rank people against each other.
- Not a measure of output volume. Lines of code, PR counts, and AI usage are not evidence; what
  the work achieved is.
- Not a requirement to be equally strong in every dimension. Spiky profiles are normal; look at
  the overall picture.
- Not fixed. It gets revised when it stops matching the work, through a PR like the rest of this
  handbook.
