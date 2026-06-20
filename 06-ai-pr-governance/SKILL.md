---
name: deployless-ai-pr-governance-any-language
description: Manage high-volume AI-generated pull requests and multi-agent coding work in deployless repositories. Use when a repo has many bot or agent PRs, parallel agent work, noisy low-context PRs, merge-queue pressure, CI exhaustion, duplicate fixes, ownership confusion, or review bottlenecks that threaten a single-mainline delivery model.
---

# AI PR Intake and Multi-Agent Governance

## Purpose

Keep a deployless mainline healthy when AI agents can generate pull requests faster than humans can review them.

The target model is:

- every AI PR has clear intent, owner, scope, tests, and rollback notes
- only small, reviewable, production-deployable changes enter the mainline queue
- multiple agents do not fight over the same files, migrations, flags, or release paths
- CI capacity is protected from speculative or duplicate work
- humans review decisions and risk, not piles of unexplained diffs

## Preconditions

Run these first when possible:

1. `01-delivery-model-audit`
2. `02-single-mainline-governance`
3. `03-runtime-release-controls`
4. `04-continuous-production-path`
5. `05-operational-safety-traces-migrations`

If the repo is already drowning in AI PRs, apply this skill immediately as an intake stopgap, then backfill the earlier deployless plan.

## Inputs to inspect

- Current open PR count, draft PR count, and PR authors
- Bot, agent, automation, and dependency-update accounts
- PR templates, issue templates, labels, milestones, projects, and merge queue
- CODEOWNERS or equivalent ownership rules
- Branch protection, required checks, and required reviews
- CI cost, queue time, cancellation behavior, and flaky checks
- Repeatedly touched files, merge-conflict hotspots, migrations, and generated files
- Existing agent instructions, task trackers, handoff docs, and branch naming conventions

## Intake states

Create or adapt labels for these states:

```text
ai/generated
ai/needs-intent
ai/needs-owner
ai/needs-scope
ai/needs-tests
ai/conflict-risk
ai/ci-expensive
ai/ready-for-human-review
ai/parked
ai/rejected
```

Use the repo's existing label style if it differs.

## Triage rules

Every AI PR must answer:

- What user or operator problem does this solve?
- What issue, ticket, incident, or decision authorized it?
- What files and behaviors are intentionally in scope?
- What tests or checks prove the change?
- What could go wrong after deployment?
- How can the change be disabled, rolled back, or followed up?

If those answers are missing, mark the PR `ai/needs-intent` or `ai/needs-scope` and keep it out of the merge queue.

Reject or close AI PRs that:

- duplicate an active PR or accepted plan
- are mostly churn, formatting, renames, or broad refactors without a named need
- modify generated files without updating the source
- change production deployment, secrets, migrations, auth, billing, or security-sensitive paths without an explicit owner
- bundle unrelated fixes that cannot be reviewed independently
- pass CI but lack a coherent reason to exist

## Low-context PR handling

Treat low-context AI PRs as untrusted until they provide evidence. A PR that only says "improves the code" or "fixes issues" is not ready for review.

Require a short PR body with:

```markdown
## Intent

## Scope

## Validation

## Risk and rollback
```

For vague PRs, ask the agent to rewrite the PR body and split the diff before requesting human review.

## Multi-agent work contract

Use these rules when several agents work in the same repository:

- One owner per task, PR, or issue.
- One active agent per risky file area unless an explicit coordinator owns the integration.
- Agents claim scope in the issue, task, or coordination doc before editing.
- Agents use short-lived branches and delete them after merge or rejection.
- Agents do not force-push, rebase, or rewrite another agent's branch without explicit handoff.
- Agents rebase or update from `main` before moving a PR to review-ready.
- Agents leave a handoff note when pausing work.

Recommended branch names:

```text
agent/<tool-or-agent>/<ticket-or-area>-<short-slug>
bot/<provider>/<ticket-or-area>-<short-slug>
human/<name>/<ticket-or-area>-<short-slug>
```

## PR size and shape

Prefer PRs that are:

- under 400 changed lines when practical
- limited to one behavior, bug, flag, migration phase, or cleanup
- reviewable without understanding unrelated rewrites
- covered by focused tests or an explicit test-gap note
- deployable immediately after merge

Exceptions are allowed for generated lockfiles, vendored updates, mechanical migrations, or broad formatting changes, but they must be labeled and reviewed as mechanical.

## Queue policy

Use a visible queue with WIP limits:

- Cap active AI PRs per repository or ownership area.
- Keep speculative AI PRs in draft until intent and tests are clear.
- Prioritize incident fixes, production blockers, security fixes, and already-reviewed small PRs.
- Batch dependency updates only when the repo already has reliable tests.
- Park stale AI PRs that conflict repeatedly with `main`.
- Close rejected PRs promptly so they do not drain review attention.

Suggested limits:

```text
max active AI PRs per owner area: 3-5
max review-ready AI PRs per human reviewer: 2-3
max parallel migration PRs touching the same data model: 1
max speculative refactor PRs in merge queue: 0
```

Adjust limits to team size and CI capacity.

## CI protection

Protect CI from PR floods:

- Run cheap checks before expensive checks.
- Cancel superseded runs on the same branch.
- Require intent/scope labels before expensive end-to-end, load, mobile, or integration jobs.
- Use path-aware checks in monorepos.
- Split flaky-check triage from product review.
- Do not let one bot account starve human fixes or incident PRs.

When a PR is `ai/ci-expensive`, require a reviewer or owner to approve full CI.

## Review gates

Before an AI PR enters merge queue, require:

- clear intent and linked issue or decision
- one accountable owner
- small enough diff or documented exception
- passing required checks
- tests for changed behavior or a documented test gap
- no unowned migration, auth, billing, secret, deployment, or security change
- release control for unfinished or risky behavior
- rollback or follow-up plan when production behavior changes

For high-risk paths, require CODEOWNER review even if CI passes.

## Docs to create

Create or update:

```text
docs/ai-pr-intake.md
docs/multi-agent-work.md
docs/deployless-mainline-plan.md
```

Use existing contribution or operations docs if the repo already has them.

## Minimum templates

### `docs/ai-pr-intake.md`

```markdown
# AI PR intake

## Intake labels

## Triage rules

## Review-ready requirements

## Queue limits

## CI policy

## Close or park rules
```

### `docs/multi-agent-work.md`

```markdown
# Multi-agent work

## Claiming work

## Branch naming

## Handoff notes

## Conflict ownership

## High-risk paths
```

## Output

Create or update:

- PR template with intent, scope, validation, risk, and rollback sections
- labels or label documentation for AI PR intake
- CODEOWNERS or ownership docs for sensitive paths
- branch protection or merge queue documentation
- CI concurrency and path-filtering settings where safe
- `docs/ai-pr-intake.md`
- `docs/multi-agent-work.md`
- `docs/deployless-mainline-plan.md`, appending AI PR governance status

## Acceptance criteria

This skill is complete when:

- AI-generated PRs have a documented intake path
- low-context PRs are kept out of review-ready and merge-queue states
- multi-agent branch and handoff rules are documented
- ownership exists for high-risk paths
- CI has a plan for flood control or the blocker is documented
- stale, duplicate, speculative, and unreviewable AI PRs can be parked or closed quickly
- deployless mainline rules still apply to every AI PR

## Anti-goals

- Do not auto-merge AI PRs just because CI passes.
- Do not create a separate long-lived AI integration branch.
- Do not let agents bypass CODEOWNERS, security review, or compliance gates.
- Do not use labels as a substitute for reading risky diffs.
- Do not preserve stale AI PRs merely because an agent spent time generating them.
