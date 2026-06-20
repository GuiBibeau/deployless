---
name: deployless-continuous-production-path-any-language
summary: Connect successful main commits to production deployment or publication for any language while preserving runtime release control.
description: Use this skill after mainline governance and release controls exist, when the repository needs CI/CD changes that automatically deploy, publish, or promote artifacts from main across any language, runtime, or hosting platform.
---

# Continuous Production Path for Any-Language Repositories

## Purpose

Make deployment from `main` mechanical, repeatable, and safe. The system should remove manual deploy decisions while preserving runtime release decisions through flags, kill switches, routing controls, or equivalent mechanisms.

The goal is:

```text
merge to main -> required checks -> build artifact -> deploy/publish/promote -> smoke checks -> observe -> release through runtime controls
```

Deployment moves code. Release exposes behavior.

## Preconditions

Run these skills first:

1. `01-delivery-model-audit`
2. `02-single-mainline-governance`
3. `03-runtime-release-controls`

Do not enable automatic production deployment when:

- required tests are missing
- deployment scripts are non-idempotent
- secrets are committed or unmanaged
- database migrations are destructive
- rollback path is unknown
- production observability is absent
- compliance requires an approval that has not been modeled

## Deployment target classification

Choose the correct target type before editing CI/CD.

| Repo type | Continuous production path means |
| --- | --- |
| Backend service | Build and deploy service artifact from `main` |
| Frontend app | Build and deploy static or SSR artifact from `main` |
| Serverless/edge | Publish functions from `main` after checks |
| Worker/batch job | Deploy worker artifact and keep risky jobs gated |
| Mobile app | Build/sign/submit or distribute candidate from `main`; release via app-store rollout, backend flags, or remote config |
| Library/package | Publish package artifacts from `main` via tags or release automation |
| CLI | Build and publish binaries/packages from `main` |
| Infrastructure | Plan on PR, apply from `main` with policy/approval gates |
| Monorepo | Deploy affected components independently from the same `main` commit |

## Inputs to inspect

- Existing CI/CD configuration
- Hosting provider or deployment platform
- Build artifact type
- Environment variables and secret management
- Current deploy scripts
- Smoke tests or health checks
- Rollback mechanism
- Migrations
- Deployment approvals
- Release tagging/changelog process
- Observability and incident process

## Pipeline architecture

Use the repo's existing CI provider when possible.

The target pipeline has these stages:

```text
validate
  format/lint/static analysis/typecheck/compile/tests/security checks

build
  produce immutable artifact or package

verify artifact
  test packaged artifact when possible

deploy from main
  deploy exact artifact associated with the main commit

post-deploy smoke
  health checks, migrations check, synthetic transaction, critical route check

observe
  emit version/deployment marker and check dashboards/alerts
```

## Artifact rules

- Build once, deploy the same artifact.
- Stamp artifacts with commit SHA, version, build time, and repo name.
- Avoid rebuilding different artifacts per environment unless the platform requires it.
- Environment-specific behavior should come from runtime config, not artifact differences, when practical.
- Store artifacts in the repo's normal registry, package store, container registry, app distribution service, or platform artifact store.

## CI/CD implementation rules

### Existing provider first

Do not migrate CI/CD providers just to implement this skill. Adapt existing systems:

- GitHub Actions
- GitLab CI
- Bitbucket Pipelines
- Azure Pipelines
- Jenkins
- CircleCI
- Buildkite
- cloud-hosted platform auto-deploy
- custom scripts

### Main trigger

Deployment or publication should trigger from `main` only after required checks pass.

Examples of acceptable triggers:

```text
push to main
merged merge queue batch
release tag created from main
manual approval job after main checks
platform auto-deploy tied to main
```

Avoid environment branches as the target model.

### Secrets

- Use the CI/CD provider's secret store.
- Do not write secrets to files in the repo.
- Do not print secrets in logs.
- Use least-privilege deployment credentials.
- Prefer short-lived cloud credentials or OIDC where the platform supports it.

### Environments

Use environments for infrastructure separation, not as long-lived source branches.

Acceptable:

```text
main commit -> staging deploy -> smoke -> production deploy
main commit -> preview deploy -> production deploy
main commit -> production canary -> production full
```

Not target model:

```text
develop branch -> staging
release branch -> qa
production branch -> production
```

### Approvals

If approvals are required, put them in the pipeline without reintroducing long-lived branches.

Examples:

- approval before production deploy job
- approval before infrastructure apply
- approval before flag reaches broad rollout
- approval before package publication

## Migration handling

If the repo uses persistent data:

- run migration validation in CI
- prefer non-destructive migrations before deploying code that uses new schema
- deploy code compatible with both old and new schema during transition
- separate destructive cleanup into a later deploy
- add rollback notes for every migration

For infrastructure repos:

- plan on PR
- policy-check the plan
- apply from `main`
- require approval when blast radius is high
- keep state locking enabled

## Rollback model

Prefer this order:

1. Turn off feature flag or kill switch.
2. Disable routing to new implementation.
3. Roll back runtime config.
4. Roll back deployment to previous artifact.
5. Roll forward with a small fix.
6. For data problems, follow the documented migration rollback/repair plan.

Document what each rollback option can and cannot undo.

## Post-deploy verification

Add smoke checks appropriate to the repo:

- service health endpoint
- frontend page fetch
- CLI version command
- background job dry run
- package install/import test
- mobile build validation
- infrastructure drift or policy check
- synthetic transaction for critical path

Smoke checks must be fast and deterministic. They are not a substitute for full tests.

## Observability requirements

Each production deploy should emit or expose:

```text
service/application name
commit SHA
artifact version
deployment time
environment
release-control provider version or config version when available
```

New flagged behavior should be observable by flag key and variant, with low-cardinality labels.

## Suggested docs

Create or update `docs/continuous-production-path.md`:

```markdown
# Continuous production path

## Trigger

## Required checks

## Artifact

## Deployment target

## Runtime release controls

## Secrets and credentials

## Post-deploy smoke checks

## Rollback

## Migration handling

## Manual approvals, if any

## Operational dashboards and alerts
```

## Output

Create or update:

- CI/CD workflow files using the existing provider
- build/test/package scripts only when missing and clearly inferable
- deployment docs
- smoke test scripts or commands
- deployment marker/version endpoint if suitable
- `docs/continuous-production-path.md`
- `docs/deployless-mainline-plan.md`, appending pipeline details

## Acceptance criteria

This skill is complete when:

- successful `main` commits have a documented path to production or publication
- the path is automated or has only documented approval gates
- the deployed artifact is tied to an exact commit
- release exposure remains controlled at runtime when possible
- post-deploy smoke checks exist or are explicitly documented as a blocker
- rollback procedure is documented
- environment branches are not required by the target model

## Anti-goals

- Do not deploy untested code automatically.
- Do not remove required compliance approvals.
- Do not commit secrets.
- Do not create a new CI/CD provider unless explicitly requested.
- Do not use deployment automation as a substitute for feature flags or migration safety.
