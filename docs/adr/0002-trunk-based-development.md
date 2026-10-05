# ADR 0002: Adopt trunk-based development with immutable artifact promotion

- **Status:** Accepted
- **Date:** 2026-01-26
- **Deciders:** Engineering lead
- **Related:** [Branching and releases](../../process/branching-and-releases.md),
  [ADR 0001](0001-record-architecture-decisions.md)

## Context

Services used environment-named branches: `develop` deployed to dev, `staging` to staging,
`production` to production. Promoting a change meant merging one branch into the next, and each
merge triggered a new build in Jenkins.

This caused recurring problems:

- **The artifact tested was not the artifact shipped.** Each environment got its own build. A
  dependency resolved differently or a merge picked up an extra commit, and staging no longer
  predicted production.
- **Branch drift.** Hotfixes applied to `production` were sometimes not merged back to `develop`,
  and later releases reintroduced bugs.
- **Painful merges.** Long-lived branches diverged; merge conflicts landed on whoever happened to
  promote that week.
- **No clear answer to "what is running where?"** It required comparing three branches.
- **Slow, batched releases.** Because promotion was manual and risky, changes piled up, which made
  each release riskier still.

CI/CD runs on Jenkins, and Docker images are published to GHCR. The team was comfortable with feature
flags in principle but had used them rarely.

## Decision

Adopt trunk-based development with a single long-lived branch, `main`, and promote one
immutable Docker image through all environments.

- Short-lived feature branches (under 2 days), squash-merged to `main` after review and green CI.
- Unfinished work is merged behind feature flags, off by default.
- Every merge to `main` builds **one** image and pushes it to GHCR tagged
  `v<next-version>-dev.<shortsha>`, for example `v1.14.0-dev.3f9c2ab`. It deploys to dev
  automatically.
- The same image (verified by digest) is promoted to staging and then production. Promotion
  changes tags and deployment targets, never the bytes.
- A release adds a clean SemVer tag (`v1.14.0`) to the tested commit and to the existing image
  digest. Production deploys only clean SemVer tags.
- Environment-specific configuration comes from the environment at runtime, not from the build.
- Rollback means redeploying the previous release tag. Database migrations are kept backward
  compatible to make this safe ([Postgres guidelines](../../backend/postgres-guidelines.md)).

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| Keep environment branches | No change needed; familiar | All the problems above |
| GitFlow (`develop`, `release/*`, `hotfix/*`) | Well known; explicit release branches | Still long-lived branches and merge overhead; builds per branch; designed for versioned products, not continuously deployed services |
| Trunk-based, rebuild per environment | Simpler pipeline change | Keeps the "tested is not shipped" problem |
| **Trunk-based with immutable artifact promotion (chosen)** | One build, traceable versions, fast rollback, small batches | Requires feature flag discipline, backward-compatible migrations, and a reliable pipeline |

## Consequences

- **Positive:**
  - What is tested in staging is byte-for-byte what runs in production.
  - "What is deployed where?" is answered by a version tag per environment.
  - Rollback is a redeploy of an existing image, measured in minutes.
  - Smaller, more frequent changes; fewer merge conflicts.
  - Release notes generated from Conventional Commits between tags.
- **Negative:**
  - Feature flags add code paths and need cleanup; stale flags become debt.
  - Migrations need expand/contract discipline; some schema changes take three releases.
  - `main` must stay releasable, which raises the bar on tests and review.
  - The team needed time to adjust; the first weeks had more questions about flags and versions.
- **Follow-ups:**
  - Rewrite Jenkins pipelines as build-once, promote-many, with digest verification.
  - Enable tag immutability in GHCR and stop using `latest` for deploys.
  - Protect `main`; delete the `develop`, `staging`, and `production` branches after the cutover.
  - Document the flow in [branching and releases](../../process/branching-and-releases.md) and walk
    the team through it.
  - Review after one quarter: deploy frequency, lead time, change failure rate, time to restore.
