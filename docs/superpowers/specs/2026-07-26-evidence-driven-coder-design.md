# Evidence-Driven Coder Design

> Date: 2026-07-26
> Status: Approved
> Target release: coder-delegation v2.0.0

## 1. Purpose

`coder-delegation` currently gives Coder a machine-readable patch script with precise anchors, actions, and complete code blocks. Although v1.2 requires a PRD + technical design + task document + code comparison, Coder is still defined as a mechanical executor: it has little room to reason from the repository, and any mismatch generally causes a full stop without a structured evidence and arbitration loop.

v2.0 changes Coder into an **evidence-driven, bounded implementation agent**. Coder must treat the checked-out repository as the authority for the system's current state, use user-approved requirements as the authority for the desired state, make local implementation decisions inside an explicit envelope, and escalate genuine conflicts through an auditable Architect arbitration protocol.

## 2. Goals

1. Give Coder enough autonomy to make local implementation decisions from verified repository evidence.
2. Prevent Coder from blindly following stale or ambiguous Architect instructions.
3. Prevent Coder from using current code behavior to silently override approved product intent.
4. Preserve the Architect boundary: Architect verifies facts and arbitrates, but does not invent a replacement technical design outside the superpowers workflow.
5. Pause only the conflicting execution branch while allowing provably independent work to continue.
6. Record conflicts, evidence, decisions, task revisions, and resumptions in files that survive independent agent sessions.
7. Ask the user interactively only when a decision affects product meaning, acceptance criteria, public contracts, scope, or risk.
8. Produce a cross-platform, directly copyable Coder dispatch prompt containing only an execution instruction and a native absolute task-document path.

## 3. Non-Goals

- Coder does not become a product owner or system architect.
- Coder may not expand scope, public contracts, dependencies, persistence semantics, or user-visible behavior without approval.
- Architect does not replace `superpowers:brainstorming` or `superpowers:writing-plans` when an approved design must change.
- The protocol does not depend on real-time cross-agent messaging. Direct messaging may accelerate notification, but files remain authoritative.
- The protocol does not require Coder to expose private chain-of-thought. It requires concise, auditable decision summaries only.

## 4. Role Model

### 4.1 Architect

Architect remains the strong-model coordinator. It:

- reuses the approved superpowers technical design and implementation plan;
- verifies code facts before producing the task contract;
- declares Coder's autonomy envelope and escalation boundaries;
- independently verifies evidence submitted by Coder;
- resolves factual and in-envelope implementation disputes;
- asks the user for product, contract, scope, and risk decisions;
- routes design-changing conflicts back through superpowers;
- updates the task version and decision record before Coder resumes;
- reviews the final diff, tests, decisions, and closed-dispute state.

Architect still does not write business code and does not independently invent a new technical design.

### 4.2 Coder

Coder becomes an evidence-driven, bounded implementation agent. It:

- reads the product sources, technical design, task contract, relevant source code, tests, configuration, schema, and dependency metadata;
- distinguishes verified facts from instructions and inference;
- chooses local implementation details inside the declared envelope;
- records concise decision summaries with evidence anchors;
- reports ambiguity or contradiction instead of silently choosing a new product meaning;
- pauses the conflicting unit and its dependents while continuing only provably independent units;
- resumes affected work only after reading a newer task version with an explicit ruling;
- self-tests, self-reviews, and reports without disguising unresolved conflicts or failures.

## 5. Dual-Truth Model

The workflow uses two complementary authorities rather than a single global source of truth.

### 5.1 Desired-State Authority

The desired behavior is determined by, in order:

1. explicit user decisions recorded during the workflow;
2. user-approved acceptance criteria;
3. the approved PRD or requirement specification;
4. the approved technical design and implementation plan, provided they do not contradict higher desired-state authorities.

These sources answer: **What must the completed system do?**

### 5.2 Current-State Authority

The current system state is established using repository evidence:

1. checked-out source signatures, call paths, schema, migrations, configuration, and dependency versions;
2. reproducible build, test, static-analysis, or runtime output;
3. project-local usage examples and generated interfaces;
4. version-control history when it explains a verified current construct;
5. comments, README content, historical design documents, and delegated scan summaries as supporting evidence only.

These sources answer: **What does the system currently contain or do?**

Source code is not automatically proof that existing behavior is correct. A disagreement between source, tests, schema, or runtime output is itself evidence that requires reconciliation.

### 5.3 Instruction Status

The technical design, implementation plan, and Coder task document are executable instructions, not unquestionable facts. Architect must verify their repository claims before dispatch, and Coder must re-verify relevant claims against its actual checkout before editing.

No source may silently override the other truth domain:

