# Context-Budget Refactoring Playbook

Use this reference for `remediate` and `cleanup`. Metrics are evidence for investigation. The acceptance question is whether common Agent tasks need less relevant context after the change.

## 1. Diagnose change vectors

Before editing a hotspot, ask: “What independent reasons can make this module change?” Name the vectors in plain language. Typical vectors include:

- orchestration or command handling;
- domain rules;
- persistence or other I/O;
- validation;
- parsing or conversion;
- serialization or formatting;
- integration adapters.

An apparent cluster of functions is not automatically a separate responsibility. Composition roots, dispatchers, routers, registries, schemas/contracts, and public facades can be intentionally high-definition or high-dependency modules.

Use imports, call counts, and Git co-change as prompts for investigation, not as architectural verdicts. A cohesive module with many dependencies may be the correct composition point.

## 2. Accept or reject an extraction

Accept an extracted module only when all of these are true:

1. The responsibility has a clear name and one owner.
2. It has its own change vector and can be understood, changed, or tested independently.
3. Dependencies flow in one direction without a cycle or bidirectional ownership.
4. It exposes no more than five operational public entry points by default. A schema, model, registry, or intentionally public API can be an exception.
5. It is not a thin wrapper, forwarding layer, compatibility shell, or metric-only helper.
6. Representative task context footprint improves under the acceptance test below.

Do not create `utils`, `helpers`, `common`, or `misc` modules without explicit ownership. Do not move a cohesive responsibility into several small files just to lower a threshold.

## 3. Task Context Footprint acceptance

For a substantial structural change, choose at least three realistic, representative modification scenarios from the repository's maintenance work. Examples include changing a command flow, changing one domain rule, and changing persistence or formatting behavior. Use the same scenarios before and after the change.

For each scenario, record:

- files that must be read;
- working-set tokens or the repository's equivalent context estimate;
- modules or directories crossed;
- new navigation jumps or searches;
- dependency direction and the number of relevant edges.

Accept by default when the median working-set tokens falls by at least 20%, or when the median number of required files falls by at least one and working-set tokens grow by no more than 10%. Reject when common tasks require more than one additional file and tokens grow by more than 10%, a cycle or bidirectional dependency appears, an originally cohesive responsibility is scattered, or many forwarding calls are added.

If footprint cannot be measured precisely, use a transparent, repeatable estimate and report the uncertainty. Do not claim an architectural improvement from scanner color alone.

## 4. Long functions

Extract only a complete step or rule with:

- a semantic name;
- explicit inputs and outputs;
- a boundary that can be understood or tested independently.

If the extracted call needs more than five business inputs or depends on a large shared mutable state, treat that as evidence that the proposed boundary is wrong. First reconsider ownership or simplify the original flow. Do not replace one long function with a chain of trivial wrappers.

## 5. Coupling and high-cost cohesive modules

Direct internal dependency count is a warning signal. Inspect whether each dependency belongs at the current boundary and whether a dependency is accidental. Remove redundant structure or move a rule to its true owner when that reduces coupling without adding navigation.

Do not refactor a composition root, dispatcher, router, registry, schema/contract module, or public facade merely because it has more than ten imports. These modules may be the shortest path for a task. A split needs an independent responsibility and footprint evidence.

## 6. Historical evidence

When enough Git history exists, use temporal coupling only as supporting evidence:

- inspect the most recent 12 months;
- use at most 200 relevant commits;
- ignore bulk or codemod commits touching more than 30 files;
- if two areas share fewer than five commits, draw no coupling conclusion;
- calculate `shared_commits / min(commits_A, commits_B)`;
- treat a result of at least 0.5 as a strong investigation signal, not proof of a split.

Without sufficient history, skip this analysis and say so. Co-change alone never determines architecture.

## 7. Historical cleanup loop

Process one independent hotspot at a time, in this order:

1. hard limit or FAIL;
2. files with multiple WARN metrics;
3. high context cost combined with meaningful maintenance churn;
4. ordinary WARN.

For each hotspot:

1. establish a test baseline;
2. identify change vectors and ownership;
3. select three representative scenarios;
4. record pre-change footprint;
5. design the smallest structural or simplifying change;
6. implement it and run relevant tests immediately;
7. compare context metrics and post-change footprint;
8. check dependency direction, public entry points, and behavior/API compatibility;
9. accept the change or restore the cohesive design and record why the warning remains;
10. only then select the next hotspot.

The cleanup target is code FAIL = 0, not WARN = 0. Keep a highly cohesive warning when further splitting would increase the context needed for normal maintenance. Do not update the final baseline until the complete cleanup scope has passed its acceptance checks.

## 8. Documentation

Documentation can be split more readily when it contains independently readable topics. Split by subject, keep a concise parent router, give child documents semantic names, and keep the parent-to-target path to at most two navigations. Keep each fact in one authoritative location and link to it elsewhere. Do not create numbered parts, miscellaneous fragments, or overlapping SOPs. The same footprint test applies: the split should reduce information-addressing cost for real tasks.

## 9. Expected decisions for adversarial cases

- An 18,000-token, highly cohesive orchestrator: retain it unless an independently owned responsibility and footprint improvement are demonstrated.
- A 10,000-token file with more than 50 definitions and several change vectors: investigate an ownership-based split; do not split on tokens alone.
- One file replaced by five mutually calling files: reject because navigation and bidirectional coupling increase.
- A composition root with 15 or more imports: retain it unless an import is misplaced and the proposed boundary improves task footprint.
- A 14,000-token Markdown file with distinct topics: split by topic and leave a concise router.
