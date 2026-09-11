# Context-Budget Check Contract

Use this reference when implementing or interpreting a scanner, baseline, local command, or CI gate. It defines behavior, not a required language or CI provider.

## Metrics

Measure only supported, meaningful source and documentation units within the selected repository or workspace scope.

| Metric | Default warning | Default hard | Interpretation |
| --- | ---: | ---: | --- |
| Code file tokens | 8,000 | 16,000 | Approximate context size of one source file |
| Largest function or method | 150 LOC | 300 LOC | One independently understood executable unit |
| Functions plus classes | 25 | 50 | Independently navigable definitions in a code file |
| Direct internal dependencies | 10 | 20 | Distinct project-owned modules directly imported or referenced |
| Document tokens | 6,000 | 12,000 | Approximate context size of one document |
| Document LOC | 500 | 1,000 | A second signal for document size |
| Largest section tokens | 2,500 | 5,000 | One heading-delimited document topic |
| Largest section LOC | 200 | 400 | A second signal for one document topic |

These are defaults only. Repository configuration wins. Do not invent a whole-code-file LOC hard limit. A metric unavailable in the repository's native parser or compiler should be marked unsupported, not fabricated.

Token counts may be deterministic estimates. The implementation must name the algorithm, apply it identically to current and baseline revisions, and avoid presenting an estimate as a tokenizer measurement.

## Baseline and severity

The scanner needs to distinguish a new finding from historical debt.

- A new file above a warning threshold is WARN; above a hard threshold is FAIL.
- A file already above warning is a regression FAIL only when the current value is more than 20% above baseline and also reaches the metric-specific minimum absolute increase: +1,000 file tokens, +30 largest-function LOC, +5 definitions, +3 dependencies, +100 document tokens or LOC, or +100 section tokens or LOC.
- The same rule applies to a pre-existing hard-limit file: unchanged historical debt may remain WARN, while meaningful further growth is FAIL.
- A current value beyond hard without a historical exemption is FAIL.
- Missing baseline entries are treated as new files, not as permission to ignore them.
- Parse or structural-analysis errors are FAIL because the result is not trustworthy.

Baseline updates are explicit maintenance operations. A check must never rewrite the baseline. A CI job must never auto-refresh it, create an ignore, or raise a threshold to make the check pass. During historical cleanup, update the baseline only once the complete, approved cleanup acceptance is finished.

WARN exits with status 0. FAIL exits with a non-zero status. A repository may add other operational failures, but it must not turn a context FAIL into success.

## Progressive-disclosure output

The scanner is itself read by coding agents, so output volume is part of the contract.

Default output contains only:

1. total files scanned;
2. PASS, WARN, and FAIL counts;
3. up to 20 FAIL file names with short triggering metrics;
4. up to 10 WARN file names, preferably ordered by severity or context cost;
5. an `... N more` marker when lists are truncated;
6. the commands used to expand one file, all details, or the top hotspots.

Default output must not print a long reason, required-action, or avoid section for every file. Multiple problems on one file are aggregated into one entry. Use stable path and metric ordering so output is reviewable and suitable for logs.

The interaction contract is:

- `context-budget` — compact repository summary;
- `context-budget --detail <file>` — full diagnosis for the selected file only;
- `context-budget --top N` — bounded hotspot list for triage and historical cleanup;
- `context-budget --details` — explicit full diagnostics for all findings.

The first version does not expose `--json`. Keep the internal result model serializable so a future machine-readable renderer can be added without changing scan semantics.

## Single-file diagnosis

When a file is explicitly expanded, aggregate all its findings and include:

- File and Status;
- Metric, Current, Baseline, Warning threshold, Hard threshold, and Change;
- Reason;
- Recommended investigation;
- Avoid.

The guidance for a code FAIL must direct the Agent to re-check responsibility boundaries and task context footprint. It must not prescribe splitting merely because a file is large. A document section finding should identify the topic and its navigation impact.

## CI integration

CI is an invocation surface, not a second implementation. The provider-specific job should install the repository using its existing mechanism, select the declared workspace scope, and invoke the same deterministic local command. Keep warnings visible but non-blocking; make hard or regression failures blocking. Use the repository's normal runner, shell-neutral package scripts, cache policy, and trigger paths. Never make a CI pass by mutating baseline, ignore, or threshold state.
