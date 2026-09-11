---
name: context-budget
description: Control the context cost of coding-agent work by scanning, explaining, remediating, and installing context-budget checks for source code and documentation. Use for context debt, context cost, large source files or documentation, WARN or FAIL remediation, historical cleanup, large-module refactoring, repository maintainability, initializing or installing a context-budget guard, migrating a scanner, or configuring context-budget CI; do not use for line-count cleanup without a context-footprint question.
---

# Context Budget

Optimize for task context footprint, not file size.

A smaller file is not automatically better. A split that adds navigation, dependencies, or tool calls can increase the context required for ordinary changes. Treat scanner metrics as investigation signals and task footprint as the architectural acceptance criterion.

## Modes

- `init`: inspect a repository and establish or migrate context-budget capability. Read [init.md](init.md) before changing anything.
- `check`: run the repository's deterministic scanner and explain context health. Read [references/ci-contract.md](references/ci-contract.md) when interpreting metrics, baselines, exit codes, or output.
- `remediate`: resolve one selected WARN or FAIL while preserving behavior. Read [references/refactoring-playbook.md](references/refactoring-playbook.md).
- `cleanup`: process historical context debt one independent hotspot at a time. Read [references/refactoring-playbook.md](references/refactoring-playbook.md) and use Git history only as supporting evidence.
- `install`: establish the capability in another repository. Follow [init.md](init.md), then read [references/portability.md](references/portability.md) and [references/ci-contract.md](references/ci-contract.md) as needed.

## Operating rules

1. Discover the repository's existing scanner, command, configuration, baseline, ignore rules, tests, and CI before choosing an implementation. Reuse a working implementation; never maintain a second scanner without an explicit equivalence plan.
2. Give priority to hard-limit/FAIL findings, files with several warnings, and high-cost files with meaningful maintenance churn. Ordinary WARN findings may remain when the module is cohesive and a split would increase task footprint.
3. Diagnose the change vectors and ownership before editing. Do not split merely because a file is long, a definition count is high, or an import count is high. Never create `utils`, `helpers`, `common`, thin wrappers, forwarding layers, or ownerless modules to make a metric green.
4. Process one independent hotspot at a time. Establish a test baseline, make the smallest behavior-preserving change, run relevant tests immediately, and recheck metrics and dependency direction before moving on.
5. WARN is non-blocking; FAIL is blocking. Do not hide a finding by changing thresholds, adding ignores, or refreshing the baseline. Baseline updates belong to an explicitly approved, final cleanup acceptance step.
6. Keep the scanner itself from becoming a context burden. The default command prints a compact repository summary, bounded FAIL and WARN file lists, and expansion instructions. It must support equivalent forms of `context-budget`, `context-budget --detail <file>`, `context-budget --top N`, and `context-budget --details`; detailed diagnostics are opt-in. Do not expose JSON in the first version.
7. Preserve external behavior and public APIs. If a structural change cannot demonstrate a smaller task context footprint, keep the cohesive design and record why the warning remains.

## Workflow

1. Identify the active mode and read only its linked guidance.
2. Run the repository command with its current configuration and baseline. Record scope, counts, exit code, and the highest-priority files.
3. For code remediation or cleanup, choose at least three representative change scenarios and compare the pre-change and post-change working set before accepting a substantial split.
4. Implement and test one hotspot. Re-run the scanner with `--detail <file>` or `--top N` as appropriate; use `--details` only when a complete diagnostic is needed.
5. Run the repository's relevant tests, lint, build, and final scanner. Report changed files, metric deltas, structural decisions, retained warnings and reasons, CI/baseline state, and any unverified gaps.

The references are intentionally one level deep:

- [references/ci-contract.md](references/ci-contract.md): metrics, baseline semantics, exit codes, and progressive-disclosure output.
- [references/refactoring-playbook.md](references/refactoring-playbook.md): change-vector analysis, footprint acceptance, and historical cleanup.
- [references/portability.md](references/portability.md): environment discovery, runtime/toolchain selection, monorepos, OS, shell, and CI portability.