- current code may not silently override approved product intent;
- product intent may not justify pretending that nonexistent APIs, paths, or data flows already exist;
- task instructions must be revised when they conflict with verified facts or approved intent.

## 6. Evidence Model

Every material Coder decision or dispute uses evidence anchors. An anchor contains:

- a native path accessible in the Coder environment;
- a stable symbol, type signature, test name, configuration key, schema object, or command;
- the observed fact;
- whether the observation is direct, reproduced, inferred, or still unverified.

Line numbers may be included for convenience but are not sufficient alone because they become stale. Method names, signatures, call relationships, and observable test behavior are preferred.

Delegated scan summaries are navigation aids, not final proof. Architect and Coder must reopen decisive code or reproduce decisive commands before treating a claim as verified.

## 7. Execution Modes

Each implementation unit declares exactly one execution mode.

### 7.1 `BOUNDED_AUTONOMY`

This is the default mode for business implementation. Coder may choose local implementation details while preserving the unit's objectives, invariants, allowed files, public contracts, dependencies, and acceptance checks.

### 7.2 `EXACT_CHANGE`

This mode is used for security-sensitive constants, exact protocol content, migrations, generated compatibility shims, or legacy v1.x task steps. Coder follows the specified change exactly and escalates any mismatch instead of adapting it silently.

### 7.3 `INVESTIGATE_FIRST`

This mode is used when Architect cannot completely verify an accessible area before dispatch. Coder first gathers the required evidence. It may proceed only if the findings match the task's declared assumptions; otherwise it opens a dispute.

## 8. Bounded Autonomy

Inside a `BOUNDED_AUTONOMY` unit, Coder may decide:

- private helper naming and decomposition;
- internal control flow and equivalent guard placement;
- reuse of existing project-local helpers and abstractions;
- equivalent null, error, and validation handling that preserves existing contracts;
- test organization, fixtures, test data, and assertion structure;
- local refactoring required to implement the unit without changing observable behavior;
- updated local anchors when line numbers moved but the intended symbol and semantics are unchanged.

Coder must escalate before:

- changing a public method, endpoint, event, DTO, error, or serialization contract;
- modifying a file outside the allowed file set;
- adding, replacing, or calling an undeclared external dependency, service, SDK, or data source;
- changing schema, migration, persistence, transactional, or data-retention semantics;
- changing user-visible behavior or acceptance criteria;
- replacing an approved architectural approach;
- accepting a security, compatibility, data-loss, or irreversible-operation risk;
- continuing when verified evidence shows that an approved design assumption cannot hold.

## 9. Conflict Classification

| Type | Meaning | Default Owner |
|------|---------|---------------|
| `LOCAL_CHOICE` | Multiple equivalent implementations exist inside the autonomy envelope | Coder decides and records |
| `FACT_MISMATCH` | A task claim about files, symbols, APIs, call paths, tests, or configuration conflicts with repository evidence | Architect verifies and decides |
| `PLAN_AMBIGUITY` | The task contract has multiple materially different interpretations | Architect decides if the approved design already determines the answer; otherwise route appropriately |
| `DESIGN_CONFLICT` | The approved technical design cannot be implemented against verified current architecture without changing the design | Return to superpowers |
| `PRODUCT_AMBIGUITY` | Product meaning or acceptance behavior is unclear | User decides interactively |
| `CONTRACT_OR_SCOPE` | Resolution changes public contracts, allowed files, dependencies, persistence, or task scope | User decides interactively after Architect analysis |
| `SAFETY_RISK` | Resolution accepts security, compatibility, migration, data-loss, or irreversible risk | Stop affected work and ask the user |

An environment or test failure is reported separately as an execution blocker unless it also exposes one of the conflict types above.

## 10. Dependency-Aware Pause

When Coder discovers a conflict:

1. identify the directly affected implementation unit;
2. identify downstream units whose correctness depends on the disputed result;
3. pause the affected unit and all downstream units;
4. continue only units whose inputs, outputs, files, and acceptance checks are demonstrably independent;
5. record why each continued unit is independent;
6. do not mark the overall task complete while any delivery-affecting dispute remains open.

When dependency independence cannot be demonstrated, Coder pauses the remaining task rather than guessing.

## 11. Dispute Artifact

The authoritative dispute file is stored beside the Coder task and report. Its generated name follows:

`coder-disputes-{task-name}.md`

The task document contains its native absolute path. Each dispute uses a stable identifier such as `D-001` and contains:

- state;
- classification;
- affected and downstream units;
- task-document claim;
- observed repository fact;
- evidence anchors and reproduction commands;
- Coder's concise analysis and recommendation;
- paused and safely continued scope;
- expected impact of each plausible resolution;
- Coder's recommendation on whether user input is needed;
- Architect verification result;
- final ruling and owner;
- task revision that incorporated the ruling;
- resume and closure record.

