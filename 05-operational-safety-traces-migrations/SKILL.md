---
name: deployless-operational-safety-traces-migrations-any-language
summary: Add production-style safety for any-language deployless mainline workflows through traces, replay tests, observability, rollback, and expand-contract migrations.
description: Use this skill after mainline deployment and runtime release controls are designed, when the repository needs operational safeguards that make continuous production deployment safe across languages and runtimes.
---

# Operational Safety, Traces, Replay, and Migrations for Any-Language Repositories

## Purpose

Add the safety layer that lets a single-mainline, continuously deployed repository operate without relying on long-lived branches as a risk buffer.

The skill focuses on:

- production trace and log correlation
- replayable representative scenarios
- flag-aware testing
- deployment observability
- kill switches and rollback drills
- expand-contract data migrations
- compatibility checks across versions

## Preconditions

Run these first:

1. `01-delivery-model-audit`
2. `02-single-mainline-governance`
3. `03-runtime-release-controls`
4. `04-continuous-production-path`

If the repo does not yet have runtime release controls or mainline deployment docs, add those before claiming deployless-style safety.

## Safety model

Long-lived branches hide risk by delaying integration. This model reduces risk by:

- integrating small changes quickly
- keeping unfinished behavior hidden behind release controls
- observing real behavior in production
- replaying representative scenarios before and after changes
- making state changes compatible across deploys
- reversing behavior through flags before rolling back infrastructure

## Inputs to inspect

- Logging format and correlation IDs
- Trace instrumentation
- Metrics and dashboards
- Error reporting
- Health checks
- Existing test fixtures
- Contract tests
- Snapshot/golden tests
- Database migrations
- Event schemas and message queues
- API versioning
- Background jobs and scheduled tasks
- Data retention and privacy constraints
- Incident response docs

## Trace context requirements

Add or standardize a request/execution context containing:

```text
trace_id
request_id or job_id
service_name
service_version or commit_sha
environment
operation name
actor or tenant opaque key when needed
release-control decisions
```

The exact mechanism depends on runtime:

- HTTP services: middleware/filter/interceptor
- GraphQL: resolver context
- message consumers: message metadata/context wrapper
- scheduled jobs: generated job execution ID
- CLI: command execution ID
- frontend/mobile: session/request correlation where privacy-safe
- infrastructure: plan/apply run ID
- libraries: caller-provided context where possible

## Logging and tracing rules

- Prefer structured logs over free-text logs for operational events.
- Include version and trace IDs in error reports.
- Record feature flag key and variant in traces when it affects behavior.
- Avoid high-cardinality labels in metrics.
- Do not log secrets, raw tokens, payment data, or sensitive personal information.
- Sample high-volume traces, but keep error traces.

## Replay testing

Replay tests use captured or synthetic production-like scenarios to detect regressions.

Suitable replay inputs:

- HTTP request fixtures with sensitive values redacted
- CLI command fixtures
- event/message payload fixtures
- job input fixtures
- contract examples from consumers
- frontend route/user-flow fixtures
- database state fixtures with synthetic data
- infrastructure plan snapshots

Create a repo-appropriate replay test directory. Examples:

```text
tests/replay/
spec/replay/
replay_tests/
testdata/replay/
fixtures/replay/
```

Use existing conventions first.

## Replay test procedure

1. Identify 3-5 critical production paths.
2. Create sanitized fixtures for those paths.
3. Add tests that run those fixtures against current code.
4. Add flag permutations for flagged behavior.
5. Validate expected outputs, status codes, events, state changes, or golden snapshots.
6. Run replay tests in CI before deployment.
7. Document how to add new fixtures during incidents or product launches.

## Flag-aware test matrix

For each important flag, test:

```text
default state
explicit off
explicit on
variant A/B when variants exist
provider unavailable
kill switch enabled
```

For combinations, avoid exhaustive explosion. Test:

- one normal path
- one rollout path
- one rollback path
- one provider-failure path

## Migration safety

Use expand-contract sequencing for persistent data, event schemas, and external contracts.

### Expand phase

- Add new schema, field, table, column, index, route, message field, or API shape without removing old behavior.
- Deploy code that can read/write both old and new forms when needed.
- Backfill safely and idempotently.
- Monitor correctness.

### Contract phase

- Confirm all deployed code and clients no longer depend on the old form.
- Remove old reads/writes.
- Remove old schema or compatibility code in a later deployment.
- Keep rollback notes and data repair path.

### Never do in a single deploy without explicit safety proof

- Drop columns or tables used by currently deployed code.
- Rename fields without compatibility handling.
- Change event payloads in a breaking way.
- Remove enum values that old producers/consumers may still use.
- Make irreversible destructive data changes without backup and recovery procedure.

## Compatibility windows

Define a compatibility window for each deploy target:

- backend service: current and previous version must tolerate shared state during rollout
- mobile app: backend must support old clients until supported app versions age out
- library: public API compatibility follows versioning policy
- event-driven systems: producers and consumers must tolerate old and new event versions
- infrastructure: plan/apply must account for state locking and provider drift

Document the compatibility window in `docs/migration-safety.md`.

## Rollback drills

Add or update incident docs with this sequence:

1. Identify version, flag state, and affected operation.
2. Turn off the relevant flag or kill switch.
3. Confirm metrics/errors recover.
4. If not, roll back deployment to previous artifact.
5. If data changed, run documented repair or compatibility path.
6. Add or update replay fixture from the incident.
7. Remove or fix the unsafe path before re-ramping.

Run a non-destructive rollback drill periodically if the repo owns production operations.

## Health and smoke checks

Add fast checks for:

- process is reachable
- critical route works
- dependency connectivity is healthy enough
- migration version is compatible
- queue worker can start
- package or CLI can be installed/invoked
- infrastructure plan/apply output is sane

Do not make smoke checks rely on fragile external data.

## Operational docs to create

Create or update:

```text
docs/operational-safety.md
docs/replay-tests.md
docs/migration-safety.md
docs/rollback.md
```

Use existing doc locations if the repo already has an operations handbook.

## Minimum templates

### `docs/replay-tests.md`

```markdown
# Replay tests

## Purpose

## Fixture sources

## Privacy/redaction rules

## How to add a fixture

## How to run replay tests locally

## CI behavior

## Flag permutations covered
```

### `docs/migration-safety.md`

```markdown
# Migration safety

## Expand-contract rule

## Compatibility window

## Backfill rules

## Rollback rules

## Destructive-change checklist
```

### `docs/rollback.md`

```markdown
# Rollback

## Prefer behavioral rollback first

## Feature flag rollback

## Deployment rollback

## Data repair or migration rollback

## Communication and audit log
```

## Output

Create or update:

- trace/request/job context helpers, if missing
- structured logging or trace annotations for release controls
- replay fixtures and replay tests for representative paths
- migration-safety docs and checks
- rollback docs
- smoke checks in CI/CD
- `docs/deployless-mainline-plan.md`, appending operational safety status

## Acceptance criteria

This skill is complete when:

- important execution paths have trace or correlation IDs
- release-control decisions are observable
- at least one representative replay test exists for a critical path
- replay tests can run in CI or the blocker is documented
- data migrations follow expand-contract rules or the repo is confirmed stateless
- rollback behavior is documented
- smoke checks exist for the deployment path or the blocker is documented

## Anti-goals

- Do not capture or commit sensitive production data.
- Do not add noisy logging that leaks private information or overwhelms systems.
- Do not treat replay tests as a replacement for unit and integration tests.
- Do not perform destructive migrations without an explicit, reviewed plan.
- Do not claim rollback is safe when data compatibility has not been considered.
