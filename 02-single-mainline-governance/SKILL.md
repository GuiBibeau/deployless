---
name: deployless-single-mainline-governance-any-language
summary: Convert any-language repositories toward one long-lived main branch with CI-gated merges and no long-lived feature, develop, or release branches.
description: Use this skill after the delivery audit when the repository needs trunk-based governance, branch protection, contribution rules, review rules, merge gates, and mainline-friendly development practices independent of programming language.
---

# Single Mainline Governance for Any-Language Repositories

## Purpose

Make `main` the single long-lived branch used for integration and release, while preserving safety through review, tests, ownership, and runtime release controls.

The intended result is not “everyone commits anything directly to production.” The intended result is:

- small changes
- short-lived review branches when needed
- fast integration into `main`
- required checks before merge
- no long-lived `develop`, `staging`, `qa`, or `release/*` branches as integration backlogs
- unfinished behavior hidden behind release controls

## Preconditions

Run `01-delivery-model-audit` first. Do not apply this skill blindly. The audit should identify:

- default branch name
- existing long-lived branches
- CI provider
- test commands
- release process
- deployment dependencies on branch names
- code review rules
- compliance or approval requirements

## Branch model target

Use this model unless repo-specific constraints require a documented exception:

```text
main                       long-lived source of truth
short-lived/topic-branch    optional, review-only, deleted after merge
hotfix branch               optional, short-lived, merged back to main immediately
release tag                 optional immutable marker, not a long-lived branch
```

Forbidden as long-lived integration branches:

```text
develop
staging
qa
release/*
env/*
production
preprod
integration
```

These names may still exist temporarily during migration, but they should not remain part of the target operating model.

## Files to inspect and possibly update

- `CONTRIBUTING.md`
- `README.md`
- `CODEOWNERS`
- `.github/pull_request_template.md` or equivalent PR templates
- `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`, `azure-pipelines.yml`, `buildkite.yml`
- release scripts and docs
- deployment configuration that references branch names
- docs mentioning `develop`, `staging`, `qa`, `release/*`, or manual branch promotion
- package publishing or artifact release config
- monorepo ownership files

## Implementation procedure

### 1. Confirm the default branch

Detect whether the default branch is `main`, `master`, or another name.

- If the repo already uses `main`, preserve it.
- If it uses `master`, do not rename automatically unless the user asked for branch renaming. Document `main` as the conceptual trunk and adapt wording to the actual default branch if needed.
- If deployment config points to another production branch, document the current behavior before changing it.

### 2. Add or update contribution guidance

Create or update `CONTRIBUTING.md` with:

```markdown
# Mainline development

This repository uses a single long-lived mainline branch.

- Keep changes small and merge frequently.
- Use short-lived branches only for review.
- Do not create long-lived feature, develop, staging, qa, or release branches.
- Keep `main` deployable at all times.
- Hide incomplete product behavior behind runtime release controls.
- Prefer forward fixes and feature-flag rollback over branch reverts.
- Use expand-contract migrations for persistent data changes.
```

Adapt wording to the repo's existing tone and compliance requirements.

### 3. Add merge readiness checklist

Add a checklist to the PR template or contribution docs:

```markdown
## Mainline readiness

- [ ] The change is small enough to review safely.
- [ ] The change is deployable when merged to main.
- [ ] Incomplete or risky behavior is behind a runtime release control.
- [ ] Required tests were added or updated.
- [ ] Observability was added for new production behavior.
- [ ] Data migrations are backward-compatible, if any.
- [ ] Rollback or kill-switch behavior is documented.
```

If the repo does not use PRs, put the same checklist in `CONTRIBUTING.md` under “Pre-merge checklist.”

### 4. Align CI with mainline

Update CI triggers so the same required checks run on:

- pull requests or merge requests into `main`
- direct pushes to `main`, if allowed
- merge queue/batch merge events, if the platform supports them

Do not assume a CI provider. Use the existing CI system.