Coder's recommendation is advisory. Architect owns classification confirmation and routing.

## 12. Arbitration State Machine

```text
DISCOVERED
  -> EVIDENCE_READY
  -> ARCHITECT_VERIFYING
  -> ARCHITECT_DECIDED
     or USER_INPUT_REQUIRED -> USER_DECIDED
     or PLAN_REVISION_REQUIRED -> superpowers revision approved
  -> TASK_UPDATED
  -> RESUMED
  -> CLOSED
```

State rules:

- Coder may create `DISCOVERED` and advance to `EVIDENCE_READY`.
- Architect advances the item to `ARCHITECT_VERIFYING` and independently reproduces decisive evidence.
- Architect records evidence as confirmed, partially confirmed, or rejected.
- Architect may issue `ARCHITECT_DECIDED` only for verified facts or choices already bounded by the approved design.
- Architect uses `USER_INPUT_REQUIRED` for product meaning, acceptance behavior, public contracts, scope, dependencies, persistence, and material risk.
- Architect uses `PLAN_REVISION_REQUIRED` when the approved technical design must change, then returns to `superpowers:brainstorming` and `superpowers:writing-plans` as appropriate.
- The task document version must increase before `TASK_UPDATED`.
- Coder may resume affected units only from the task revision named in the ruling.
- Architect closes the dispute only after verifying the ruling was implemented and tested.

## 13. Interactive User Decisions

When user input is required, Architect asks one decision at a time. The question must include:

- a concise description of the conflict;
- the verified evidence;
- two or three mutually exclusive options when practical;
- behavior, compatibility, scope, and risk impact for each option;
- Architect's recommendation and rationale.

Architect records the user's answer as a desired-state authority in both the task decision log and dispute item. Coder never asks the user directly unless the runtime forces the Coder and Architect roles into the same session and the role boundary remains explicit.

## 14. Task Contract v2

The v2 task document is an implementation contract rather than a line-oriented patch script. It contains:

1. document state: protocol version, task revision, baseline commit, repository root, task path, report path, dispute path, status, and open/closed dispute identifiers;
2. desired-state sources: PRD, acceptance criteria, and user decisions;
3. verified current-state facts: actual entry points, call paths, data flow, tests, configuration, schema, and dependency versions;
4. autonomy envelope: allowed files, allowed choices, invariants, forbidden changes, and escalation triggers;
5. implementation units: execution mode, objective, dependencies, evidence anchors, required behavior, optional implementation hints, and acceptance checks;
6. self-test commands and success criteria;
7. conflict, pause, arbitration, version, and resume protocol;
8. self-review checklist;
9. report requirements;
10. append-only Architect decision log.

For `BOUNDED_AUTONOMY`, a complete code block is optional and explicitly non-binding unless it encodes an invariant. For `EXACT_CHANGE`, the exact patch content remains required. For `INVESTIGATE_FIRST`, the evidence questions and proceed/stop conditions are required.

## 15. Decision Summaries

Coder records concise decision summaries for nontrivial `LOCAL_CHOICE` decisions:

```markdown
- Decision: Reuse the existing `parseUserId` helper.
- Evidence: The same module uses it at three existing call sites and its tests define the required error behavior.
- Rationale: This preserves the current error contract and avoids duplicate parsing logic.
- Boundary check: No public contract, dependency, persistence behavior, or allowed-file boundary changed.
```

These summaries are audit records, not requests for private chain-of-thought.

## 16. Cross-Platform Absolute Paths

All cross-session artifact paths in the task document use paths that are native and accessible in the Coder environment.

- macOS and Linux use POSIX absolute paths such as `/Users/name/project/...` or `/home/name/project/...`.
- Windows uses drive-qualified paths such as `C:\Users\name\project\...` or UNC paths such as `\\server\share\project\...`.
- WSL uses its accessible POSIX namespace, such as `/mnt/c/Users/name/project/...`, rather than a Windows host path.
- Paths may not use `~`, environment-variable expansion, relative segments, `file://` URIs, unresolved template variables, or mixed separators.

Architect must detect or confirm the Coder environment, resolve the task path, and verify that the task file exists. If Architect and Coder do not share a filesystem, or the Coder environment is unclear, Architect asks the user for the target environment and accessible path rather than guessing.

## 17. Copyable Dispatch Prompt

After task approval, Architect outputs only one standalone code block containing an execution instruction and the native absolute task-document path. It does not repeat the working directory, branch, report path, constraints, or explanatory prose because those values are authoritative inside the task document.

macOS/Linux example:

```text
请读取并执行编码任务文档：/Users/name/project/.claude/user-auth/coder/coder-task-user-auth.md
```

Windows example:

