# skillrail

**One skill source. Every agent surface.**

Your org writes the same agent instructions five times: a Claude Code skill, a Cursor rule, a Copilot instructions file, a Windsurf rule, and an `AGENTS.md` section. They drift immediately, nobody knows which version is current, and there's no owner, no versioning, and no review gate.

Anthropic's own enterprise guidance tells you to solve this yourself:

> "Custom Skills do not sync across surfaces… If your organization deploys Skills across multiple surfaces, implement your own synchronization process."
> "Maintain an internal registry for each Skill with: Purpose… Owner… Version… Dependencies… Evaluation status."
> — [Anthropic Agent Skills enterprise docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/enterprise)

skillrail is that missing piece: a canonical, versioned skill registry that **compiles** to every tool your teams actually use.

## How it works

Skills live once, in the open Anthropic `SKILL.md` format, with org metadata:

```
skills/
  deploy-checklist/
    SKILL.md          # frontmatter: name, description, version, owner, targets
    reference.md      # optional supporting files (copied for Claude)
```

```bash
npm i -g skillrail
skillrail init                 # set up skillrail.json + skills/ in a repo
skillrail add deploy-checklist # scaffold a skill
skillrail sync                 # compile + write to every configured target
```

`sync` generates, idempotently, with do-not-edit provenance headers:

| Target     | Output |
|------------|--------|
| `claude`   | `.claude/skills/<name>/` (verbatim, supporting files included) |
| `cursor`   | `.cursor/rules/<name>.mdc` |
| `copilot`  | `.github/instructions/<name>.instructions.md` |
| `windsurf` | `.windsurf/rules/<name>.md` |
| `agentsmd` | one managed block in `AGENTS.md` (Codex, Jules, Amp, …) |

Per-skill targeting: `targets: [claude, cursor]` in frontmatter overrides the project default.

## Share skills across the org

Any git repo is a registry — platform team publishes, product teams consume:

```bash
skillrail pull git@github.com:acme/agent-skills.git deploy-checklist
skillrail sync
```

The lockfile records origin + commit for every installed skill.

## Governance built in

```bash
skillrail check    # lint: description present & ≤1024 chars, owner set, semver,
                   # body ≤500 lines — exit 1 on failure (CI gate)
skillrail status   # exit 2 if any target is stale, missing, or DRIFTED
                   # (drift = someone hand-edited a generated file; flagged before sync overwrites it)
skillrail list     # inventory: version, owner, targets, origin
```

Drop `check` + `status` into CI and skill quality/consistency becomes a merge requirement, exactly the lifecycle Anthropic's enterprise docs prescribe — but automated.

## Why this exists (the market gap, verified mid-2026)

- Agent observability/eval is saturated (Braintrust $80M Series B; Langfuse acquired by ClickHouse; promptfoo acquired by OpenAI).
- Memory, orchestration, and interop are absorbed by funded startups and hyperscalers (Mem0, Letta; A2A in Copilot Studio and Bedrock AgentCore).
- But **skill/instruction management across tools and teams has no incumbent**: vendors each ship their own silo format, Anthropic explicitly punts cross-surface sync to customers, and 97% of orgs report they haven't scaled agents across the organization.

## Roadmap (the paid layer)

The CLI stays MIT/open-source. The hosted product adds what teams pay for:

1. **Hosted registry** — private org registry with RBAC, review/approval flow on skill versions, and audit log.
2. **Usage telemetry** — which skills get invoked, by which agents, and which correlate with successful runs; kill dead skills with data.
3. **API/claude.ai surface push** — sync skills to Anthropic API workspaces and claude.ai, not just repos.
4. **Skill eval harness** — run each version against recorded scenarios before promotion (CI for skills).

Pricing sketch: free CLI → $29/seat/mo Team (hosted registry, telemetry) → Enterprise (SSO, audit, on-prem).

## Notes

- Zero runtime dependencies, Node ≥18, single file.
- Name check pending before npm publish (`skillrail` npm namespace / trademark).