Minimum checks should include the repo-appropriate equivalent of:

- dependency install or restore
- formatting check
- lint or static analysis
- type check or compile
- unit tests
- integration or contract tests where available
- security/dependency checks where already present
- migration validation where persistent data exists
- build/package verification

### 5. Document branch protection rules

If the connector or repo access allows branch protection changes, configure them only when safe. Otherwise create `docs/mainline-branch-protection.md` with desired settings:

```markdown
# Mainline branch protection

Required for `main`:

- Pull request or equivalent review before merge, unless direct-to-main is explicitly approved.
- Required status checks must pass.
- Branch must be up to date or merge queue must be enabled.
- No force-pushes.
- No deletion of `main`.
- Code owner review for owned paths.
- Signed commits if the repo already requires them.
- Admin bypass only for documented break-glass cases.
```

### 6. Replace branch-based release language

Search docs and scripts for branch-promotion vocabulary:

- `merge to develop`
- `promote staging to production`
- `cut release branch`
- `qa branch`
- `production branch`
- `release/*`

Replace with mainline-oriented language:

- merge to `main`
- deploy from `main`
- release through runtime controls
- validate via preview/staging environment generated from commit or artifact
- tag immutable release points only if needed

### 7. Preserve necessary compliance gates

Some environments require approvals, change tickets, or segregation of duties. Do not remove them. Instead, make them part of the `main` merge or deployment gate.

Examples:

- approval required before enabling a flag to 100%
- approval required before production deploy job proceeds
- change ticket linked in PR template
- CODEOWNERS required for sensitive paths
- audit log entry for flag changes

## Migration plan from long-lived branches

When a repo currently depends on long-lived branches, use a staged migration:

1. Freeze new work on long-lived branches.
2. Rebase or merge active work into short-lived branches targeting `main`.
3. Move environment deployment triggers away from branch names and toward commits/artifacts from `main`.
4. Convert unfinished work to runtime release controls.
5. Delete stale branches only after deployment and release docs no longer reference them.
6. Protect `main` and document the new process.

## Monorepo rules

For monorepos:

- `main` is still the only long-lived branch.
- Use path-aware CI to run affected checks.
- Keep ownership in `CODEOWNERS` or equivalent.
- Deploy services independently from the same mainline when possible.
- Use per-service release controls.
- Avoid service-specific long-lived branches.

## Library/package rules

For libraries:

- `main` must always build and pass tests.
- Publishing may be tag-based, but tags are immutable markers, not integration branches.
- Use compatibility tests and semantic versioning policy where applicable.
- Use feature flags only for runtime libraries that can evaluate runtime context; otherwise use versioned APIs and compatibility shims.

## Infrastructure repo rules

For infrastructure-as-code repos:

- `main` must represent intended infrastructure state.
- Pull requests should run format, validate, plan, and policy checks.
- Production apply can remain approval-gated.
- Do not maintain long-lived environment branches unless there is a documented regulatory reason.
- Prefer environment directories, workspaces, stacks, or overlays over environment branches.

## Output

Create or update:

- `CONTRIBUTING.md`
- PR/MR template, if present or appropriate
- `docs/mainline-branch-protection.md`
- `docs/deployless-mainline-plan.md`, appending a “Mainline governance” section
- CI config triggers and required checks, when safe to edit

## Acceptance criteria

This skill is complete when:

- the repo has documented single-mainline rules
- long-lived branch usage is identified and either removed or given a migration plan
- PR/merge readiness criteria mention deployability and release controls
- CI runs required checks for changes targeting `main`
- branch protection expectations are documented or configured
- deployment/release docs no longer teach branch promotion as the target model

## Anti-goals

- Do not remove approvals required for safety, security, or compliance.
- Do not delete active branches without explicit instruction and a migration plan.
- Do not force direct commits to `main` if PRs are the established review mechanism.
- Do not replace runtime release controls with branch discipline alone.