```text
请读取并执行编码任务文档：C:\Users\name\project\.claude\user-auth\coder\coder-task-user-auth.md
```

WSL example:

```text
请读取并执行编码任务文档：/mnt/c/Users/name/project/.claude/user-auth/coder/coder-task-user-auth.md
```

Actual output must never contain example values or unresolved placeholders.

## 18. Final Review Changes

Architect's final review additionally verifies:

- every modified file is inside the final autonomy envelope;
- each nontrivial autonomous choice has an evidence-backed decision summary;
- no Coder decision crossed a mandatory escalation boundary;
- every dispute affecting delivery reached `CLOSED`;
- the implementation used the task revision named by each ruling;
- tests cover the approved resolution;
- the report does not claim completion while blockers or open disputes remain.

The existing BLOCKER/HIGH and AC bridge thresholds remain unchanged. An unresolved delivery-affecting dispute is a BLOCKER regardless of diff size.

## 19. Backward Compatibility

- Existing v1.x task documents remain executable as legacy `EXACT_CHANGE` tasks.
- An already dispatched v1.x task is not silently rewritten.
- Migrating an in-flight task creates a v2 task revision, records the migration decision, and requires renewed user confirmation before dispatch.
- Reports from legacy tasks may omit v2 decision and dispute sections, but Architect must not describe them as v2 evidence-driven executions.
- The plugin remains documentation- and protocol-based with no third-party runtime dependency.

## 20. Repository Changes

### 20.1 Create

- `skills/coder-delegation/references/evidence-dispute-arbitration.md`
- `skills/coder-delegation/templates/coder-disputes.md`
- `docs/superpowers/specs/2026-07-26-evidence-driven-coder-design.md`

### 20.2 Modify

- `AGENTS.md`
- `README.md`
- `docs/spec-architect-code.md`
- `docs/changelog.md`
- `skills/coder-delegation/SKILL.md`
- `skills/coder-delegation/references/coder-hard-constraints.md`
- `skills/coder-delegation/references/coder-prompt-spec.md`
- `skills/coder-delegation/references/scan-delegation.md`
- `skills/coder-delegation/references/review-ac-bridge.md`
- `skills/coder-delegation/templates/coder-task.md`
- `skills/coder-delegation/templates/codebase-scan-summary.md`
- `skills/coder-delegation/templates/prd-code-mapping.md`
- `marketplace.json`
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `.codex-plugin/plugin.json`
- `.agents/plugins/marketplace.json`

`skills/cdel/SKILL.md` remains a thin alias and requires no behavioral duplication.

## 21. Versioning

All plugin manifests and protocol documents move to v2.0.0 because the Coder role and task contract have breaking semantic changes. Descriptions must remove claims that Coder executes mechanically or does not think, and replace them with the evidence-driven bounded-autonomy model.

## 22. Acceptance Criteria

The implementation is accepted when all of the following are true:

1. Coder can make documented local implementation choices inside a declared autonomy envelope.
2. Coder must use verified repository evidence rather than blindly following stale task claims.
3. Current-state evidence cannot silently override user-approved desired behavior.
4. Conflicting units and downstream units pause while demonstrably independent units may continue.
5. Every material conflict is recorded in a structured dispute artifact.
6. Architect independently verifies decisive evidence before ruling.
7. Product, acceptance, public-contract, scope, dependency, persistence, and material-risk decisions require interactive user confirmation.
8. Design-changing conflicts return to superpowers instead of being redesigned ad hoc by Architect or Coder.
9. Affected work resumes only from a newer task revision containing an explicit ruling.
10. Open delivery-affecting disputes prevent completion.
11. Legacy v1.x tasks remain usable through `EXACT_CHANGE` compatibility.
12. The dispatch prompt is directly copyable and contains only the execution instruction plus a verified native absolute path.
13. macOS, Linux, Windows, UNC, and WSL path forms are distinguished without hard-coded assumptions.
14. All new references and templates are linked from the main skill.
15. All manifest versions and user-facing descriptions agree on v2.0.0 behavior.

## 23. Validation

Implementation validation uses the repository's zero-dependency constraints:

- run `git diff --check`;
- parse every JSON manifest with Node.js `JSON.parse`;
- search for stale phrases that define Coder as a mechanical executor or prohibit all implementation judgment;
- verify the main skill references every new reference and template;
- verify task and dispatch templates prohibit relative, home-relative, URI, mixed-separator, and unresolved paths;
- verify the Coder checklist covers evidence verification, autonomy boundaries, decision summaries, dispute closure, tests, file scope, contracts, dependencies, sensitive output, and process cleanup;
- verify the Architect review covers independent evidence verification and unresolved-dispute blocking;
- verify the documented version and changelog entries are consistent across manifests, specifications, references, and templates.
