# context-budget

An Agent Skill for reducing the context cost of understanding and modifying a repository.

> Optimize for task context footprint, not file size.

A smaller file is not automatically better. Splitting cohesive code can add navigation, dependencies, and tool calls. `context-budget` treats scanner metrics as investigation signals and uses real maintenance tasks to decide whether a structural change helps.

## Capabilities

- `init` — inspect a repository and establish or migrate context-budget capability
- `check` — scan and explain repository context health
- `remediate` — resolve one WARN or FAIL while preserving behavior and API
- `cleanup` — reduce historical context debt one independent hotspot at a time
- `install` — establish the capability in another repository

## Local command contract

Use the repository's native runtime and task runner. The command should provide:

```text
context-budget
context-budget --detail <file>
context-budget --top N
context-budget --details
```

The default output is a compact summary with bounded FAIL and WARN file lists. Use `--detail <file>` for one complete diagnosis, `--top N` for hotspot triage, and `--details` only for a full expansion. WARN exits successfully; FAIL exits non-zero. The first version does not expose JSON output.

## Installation and use

Read [init.md](init.md) for the environment-first initialization and installation SOP. It selects the existing scanner or repository runtime before adding dependencies or CI configuration.

For a check, read [references/ci-contract.md](references/ci-contract.md). For remediation or historical cleanup, read [references/refactoring-playbook.md](references/refactoring-playbook.md). For cross-language, cross-platform, monorepo, or CI-provider decisions, read [references/portability.md](references/portability.md).

## Design boundaries

- Reuse an existing scanner instead of maintaining a second implementation.
- Do not split code merely to satisfy a threshold.
- Do not create ownerless `utils`, `helpers`, or `common` modules, thin wrappers, or forwarding layers.
- Preserve behavior and public APIs.
- Do not update baselines, add ignores, or raise thresholds to hide debt.
- Configure CI only when the repository already has CI or the user explicitly requests enforcement.

## Repository layout

```text
context-budget/
├── SKILL.md
├── README.md
├── init.md
└── references/
    ├── ci-contract.md
    ├── refactoring-playbook.md
    └── portability.md
```

The Skill is intentionally implementation-neutral. Repository-specific thresholds, baselines, scan scope, commands, and CI entry points remain owned by the target repository.
