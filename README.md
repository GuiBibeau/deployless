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
- AI-generated PRs are scoped, owned, queued, and reviewed before they can pressure the mainline.
- Agent work moves through scope challenge, planning artifacts, test-first implementation, and independent review.
- Production behavior is observable, reversible, and covered by replay tests where practical.

## What's included

| Skill | Purpose |
| --- | --- |
| `deployless-audit` | Inspect a repo and create a deployless migration plan. |
| `deployless-mainline` | Move toward one long-lived mainline with merge gates. |
| `deployless-release-controls` | Add feature flags, kill switches, routing controls, or similar runtime gates. |
| `deployless-production-path` | Connect successful `main` commits to deployment, publication, or artifact promotion. |
| `deployless-operational-safety` | Add trace/replay testing, observability, rollback docs, and safe migration rules. |
| `deployless-ai-pr-governance` | Manage large AI PR inflows, queue pressure, and multi-agent work. |

The skills are language-agnostic. They can be applied to services, frontends, CLIs, libraries, mobile apps, infrastructure repos, serverless functions, and monorepos.

Each skill follows the standard Codex skill layout:

```text
skill-name/
  SKILL.md
  agents/openai.yaml
```

## Install

Clone this repository, then copy the skill directories into your Codex skills directory:

```sh
git clone https://github.com/GuiBibeau/deployless.git
cd deployless

DEST="${CODEX_HOME:-$HOME/.codex}/skills"
mkdir -p "$DEST"
cp -R deployless-audit "$DEST/"
cp -R deployless-mainline "$DEST/"
cp -R deployless-release-controls "$DEST/"
cp -R deployless-production-path "$DEST/"
cp -R deployless-operational-safety "$DEST/"
cp -R deployless-ai-pr-governance "$DEST/"
```

Restart Codex or start a new session so the skills are discovered.

## Use

Run the skills in order. Start with the audit:

```text
Use $deployless-audit on this repository.
```

Then continue through the sequence:

```text
Use $deployless-mainline.
Use $deployless-release-controls.
Use $deployless-production-path.
Use $deployless-operational-safety.
Use $deployless-ai-pr-governance.
```

The audit should create or update `docs/deployless-mainline-plan.md`. Later skills append implementation details and safety guidance to that plan.

## Recommended rollout

1. Audit the current branch, release, deployment, test, and observability model.
2. Make `main` the source of deployable truth.
3. Add runtime release controls before exposing unfinished behavior.
4. Automate deployment or publication from successful `main` commits.
5. Add rollback, tracing, replay tests, and migration safety.
6. Add AI PR intake and multi-agent work rules before PR volume overwhelms review.
7. Use a coordinator-led agent workflow: challenge unclear scope, publish ready slices, design alternatives for important interfaces, build with vertical test-first loops, and review standards plus spec before merge.

Do not enable automatic production deployment until the audit shows that tests, release controls, rollback, and migration safety are adequate.

## Safety rules

- Do not remove review, tests, approvals, or compliance gates.
- Do not use long-lived environment branches as the target model.
- Do not conflate deployment with release.
- Do not commit production secrets or provider tokens.
- Do not perform destructive migrations without an expand-contract plan.
- Do not auto-merge AI PRs just because CI is green.
- Do not let multiple agents edit the same risky area without an owner.
- Do not let agents implement unclear work before scope, acceptance criteria, and test seams are resolved.
- Do not claim deployless delivery is complete while known safety blockers remain.

## References

- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
- [Atlassian: Trunk-based development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development)

## License

[MIT](LICENSE)
