# Initialize or Install Context Budget

This is the executable SOP for first use in a repository. Follow the steps in order. The repository's explicit request and existing configuration take precedence over these defaults.

## Step 1: Confirm the target and boundary

Before modifying files, establish:

- whether the repository already has a context-budget scanner, command, configuration, baseline, or CI check;
- whether the requested outcome is local checking only, CI enforcement, migration of an existing implementation, historical debt cleanup, or all of them;
- whether changes to CI, dependencies, configuration, and directory structure are allowed;
- whether this is a monorepo and whether it contains multiple independent packages or workspaces.

If the user did not request it, do not delete an existing implementation or add CI enforcement. Record unknowns instead of silently broadening scope.

## Step 2: Identify the repository environment

Inspect repository configuration and manifests before inferring from extensions. Record the evidence and the selected enforcement path.

### Language and toolchain

Identify:

- primary languages and their compiler, interpreter, or runtime;
- package manager and lockfile;
- workspace or monorepo manager and package boundaries;
- formatter, linter, test runner, and build commands;
- available parser, AST, compiler API, or equivalent structural analysis.

Prefer existing project commands and parsers. Do not introduce a language runtime solely for context-budget.

### Operating environment

Identify the current operating system and available shells, then separately inspect CI runner operating systems. Check whether Windows compatibility is required. Never treat the current shell as the only supported execution environment.

### CI and repository layout

Look for GitHub Actions, GitLab CI, CircleCI, Azure Pipelines, Buildkite, Jenkins, other provider configuration, or no CI. If multiple configurations exist, confirm which path is the primary enforcement path before editing it.

Map source, test, documentation, generated, vendor, build, cache, lock, and snapshot paths. Identify nested package-level CI and workspace boundaries; do not merge independent packages into one accidental scope.

### Git history

Check whether the directory is a Git repository, the current branch, the default branch if discoverable, available history depth, existing baseline files, and prior context-budget or structural-cleanup commits. If Git or sufficient history is unavailable, record `unavailable`; this must not block local `check` or `remediate`.

## Step 3: Recover existing context-budget capability

Locate and understand the scanner or command, thresholds, baseline, ignore/include rules, tests, WARN and FAIL classification, output formats, CI invocation, baseline update mechanism, and user-facing entry points. Read the relevant implementation and tests, then inspect recent history for intentional structural changes and prior cleanup decisions.

An existing runnable implementation is the primary source of truth. Do not replace a mature scanner merely to make it look like this SOP. If migration is requested, keep the old enforcement path until the equivalence checks in Step 9 pass.

## Step 4: Select an installation mode

Choose exactly one mode for the initial setup and state why.

### Mode A — reuse the existing implementation

Use when a scanner already runs and its metrics and output meet the contract.

1. Preserve the scanner.
2. Record the actual command, configuration, baseline, scope, and CI invocation.
3. Add only missing remediation, cleanup, detail, or initialization guidance.
4. Do not copy the scanner into the Skill or add a parallel implementation.

### Mode B — use the existing runtime/toolchain

Use when no scanner exists but the repository's runtime can reliably calculate the required metrics.

1. Use the repository's existing runtime and parser/compiler facilities.
2. Place the scanner in the repository's established tooling location, or use an explicitly adopted reusable implementation.
3. Add the smallest configuration, baseline, local command, and provider-specific CI invocation.
4. Use the existing test runner and package management flow.

### Mode C — add a lightweight dependency

Use only when native project capabilities cannot reliably calculate a required metric and a small ecosystem-aligned development dependency materially reduces risk.

1. State which metric native tooling cannot provide.
2. Add one maintained, lightweight dependency through the existing lockfile and development workflow.
3. Do not introduce a separate language runtime.
4. Provide cross-platform invocation and mark unsupported or estimated metrics explicitly.

### Mode D — local deterministic command only

Use when there is no CI or CI changes are out of scope.

1. Provide a local command using the repository's runtime.
2. Make scope, metrics, configuration, baseline behavior, and exit codes deterministic.
3. Document how an Agent invokes it; do not create a fictional CI configuration.

