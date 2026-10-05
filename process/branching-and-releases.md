# Branching and Releases

Development is trunk-based, with short-lived branches, and one immutable artifact is promoted
through every environment. The decision and the rejected alternatives are in
[ADR 0002](../docs/adr/0002-trunk-based-development.md).

## Summary

| Topic | Rule |
| --- | --- |
| Long-lived branches | Only `main` |
| Feature branches | Live less than 2 days, merged by squash |
| Unfinished work | Merged behind a feature flag, off by default |
| Build | Once per commit on `main`; the same image goes to every environment |
| Dev versions | `v<version>-dev.<shortsha>`, e.g. `v1.14.0-dev.3f9c2ab` |
| Release versions | Clean SemVer, e.g. `v1.14.0` |
| Registry | Docker images in GHCR |
| Pipelines | Jenkins, defined as code in the repo |
| Rollback | Redeploy the previous release tag; never rebuild |

## Why not environment branches

The team used to have `develop`, `staging`, and `production` branches, with merges between them to
"deploy". The problems were predictable:

- What ran in production was a different build from what was tested in staging.
- Merges between environment branches drifted and produced conflicts nobody owned.
- Hotfixes went to `production` and were forgotten on `develop`.
- "What is deployed where?" required reading git history across three branches.

An environment is a deployment target, not a branch. A version number now answers "what is deployed
where?".

## Branches

- Branch from `main`, name it `<type>/<ticket>-<slug>`:
  `feat/PAY-412-wager-progress-endpoint`, `fix/PAY-430-double-credit-on-retry`.
- Rebase or merge `main` into your branch at least daily if it lives that long.
- Open the PR early as a draft if you want feedback; mark ready when CI is green.
- Squash merge. The PR title becomes the commit message in
  [Conventional Commits](https://www.conventionalcommits.org/) form:

```text
feat(wallet): expose wagering progress per bonus (PAY-412)
fix(ledger): ignore duplicate settlement with same idempotency key (PAY-430)
chor(deps): bump @nestjs/core to 10.4.x
```

## Feature flags

Flags separate **deploy** (code is in production) from **release** (users see it).

| Flag type | Lifetime | Example |
| --- | --- | --- |
| Release flag | Removed within 2 sprints of full rollout | `wallet.wagering-progress-v2` |
| Ops / kill switch | Permanent, documented in the runbook | `referrals.payouts-enabled` |
| Experiment | Removed when the experiment ends | `bonus.welcome-offer-variant-b` |

Rules:

- New flags default to off.
- Every release flag has a cleanup ticket created the same day the flag is.
- Kill switches exist for every money-moving integration and are tested in staging.
- Do not nest flags. If you need two, you need a design conversation.

## Versioning and artifacts

### One build, many environments

```text
commit on main
   |
   v
Jenkins: lint -> test -> build image -> push ghcr.io/<org>/<service>:v1.14.0-dev.3f9c2ab
   |
   v
deploy dev (automatic)
   |
   v
deploy staging (automatic or one click, same image)
   |
   v
release: tag v1.14.0 on the same commit, retag the same image digest
   |
   v
deploy production (approved promotion, same digest)
```

- The image is built **once**. Promotion changes the tag, never the bytes. Verify by digest.
- Configuration differs per environment and comes from the environment, not from the image.
- Images are never overwritten. Tags are immutable in GHCR; `latest` is not used for deploys.

### Version scheme

| Kind | Format | Example | Created by |
| --- | --- | --- | --- |
| Dev build | `v<next-version>-dev.<shortsha>` | `v1.14.0-dev.3f9c2ab` | CI, every merge to `main` |
| Release | `v<major>.<minor>.<patch>` | `v1.14.0` | Release job, from a tested dev build |
| Hotfix release | Patch bump | `v1.14.1` | Same pipeline, same rules |

`<next-version>` is computed from Conventional Commits since the last release: `feat` bumps minor,
`fix` bumps patch, `BREAKING CHANGE` bumps major. Dev tags sort as SemVer pre-releases, so
`v1.14.0-dev.3f9c2ab` < `v1.14.0`.

### Releasing

1. Pick the dev build that passed staging, for example `v1.14.0-dev.3f9c2ab`.
2. Run the release job with that tag. It creates the git tag `v1.14.0` on commit `3f9c2ab`, adds
   the `v1.14.0` tag to the same image digest in GHCR, and generates release notes from commits.
3. Promote `v1.14.0` to production. Promotion requires an approval in Jenkins from someone other
   than the person who triggered it.
4. Watch the dashboards for 15 minutes. The deployer stays reachable for an hour.

### Hotfixes

There is no hotfix branch by default. Fix forward on `main`, let it flow through the normal
pipeline, release a patch version. The pipeline should be fast enough that this is the quickest
path. If `main` contains unreleasable work, it should have been behind a flag.

Exception: if `main` truly cannot ship, branch `release/v1.14` from the `v1.14.0` tag, cherry-pick
the fix, release `v1.14.1` from there, and write down why in the incident postmortem.

## Rollback

| Situation | Action |
| --- | --- |
| Bad code, no schema change | Redeploy the previous release tag. Target: under 10 minutes |
| Feature misbehaving | Turn the flag off. Target: under 2 minutes |
| Bad migration | Do not roll back the schema; fix forward. Migrations are backward compatible by design |

Rollback works because of two rules in the [Postgres guidelines](../backend/postgres-guidelines.md):
every migration is backward compatible with the previous release, and destructive steps ship in a
later release than the code that stops using them.

## Pipeline requirements

- `main` is protected: no direct pushes, green CI and an approval required.
- Pipeline from merge to dev deploy: under 15 minutes. If it gets slower, it is a tech debt item.
- Every stage that can fail the build runs before the image is pushed.
- Pipelines are in `Jenkinsfile` in the service repo and reviewed like code.
- Deploys post to the team channel with version, commit range, and who promoted.
