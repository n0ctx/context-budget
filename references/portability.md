# Portability and Installation Boundaries

Use this reference when installing context-budget in a repository whose language, operating system, package manager, or CI provider is not known in advance.

## Environment-first selection

Inspect the target repository before selecting an implementation. Use its manifests, lockfiles, task definitions, build configuration, parser/compiler APIs, test runner, and existing CI files as evidence. Select the first option that fully satisfies the metrics:

1. existing context-budget implementation;
2. existing repository runtime and toolchain;
3. standard library or platform-native analysis;
4. a maintained, lightweight dependency aligned with the repository ecosystem;
5. a small custom implementation.

Do not introduce a new language runtime only for this check. If a metric cannot be supported reliably, mark it unsupported or estimated and keep that fact visible in the result. Never substitute a superficial text count for a structural metric while presenting it as equivalent.

## Repository boundaries

Identify whether the repository is a single package or a monorepo. Record each independent workspace, source root, test root, documentation root, and package-level CI path. Scan the declared scope rather than the physical repository root when those differ. Apply exclusions consistently to every workspace.

The default exclusions are generated output, vendor code, build and distribution output, dependency directories, caches, lock material, snapshots, and repository control metadata. A repository may override these with explicit configuration. Do not exclude a finding merely because it is inconvenient; make ignores reviewable and intentional.

## Command and shell portability

Expose the scanner through the repository's native task mechanism or an executable entry point. Keep arguments and exit codes stable across macOS, Linux, and Windows. Do not require Bash, Zsh, PowerShell, Unix path syntax, or shell-specific environment expansion in the Skill contract.

When a wrapper is necessary, keep it a thin invocation owned by the repository's existing tooling and provide an equivalent native command for supported platforms. Do not duplicate scanner logic in separate shell scripts.

## CI provider portability

If CI exists, adapt the smallest invocation to the provider already used by the repository: its runner image, dependency installation, cache, workspace selection, and trigger conventions. Do not generate a provider-specific file from a generic assumption.

If no CI exists and the user did not request enforcement, install only a deterministic local command. A repository must not receive a fictional CI configuration. If the user explicitly requests enforcement, confirm the desired provider or use a provider already selected by repository policy before adding configuration.

The CI job must call the same local scanner. WARN remains visible and non-blocking; FAIL blocks. CI must not rewrite baseline, append ignores, or change thresholds. State platform limitations rather than silently generating a command that works only on the current machine.

## Git availability

Git history improves prioritization but is optional. With full history, use the bounded temporal-coupling rules in the refactoring playbook. With a shallow repository, no Git repository, or an unavailable default branch, mark history as unavailable and continue with current metrics, tests, and ownership analysis. `check` and `remediate` must remain usable without history.

## Configuration ownership

Keep repository-specific thresholds, baseline, include/exclude rules, workspace selection, tokenizer choice, and CI entry points in repository-owned configuration or its normal task files. Keep general decisions and workflow in the Skill. Do not require every repository to copy a full scanner source tree. If an existing scanner is retained, the Skill documents how to invoke and govern it; if it is migrated, complete the equivalence acceptance before removing anything.