## Step 5: Establish configuration and scan scope

Confirm or create thresholds, baseline, ignore/include rules, workspace boundaries, token estimation strategy, and exclusions for generated, vendor, build, cache, lock, and snapshot material. Respect this precedence:

`explicit user request > repository configuration > Skill defaults`

When no repository values exist, use these defaults:

| Area | Warning | Hard / FAIL |
| --- | ---: | ---: |
| Code file tokens | 8,000 | 16,000 |
| Largest function or method | 150 LOC | 300 LOC |
| Functions plus classes | 25 | 50 |
| Direct internal dependencies | 10 | 20 |
| Document tokens or LOC | 6,000 or 500 | 12,000 or 1,000 |
| Largest document section tokens or LOC | 2,500 or 200 | 5,000 or 400 |

Do not set a whole-code-file LOC hard limit. If no real tokenizer is available, use a stable estimate, label it as an estimate, and use the identical algorithm for current and baseline values.

An existing warning becomes a regression FAIL only when growth exceeds 20% and also reaches the metric's minimum absolute growth: file tokens +1,000; largest function +30 LOC; definitions +5; dependencies +3; document tokens or LOC +100; section tokens or LOC +100. New files above WARN warn; new files above HARD fail. A pre-existing hard-limit file may remain a warning while it is not materially growing. Parse failures are failures.

## Step 6: Establish the local command

Provide the repository-appropriate command, with equivalent capabilities for:

- `context-budget`: compact full-repository summary;
- `context-budget --detail <file>`: complete diagnosis for only one file;
- `context-budget --top N`: the most severe or costly hotspots for bounded triage;
- `context-budget --details`: complete diagnostics for all findings when explicitly needed.

The command must work without CI, use the repository runtime, have stable exit codes, and keep the default output small. Do not add JSON output in the first version. Prefer a package/task-manager entry point over shell-specific wrappers.

## Step 7: Configure CI only when authorized

Configure CI only if the repository already has CI or the user explicitly requested enforcement.

1. Use the repository's existing provider, runner, install process, cache conventions, and workspace selection.
2. Invoke the same local implementation; do not maintain CI-only scanning logic.
3. Make WARN exit with zero and FAIL exit non-zero.
4. Do not refresh the baseline, add ignores, or raise thresholds automatically in CI.
5. State the trigger scope and any runner or platform limitation.
6. Check that the command does not depend on a single shell; document a real limitation if the repository cannot support all target platforms.

## Step 8: Verify initialization

Run the local command and relevant repository tests. Verify all of the following:

- configuration, baseline, ignore rules, and workspace scope are actually read;
- WARN returns zero and FAIL returns non-zero;
- default output is compact;
- `--detail <file>` expands only the requested file;
- `--top N` bounds hotspot output and `--details` is the explicit full expansion;
- local and CI commands use the same implementation;
- check and remediate work without Git history;
- no unnecessary runtime, dependency, CI provider, or directory structure was introduced.

## Step 9: Equivalence acceptance for scanner migration

If the old scanner is being replaced, run old and new implementations against the same revision and compare:

- scan scope and exclusions;
- metric values;
- WARN and FAIL assignments;
- exit codes;
- baseline and historical-growth behavior.

Explain every difference, run all relevant tests, and run the candidate Skill or CLI against an isolated temporary fixture repository outside the target repository. Delete the old implementation only after equivalent behavior or an explicitly accepted, evidenced change is established. Never remove repository CI enforcement as an incidental part of migration.

## Step 10: Report initialization

Report:

1. detected languages, runtime, package manager, parsers, and CI provider;
2. source, test, documentation, and workspace boundaries;
3. existing scanner, baseline, configuration, and CI;
4. selected mode and reason;
5. created or reused command;
6. created or changed configuration;
7. whether CI was configured and why;
8. verification results;
9. whether replacement of the old implementation is now justified;
10. remaining environment limitations or unavailable history.
