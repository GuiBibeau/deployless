# Deployless

Codex skills for moving a repository toward deployless delivery.

Deployless does not mean "no deployments." It means deployments become mechanical: every safe change merges to one mainline and moves through an automated path, while user-facing release is controlled separately at runtime with flags, kill switches, routing, config, or capability gates.

The result is a delivery model where:

- `main` is the single long-lived integration branch.
- Every merged change is production-deployable.
- CI gates protect the mainline before merge and deploy.
- Deployment from `main` is automated and repeatable.
- Release is controlled at runtime, not by long-lived branches.
- Risky data changes use expand-contract migrations.
- Production behavior is observable, reversible, and covered by replay tests where practical.

## What's included

| Skill | Purpose |
| --- | --- |
| `01-delivery-model-audit` | Inspect a repo and create a deployless migration plan. |
| `02-single-mainline-governance` | Move toward one long-lived mainline with merge gates. |
| `03-runtime-release-controls` | Add feature flags, kill switches, routing controls, or similar runtime gates. |
| `04-continuous-production-path` | Connect successful `main` commits to deployment, publication, or artifact promotion. |
| `05-operational-safety-traces-migrations` | Add tracing, replay tests, rollback docs, and safe migration rules. |

The skills are language-agnostic. They can be applied to services, frontends, CLIs, libraries, mobile apps, infrastructure repos, serverless functions, and monorepos.

## Install

Clone this repository, then copy the skill directories into your Codex skills directory:

```sh
git clone https://github.com/GuiBibeau/deployless.git
cd deployless

DEST="${CODEX_HOME:-$HOME/.codex}/skills"
mkdir -p "$DEST"
cp -R 01-delivery-model-audit "$DEST/"
cp -R 02-single-mainline-governance "$DEST/"
cp -R 03-runtime-release-controls "$DEST/"
cp -R 04-continuous-production-path "$DEST/"
cp -R 05-operational-safety-traces-migrations "$DEST/"
```

Restart Codex or start a new session so the skills are discovered.

## Use

Run the skills in order. Start with the audit:

```text
Use deployless-delivery-model-audit-any-language on this repository.
```

Then continue through the sequence:

```text
Use deployless-single-mainline-governance-any-language.
Use deployless-runtime-release-controls-any-language.
Use deployless-continuous-production-path-any-language.
Use deployless-operational-safety-traces-migrations-any-language.
```

The audit should create or update `docs/deployless-mainline-plan.md`. Later skills append implementation details and safety guidance to that plan.

## Recommended rollout

1. Audit the current branch, release, deployment, test, and observability model.
2. Make `main` the source of deployable truth.
3. Add runtime release controls before exposing unfinished behavior.
4. Automate deployment or publication from successful `main` commits.
5. Add rollback, tracing, replay tests, and migration safety.

Do not enable automatic production deployment until the audit shows that tests, release controls, rollback, and migration safety are adequate.

## Safety rules

- Do not remove review, tests, approvals, or compliance gates.
- Do not use long-lived environment branches as the target model.
- Do not conflate deployment with release.
- Do not commit production secrets or provider tokens.
- Do not perform destructive migrations without an expand-contract plan.
- Do not claim deployless delivery is complete while known safety blockers remain.

## References

- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
- [Atlassian: Trunk-based development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development)

## License

[MIT](LICENSE)
