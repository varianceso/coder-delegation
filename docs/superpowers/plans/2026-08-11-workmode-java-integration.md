# Workmode and Java Standards Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans (inline execution is authorized by the user). Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade coder-delegation so its Coder, Architect, and independent reviewer contracts align with the supplied work mode and Java backend coding standard while retaining compatibility with legacy v1 tasks.

**Architecture:** Treat the existing v2 evidence-driven design as the protocol base. Add a normative work-mode reference for role mapping, blind review, validation allowlists, dispatch, archive ownership, retry fallback, and reviewer hard constraints. Add a Java backend reference that can be selected by task scope. Replace the v1 line-oriented task contract with bounded implementation units and explicit dispute handling, then synchronize the entry skill, templates, docs, and manifests.

**Tech Stack:** Markdown protocol files, JSON plugin manifests, Node.js built-in JSON parsing, ripgrep, read-only Git validation.

---

### Task 1: Add normative work-mode and Java references

**Files:**
- Create: `skills/coder-delegation/references/workmode-protocol.md`
- Create: `skills/coder-delegation/references/java-backend-standard.md`
- Create: `skills/coder-delegation/references/evidence-dispute-arbitration.md`
- Create: `skills/coder-delegation/templates/coder-disputes.md`

- [ ] Define the work-mode roles, reviewer roster, A/B review modes, double-blind rules, validation allowlist, retry fallback, commit cap, dispatch summary, and archive ownership in one normative reference.
- [ ] Translate the supplied Java rules into a task-selectable reference covering naming, method/parameter limits, formatting, layering, persistence, exceptions, logging, comments, and MySQL.
- [ ] Define the v2 dual-truth, evidence anchors, dispute classifications, dependency-aware pause, ruling state machine, absolute-path rules, and v1 compatibility.
- [ ] Add the machine-readable dispute ledger template with evidence, ruling, task-version, and resume fields.

### Task 2: Replace Coder constraints and task contract

**Files:**
- Modify: `skills/coder-delegation/references/coder-hard-constraints.md`
- Modify: `skills/coder-delegation/references/coder-prompt-spec.md`
- Modify: `skills/coder-delegation/templates/coder-task.md`
- Modify: `skills/coder-delegation/templates/codebase-scan-summary.md`
- Modify: `skills/coder-delegation/templates/prd-code-mapping.md`

- [ ] Replace mechanical-only constraints with v2 bounded-autonomy constraints, including evidence, disputes, resume gates, validation allowlists, Java-reference selection, process cleanup, and reviewer boundaries.
- [ ] Define implementation units with `BOUNDED_AUTONOMY`, `EXACT_CHANGE`, and `INVESTIGATE_FIRST`; prohibit line-only instructions and require absolute native paths in actual dispatch documents.
- [ ] Make scan and PRD mapping templates distinguish direct facts, reproduced results, inferences, and unverified claims.
- [ ] Include standard delivery-report sections and a copyable summary limited to 200 Chinese characters.

### Task 3: Upgrade orchestration, review bridge, and user-facing docs

**Files:**
- Modify: `skills/coder-delegation/SKILL.md`
- Modify: `skills/coder-delegation/references/review-ac-bridge.md`
- Modify: `skills/coder-delegation/references/scan-delegation.md`
- Modify: `README.md`
- Modify: `AGENTS.md`
- Modify: `docs/spec-architect-code.md`
- Modify: `docs/changelog.md`

- [ ] Route design through superpowers, permit Claude/Architect to implement only when the active work mode assigns it, and keep Coder/reviewer write boundaries explicit.
- [ ] Add modes A/B, codex/opencode/mico/zcode mappings, double-blind handling, no-self-QA rule, archive paths, allowlist-only validation, and codex-to-zcode fallback.
- [ ] Make Architect review independent of Coder reports and block unresolved disputes or reviewer hard-constraint violations.
- [ ] Synchronize the user workflow, file names, version, and documented incompatibilities.

### Task 4: Update plugin metadata and validate

**Files:**
- Modify: `marketplace.json`
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`
- Modify: `.codex-plugin/plugin.json`
- Modify: `.agents/plugins/marketplace.json`

- [ ] Bump all plugin metadata to v2.0.0 and describe evidence-driven execution, work-mode reviewer support, and Java task guidance.
- [ ] Parse every JSON manifest with Node.js.
- [ ] Check all references/templates named by the main skill exist, all paths are consistent, and no old v1-only wording remains in the v2 entry points.
- [ ] Report the final diff, validation results, and any intentionally preserved user changes without Git write operations.
