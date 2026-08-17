# Evidence-Driven Coder v2.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade coder-delegation to v2.0.0 so Coder reasons from verified repository evidence inside an explicit autonomy envelope, records disputes, and resumes only after Architect or user arbitration.

**Architecture:** Replace the v1.x line-oriented patch-script contract with a file-backed implementation contract. Add one normative evidence/arbitration reference and one dispute template, then update Coder constraints, task specification, orchestration, review, user documentation, and plugin metadata around the same state machine. Keep legacy v1.x tasks executable as `EXACT_CHANGE` and keep files as the cross-runtime source of truth.

**Tech Stack:** Markdown skill protocol, JSON plugin manifests, Git read-only validation, Node.js built-in `fs`/`JSON.parse`, ripgrep.

**Normative design:** `docs/superpowers/specs/2026-07-26-evidence-driven-coder-design.md`

**Repository constraint:** Do not run `git add`, `git commit`, `git push`, or other Git write operations. Commit steps are intentionally omitted because this repository's Coder contract forbids Git writes and the user did not request commits.

---

## File Responsibility Map

| File | Responsibility after v2.0 |
|------|---------------------------|
| `skills/coder-delegation/references/evidence-dispute-arbitration.md` | Normative dual-truth, evidence, dispute, pause, arbitration, and user-interaction protocol |
| `skills/coder-delegation/templates/coder-disputes.md` | Machine-readable dispute ledger and ruling/resume template |
| `skills/coder-delegation/references/coder-hard-constraints.md` | Non-negotiable Coder boundaries and v2 self-review checklist |
| `skills/coder-delegation/references/coder-prompt-spec.md` | Normative Coder task-contract v2 schema |
| `skills/coder-delegation/templates/coder-task.md` | Fillable v2 task, report, decision-log, and absolute-path template |
| `skills/coder-delegation/references/scan-delegation.md` | Evidence quality and delegated-scan limitations |
| `skills/coder-delegation/templates/codebase-scan-summary.md` | Fact/inference/unverified evidence capture |
| `skills/coder-delegation/templates/prd-code-mapping.md` | Desired-state/current-state mapping and contradictions |
| `skills/coder-delegation/SKILL.md` | Architect orchestration, dispatch, arbitration, resume, and review flow |
| `skills/coder-delegation/references/review-ac-bridge.md` | Final review rules, unresolved-dispute BLOCKER, unchanged AC thresholds |
| `AGENTS.md` | Repository-wide role and behavioral instructions |
| `README.md` | User-facing v2 workflow and platform-specific copyable prompts |
| `docs/spec-architect-code.md` | Consolidated implemented v2 protocol specification |
| `docs/changelog.md` | v2.0.0 breaking-change and compatibility record |
| Plugin manifests | v2.0.0 metadata and evidence-driven positioning |

### Task 1: Add Evidence and Arbitration Protocol

**Files:**
- Create: `skills/coder-delegation/references/evidence-dispute-arbitration.md`
- Create: `skills/coder-delegation/templates/coder-disputes.md`
- Reference: `docs/superpowers/specs/2026-07-26-evidence-driven-coder-design.md:55`

- [ ] **Step 1: Verify the new protocol files do not already exist**

Run:

```bash
test ! -e skills/coder-delegation/references/evidence-dispute-arbitration.md \
  && test ! -e skills/coder-delegation/templates/coder-disputes.md
```

Expected: exit status `0` with no output.

- [ ] **Step 2: Create the normative evidence/arbitration reference**

Create `skills/coder-delegation/references/evidence-dispute-arbitration.md` with this exact top-level structure:

```markdown
# 证据、分歧与裁决协议

- 版本：v2.0.0 (2026-07-26)
- 适用：Architect 与 Coder
- 原则：文件记录为跨 session 权威状态；实时消息只作通知

## 1. 双真相模型
## 2. 证据等级与锚点
## 3. 执行模式
## 4. 受约束自主边界
## 5. 分歧分类
## 6. 依赖感知暂停
## 7. 分歧单必填字段
## 8. 裁决状态机
## 9. Architect 核实与裁决
## 10. 用户交互边界
## 11. 任务版本与恢复门
## 12. 跨平台绝对路径
## 13. 兼容 v1.x
## 版本记录
```

Populate those sections from design §§5–13 and §§16–19. Include these exact normative definitions:

```markdown
| 类型 | 默认处理 |
|------|----------|
| `LOCAL_CHOICE` | Coder 在自主边界内决定并写决策摘要 |
| `FACT_MISMATCH` | 暂停受影响分支，Architect 独立核实 |
| `PLAN_AMBIGUITY` | Architect 按已批准方案裁决；方案未覆盖则升级 |
| `DESIGN_CONFLICT` | 返回 superpowers 修订方案与执行计划 |
| `PRODUCT_AMBIGUITY` | Architect 整理证据与选项后询问用户 |
| `CONTRACT_OR_SCOPE` | 必须询问用户 |
| `SAFETY_RISK` | 停止相关工作，必须询问用户 |
```

```text
DISCOVERED
  → EVIDENCE_READY
  → ARCHITECT_VERIFYING
  ├→ ARCHITECT_DECIDED
  ├→ USER_INPUT_REQUIRED → USER_DECIDED
  └→ PLAN_REVISION_REQUIRED → superpowers 修订并获批
  → TASK_UPDATED
  → RESUMED
  → CLOSED
```

State that line numbers are supporting anchors only; symbols, signatures, call relations, tests, schema objects, configuration keys, dependency versions, and reproduced command output are primary evidence. State that delegated summaries never replace independent verification for a decisive ruling.

- [ ] **Step 3: Create the dispute ledger template**

Create `skills/coder-delegation/templates/coder-disputes.md` with this exact template shape:

```markdown
# Coder 分歧单：{{任务名}}

> 协议版本：v2.0.0
> 任务文档：`{{Coder 环境可访问的原生绝对路径}}`
> 当前任务版本：{{vN}}
> 分歧单路径：`{{Coder 环境可访问的原生绝对路径}}`

## 状态摘要

| ID | 类型 | 状态 | 影响单元 | 裁决者 | 纳入任务版本 |
|----|------|------|----------|--------|--------------|
| D-001 | {{FACT_MISMATCH}} | {{EVIDENCE_READY}} | {{Unit 2, Unit 4}} | {{待定}} | {{待定}} |

## D-001

- **状态**：`EVIDENCE_READY`
- **类型**：`{{LOCAL_CHOICE / FACT_MISMATCH / PLAN_AMBIGUITY / DESIGN_CONFLICT / PRODUCT_AMBIGUITY / CONTRACT_OR_SCOPE / SAFETY_RISK}}`
- **发现于任务版本**：`{{vN}}`
- **影响单元**：{{直接受影响单元}}
- **下游单元**：{{依赖该结果的单元}}
- **任务文档主张**：{{精确描述}}
- **代码事实**：{{精确描述}}
- **证据锚点**：
  - `{{原生绝对路径}}` — `{{符号/签名/配置键/测试名}}`：{{观察事实}}
- **复现命令与结果**：
  - `{{只读命令}}` → {{关键结果}}
- **Coder 分析摘要**：{{结论性摘要，不输出冗长思维过程}}
- **Coder 建议**：{{建议裁决}}
- **已暂停范围**：{{单元及原因}}
- **可继续范围**：{{独立单元及独立性依据}}
- **建议用户介入**：{{是/否 + 原因}}

### Architect 核实

- **核实状态**：{{CONFIRMED / PARTIALLY_CONFIRMED / REJECTED}}
- **独立核实证据**：{{Architect 重新读取或复现的证据}}
- **最终分类**：{{类型}}
- **路由**：{{ARCHITECT_DECIDED / USER_INPUT_REQUIRED / PLAN_REVISION_REQUIRED}}

### 裁决与恢复

- **裁决结果**：{{明确结果}}
- **裁决者**：{{Architect / User / superpowers-approved design}}
- **用户决定记录**：{{不需要则写“不适用”}}
- **纳入任务版本**：`{{vN+1}}`
- **恢复范围**：{{允许恢复的单元}}
- **关闭条件**：{{实现与测试要求}}
- **最终状态**：`{{TASK_UPDATED / RESUMED / CLOSED}}`
```

The double-braced values are intentional template syntax, not unfinished implementation notes.

- [ ] **Step 4: Validate protocol vocabulary and template completeness**

Run:

```bash
rg -n '双真相模型|BOUNDED_AUTONOMY|EXACT_CHANGE|INVESTIGATE_FIRST|FACT_MISMATCH|USER_INPUT_REQUIRED|PLAN_REVISION_REQUIRED|TASK_UPDATED|RESUMED|CLOSED|Windows|WSL' \
  skills/coder-delegation/references/evidence-dispute-arbitration.md \
  skills/coder-delegation/templates/coder-disputes.md
```

Expected: every execution mode, conflict type, state, and platform appears at least once in the appropriate file.

### Task 2: Replace Coder Mechanical Constraints

**Files:**
- Modify: `skills/coder-delegation/references/coder-hard-constraints.md:1`
- Reference: `skills/coder-delegation/references/evidence-dispute-arbitration.md:1`

- [ ] **Step 1: Capture the v1 assumptions before replacement**

Run:

```bash
rg -n '9 条硬约束|按文档|停止编码|技术方案自身问题.*不停止|所有执行步骤已按顺序完成' \
  skills/coder-delegation/references/coder-hard-constraints.md
```

Expected: matches show the v1.2 comparison and mechanical-execution wording.

- [ ] **Step 2: Rewrite the file as v2.0.0 constraints**

Replace the file with these exact numbered constraints and expand each with normative bullets from the approved design:

```markdown
# Coder 硬约束

- 版本：v2.0.0 (2026-07-26)
- 适用：coder-delegation 工作流中的 Coder
- 性质：违反任一条 = 不可交付

## 11 条硬约束

### 0. 编码前核实任务版本、目标真相与当前代码事实
### 1. 只在自主边界和允许文件内工作
### 2. 在边界内自主实现，不机械照抄提示
### 3. 不擅自改变接口、产品语义、持久化与风险承诺
### 4. 不调用未声明的外部服务或依赖
### 5. 关键自主决策必须记录证据与决策摘要
### 6. 分歧必须落盘并依赖感知暂停
### 7. 只从包含明确裁决的新任务版本恢复
### 8. 不执行 git 写操作
### 9. 编码完成后必须自测并如实报告
### 10. 必须完成自审、脱敏并关闭启动进程
```

Constraint 2 must allow private helpers, equivalent control flow, existing project-local utility reuse, tests, and local refactoring only when contracts and observable behavior stay unchanged.

Constraint 6 must require a dispute item for every material mismatch, pause affected/downstream units, and allow continuation only with a written independence justification.

Constraint 7 must reject verbal-only rulings: the dispute identifier and ruling must be present in a newer task revision.

Constraint 8 must retain the current prohibition on `git add`, `git commit`, `git push`, `git reset --hard`, and `git checkout --` while allowing read-only Git commands.

- [ ] **Step 3: Replace the self-review checklist with v2 checks**

Use this exact checklist:

```markdown
## 自审 Checklist（嵌入任务文档）

- [ ] 已确认执行的是任务文档最新版本，基线 commit 与当前工作区一致或差异已记录
- [ ] 已核实目标真相与当前代码事实，未把任务文档当成不可质疑事实
- [ ] 所有自主决策均在允许文件、接口、依赖、持久化和行为边界内
- [ ] 所有非平凡自主决策均有“决策 + 证据 + 原因 + 边界检查”摘要
- [ ] 所有分歧均已写入分歧单，影响交付的分歧全部为 `CLOSED`
- [ ] 仅执行了裁决明确允许恢复的单元，没有依据口头消息跳过任务版本门
- [ ] 编译、单测及任务文档要求的检查全部通过
- [ ] 未修改任务文档允许范围外的文件
- [ ] 未执行 git 写操作，输出已脱敏
- [ ] 启动的进程已全部关闭，所有可交付实现单元均已完成
```

- [ ] **Step 4: Validate that bounded autonomy replaced mechanical execution**

Run:

```bash
rg -n '边界内自主实现|决策摘要|依赖感知暂停|新任务版本|git 写操作|CLOSED' \
  skills/coder-delegation/references/coder-hard-constraints.md
```

Expected: all six concepts appear; the title reports v2.0.0 and 11 constraints.

### Task 3: Define Task Contract v2 and Template

**Files:**
- Modify: `skills/coder-delegation/references/coder-prompt-spec.md:1`
- Modify: `skills/coder-delegation/templates/coder-task.md:1`
- Reference: `skills/coder-delegation/templates/coder-disputes.md:1`

- [ ] **Step 1: Verify the old mandatory patch fields are present**

Run:

```bash
rg -n '每条步骤必须包含 5 个字段|insert_after|完整代码块|所有执行步骤已按顺序完成' \
  skills/coder-delegation/references/coder-prompt-spec.md \
  skills/coder-delegation/templates/coder-task.md
```

Expected: matches confirm the v1 patch-script contract.

- [ ] **Step 2: Rewrite the task specification around implementation units**

Replace `coder-prompt-spec.md` with this exact section contract:

```markdown
# Coder 编码任务文档规范

- 版本：v2.0.0 (2026-07-26)

## 1. 文档定位
## 2. 质量门
## 3. 必需章节
## 4. §0 文档状态与原生绝对路径
## 5. §1 目标真相
## 6. §2 已核实当前代码事实
## 7. §3 自主决策边界
## 8. §4 实现单元与三种执行模式
## 9. §5 自测要求
## 10. §6 分歧、暂停、裁决与恢复协议
## 11. §7 自审 Checklist
## 12. §8 交付报告要求
## 13. §9 Architect 追加式决策记录
## 14. v1.x `EXACT_CHANGE` 兼容
## 版本记录
```

Define these exact per-unit fields:

```markdown
- **单元 ID**：`Unit N`
- **执行模式**：`BOUNDED_AUTONOMY | EXACT_CHANGE | INVESTIGATE_FIRST`
- **依赖单元**：`无` 或明确的 Unit ID
- **允许文件**：Coder 环境可访问的原生绝对路径
- **实现目标**：可观察结果
- **必须保持的不变量**：接口、异常、数据、兼容性和安全边界
- **已核实证据锚点**：路径 + 符号/签名/测试/配置/Schema + 事实
- **允许自主决定**：本单元内的局部选择
- **必须升级裁决**：本单元特有升级条件
- **可选实现提示**：仅供参考；`EXACT_CHANGE` 除外
- **验收检查**：可直接运行的命令和明确结果
```

Require `EXACT_CHANGE` to contain exact action and exact content, and require `INVESTIGATE_FIRST` to contain evidence questions plus explicit proceed/stop conditions.

- [ ] **Step 3: Rewrite the Coder task template**

Replace `templates/coder-task.md` with this top-level structure:

```markdown
# Coder 编码任务：{{任务名}}

## 0. 文档状态
## 1. 目标真相
## 2. 已核实当前代码事实
## 3. 自主决策边界
## 4. 实现单元
## 5. 自测要求
## 6. 分歧、暂停、裁决与恢复协议
## 7. 自审 Checklist
## 8. 交付报告要求
## 9. Architect 决策记录
```

The §0 literal template fields must be:

```markdown
- **协议版本**：`v2.0.0`
- **任务版本**：`{{v1}}`
- **任务状态**：`{{READY / PAUSED / READY_TO_RESUME / COMPLETED}}`
- **基线 commit**：`{{完整 commit SHA}}`
- **Coder 运行环境**：`{{macOS / Linux / Windows / WSL}}`
- **仓库根目录**：`{{Coder 环境可访问的原生绝对路径}}`
- **本任务文档**：`{{Coder 环境可访问的原生绝对路径}}`
- **PRD**：`{{Coder 环境可访问的原生绝对路径}}`
- **技术方案**：`{{Coder 环境可访问的原生绝对路径}}`
- **执行计划**：`{{Coder 环境可访问的原生绝对路径}}`
- **分歧单**：`{{Coder 环境可访问的原生绝对路径}}`
- **交付报告**：`{{Coder 环境可访问的原生绝对路径}}`
- **未关闭分歧**：`{{无 / D-001, D-002}}`
- **已关闭分歧**：`{{无 / D-003}}`
```

Add one complete example each for `BOUNDED_AUTONOMY`, `EXACT_CHANGE`, and `INVESTIGATE_FIRST`. Use neutral example symbols such as `UserService`, but keep every file value as a literal native-absolute-path template variable rather than a concrete macOS path.

The report template must contain:

```markdown
## 1. 执行版本与基线
## 2. 实现单元结果
## 3. 自主决策摘要
## 4. 分歧与裁决执行情况
## 5. git diff
## 6. 自测结果
## 7. 自审 Checklist
## 8. 未完成或阻塞项
```

State that an open delivery-affecting dispute forces a blocked report and forbids `COMPLETED`.

- [ ] **Step 4: Add native-path and dispatch invariants**

In both files, include this exact rule:

```markdown
实际任务文档不得包含 `~`、环境变量、相对路径、`file://` URI、未解析模板变量或混合路径分隔符。Architect 必须按 Coder 环境生成并验证原生绝对路径；环境不明确或文件系统不共享时必须先询问用户。
```

Add the normative dispatch form:

```text
请读取并执行编码任务文档：<已验证的 Coder 环境原生绝对路径>
```

Clarify that the angle-bracket text is specification notation only and must never appear in actual dispatch output.

- [ ] **Step 5: Validate the v2 contract**

Run:

```bash
rg -n '协议版本.*v2.0.0|BOUNDED_AUTONOMY|EXACT_CHANGE|INVESTIGATE_FIRST|自主决策摘要|分歧单|原生绝对路径|未关闭分歧.*COMPLETED' \
  skills/coder-delegation/references/coder-prompt-spec.md \
  skills/coder-delegation/templates/coder-task.md
```

Expected: all v2 contract elements appear in the specification and template.

### Task 4: Make Scans and PRD Mapping Evidence-Aware

**Files:**
- Modify: `skills/coder-delegation/references/scan-delegation.md:1`
- Modify: `skills/coder-delegation/templates/codebase-scan-summary.md:1`
- Modify: `skills/coder-delegation/templates/prd-code-mapping.md:1`

- [ ] **Step 1: Add evidence status to delegated scanning rules**

Update `scan-delegation.md` to v2.0.0 and add `## 证据质量与事实/推断分离` with:

```markdown
| 状态 | 含义 | 可否直接用于裁决 |
|------|------|------------------|
| `DIRECT` | 已直接读取当前代码、配置、Schema 或依赖元数据 | 可作为证据，关键项仍由裁决者独立复核 |
| `REPRODUCED` | 已运行只读命令、构建、测试或检查并记录结果 | 可作为证据，保留命令与摘要 |
| `INFERRED` | 根据命名、调用邻接或历史资料推断 | 不可直接用于裁决 |
| `UNVERIFIED` | 未访问或访问失败 | 不可写成事实 |
```

State that a delegated summary is never sufficient by itself for `ARCHITECT_DECIDED`.

- [ ] **Step 2: Extend the scan summary template**

Add this evidence table shape wherever the template records decisive facts:

```markdown
| 原生绝对路径 | 符号/配置/Schema | 观察事实 | 证据状态 | 核实命令 | 备注 |
```

Add:

```markdown
## 6. 事实、推断与未核实项
## 7. 建议 Architect/Coder 重新核实的决定性证据
```

Require the appendix checklist to label every inference and give every decisive claim a direct or reproduced anchor.

- [ ] **Step 3: Split PRD mapping into desired and current truth**

Update `prd-code-mapping.md` to v2.0.0 with:

```markdown
# 目标真相↔当前代码事实映射：{{任务名}}

## 1. 目标真相来源
## 2. PRD/验收项 → 当前代码入口
## 3. 当前代码改动点 → 目标真相追溯
## 4. 目标与现状的已知差异
## 5. 跨仓调用链证据
## 6. 字段透传链路证据
## 7. 框架/组件 API 项目内证据
## 8. 未核实项与计划中的 `INVESTIGATE_FIRST` 单元
```

For every known difference, require a conflict classification and either a dispute ID or a written explanation that it is a normal implementation gap.

- [ ] **Step 4: Validate evidence labels**

Run:

```bash
rg -n 'DIRECT|REPRODUCED|INFERRED|UNVERIFIED|事实、推断|目标真相|当前代码事实|INVESTIGATE_FIRST' \
  skills/coder-delegation/references/scan-delegation.md \
  skills/coder-delegation/templates/codebase-scan-summary.md \
  skills/coder-delegation/templates/prd-code-mapping.md
```

Expected: all four evidence states and dual-truth terminology appear.

### Task 5: Upgrade the Main Orchestration Skill

**Files:**
- Modify: `skills/coder-delegation/SKILL.md:1`
- Reference: `skills/coder-delegation/references/evidence-dispute-arbitration.md:1`
- Reference: `skills/coder-delegation/templates/coder-task.md:1`
- Reference: `skills/coder-delegation/templates/coder-disputes.md:1`

- [ ] **Step 1: Record stale role descriptions before editing**

Run:

```bash
rg -n 'executes mechanically|Coder 不出方案|机械执行|按文档编码|不设计方案|提示词只给路径' \
  skills/coder-delegation/SKILL.md
```

Expected: the current mechanical-role and old dispatch wording match.

- [ ] **Step 2: Replace the frontmatter and role table**

Use a description that contains this complete behavior statement:

```text
Architect reuses a superpowers-approved design and plan, verifies repository facts, and creates an evidence-backed bounded-autonomy Coder task contract. Coder reasons from code, tests, configuration, schema, and dependency evidence; makes local in-envelope implementation decisions; records disputes; and pauses affected dependency branches. Architect independently verifies evidence, arbitrates technical facts, asks the user for product/contract/scope/risk decisions, and optionally bridges final review to multi-reviewer.
```

The role table must define Coder's positive responsibility as `证据驱动的受约束实现`. It must prohibit independent product decisions, public-contract changes, dependency expansion, persistence changes, and architecture redesign rather than prohibiting all implementation judgment.

- [ ] **Step 3: Replace the workflow with a seven-step v2 flow**

Use these exact stages:

```text
Step 1: 前置 superpowers 方案/计划检查
Step 2: 扫码、双真相映射与证据核实
Step 3: 生成 v2 任务合同、分歧单路径与自主边界
Step 4: 用户审阅任务合同
Step 5: 输出跨平台原生绝对路径派发提示词
Step 6: Coder 证据驱动执行；出现分歧时进入核实/用户/方案修订裁决环并更新任务版本
Step 7: Architect 最终审查与 AC 阈值判定
```

The Step 6 loop must use the exact state names from the arbitration reference. State that independent units may continue only with a written dependency-independence basis. State that a design-changing conflict returns to superpowers rather than being redesigned by Architect.

- [ ] **Step 4: Implement copy-only dispatch behavior**

Replace the old dispatch prose with:

```markdown
Architect 先确认 Coder 运行环境并验证任务文档原生绝对路径。最终派发输出只能包含一个独立代码块，不得在代码块前后添加说明，不得重复工作目录、分支、报告路径或硬约束。
```

Include these platform examples:

```text
请读取并执行编码任务文档：/Users/name/project/.claude/user-auth/coder/coder-task-user-auth.md
```

```text
请读取并执行编码任务文档：C:\Users\name\project\.claude\user-auth\coder\coder-task-user-auth.md
```

```text
请读取并执行编码任务文档：/mnt/c/Users/name/project/.claude/user-auth/coder/coder-task-user-auth.md
```

Require an interactive question when the target OS or shared-filesystem path is unknown.

- [ ] **Step 5: Update artifacts and reference tables**

Add these rows:

```markdown
| Step 3 | Coder v2 任务合同 | Coder 环境原生绝对路径 | Architect |
| Step 6 | Coder 分歧单 | 任务文档中指定的原生绝对路径 | Coder + Architect |
| Step 6 | Architect 追加式决策记录 | Coder 任务合同 §9 | Architect |
| Step 6 | Coder 交付报告 | 任务文档中指定的原生绝对路径 | Coder |
```

Add references to `references/evidence-dispute-arbitration.md` and `templates/coder-disputes.md`. Correct the existing broken reference name from `references/review-mrcc-bridge.md` to `references/review-ac-bridge.md`.

- [ ] **Step 6: Update Architect hard constraints and FAQ**

Require Architect to:

```text
- distinguish desired-state authority from current-state evidence;
- define an explicit autonomy envelope instead of complete code for every bounded unit;
- independently verify decisive Coder evidence;
- never treat delegated scan summaries as final proof;
- ask the user for product, contract, scope, dependency, persistence, and material-risk decisions;
- return design-changing conflicts to superpowers;
- version the task before resume;
- output only the copyable absolute-path dispatch code block.
```

Update the FAQ answer for Coder mistakes to describe evidence-backed disputes, arbitration, task revision, and resume rather than relying only on final review.

- [ ] **Step 7: Validate the main workflow**

Run:

```bash
rg -n '证据驱动|受约束|双真相|BOUNDED_AUTONOMY|分歧单|USER_INPUT_REQUIRED|PLAN_REVISION_REQUIRED|原生绝对路径|review-ac-bridge.md|coder-disputes.md' \
  skills/coder-delegation/SKILL.md
```

Expected: every v2 orchestration concept and both new files appear.

### Task 6: Make Final Review Dispute-Aware

**Files:**
- Modify: `skills/coder-delegation/references/review-ac-bridge.md:1`

- [ ] **Step 1: Add v2 review gates before threshold evaluation**

Update the version to v2.0.0. Before the existing file/line/HIGH thresholds, require Architect to verify:

```markdown
1. Coder 执行的是最新任务版本和预期基线。
2. 每个改动文件都在最终自主边界内。
3. 每个非平凡自主决策都有证据支持的决策摘要。
4. 没有决策越过强制升级边界。
5. 每个影响交付的分歧均为 `CLOSED`，且实现基于裁决指定的任务版本。
6. 改变实现行为的裁决有对应测试覆盖。
```

- [ ] **Step 2: Define unresolved disputes as BLOCKER**

Add this exact rule before the threshold table:

```markdown
任何影响交付的未关闭分歧、未纳入新任务版本的口头裁决、或 Coder 越过强制升级边界的实现，均计为 BLOCKER。该规则先于文件数、行数和 HIGH 阈值短路执行。
```

Keep the existing thresholds unchanged: BLOCKER ≥1 returns to Coder; otherwise files >3, lines >150, or HIGH ≥3 suggests AC.

- [ ] **Step 3: Update review output wording**

Passing output must include task revision and closed-dispute count. BLOCKER output must include a dispute ID or boundary violation. AC suggestion wording remains user-confirmed and must not describe unresolved disputes as mere threshold risk.

- [ ] **Step 4: Validate threshold preservation and new gates**

Run:

```bash
rg -n '最新任务版本|自主决策摘要|CLOSED|未关闭分歧.*BLOCKER|文件数.*3|行数.*150|HIGH.*3' \
  skills/coder-delegation/references/review-ac-bridge.md
```

Expected: the dispute gates and original numerical thresholds both appear.

### Task 7: Update Repository and User Documentation

**Files:**
- Modify: `AGENTS.md:1`
- Modify: `README.md:1`
- Modify: `docs/spec-architect-code.md:1`
- Modify: `docs/changelog.md:1`

- [ ] **Step 1: Update repository-wide agent instructions**

In `AGENTS.md`, replace the project summary and role table with this v2 principle:

```markdown
**核心思想**：强模型负责复用 superpowers 方案、核实证据、定义自主边界与裁决；Coder 根据目标真相和当前代码事实进行受约束实现。Coder 不盲从任务文档，也不擅自改变产品目标；出现分歧时落盘证据，暂停受影响依赖分支，由 Architect 或用户裁决。
```

Update the core flow to include dispute arbitration and task-version resume. Replace the Coder hard-constraint table with the 11 v2 constraints from Task 2. Add the new reference and template to the file tree. Preserve installation and AC threshold sections.

- [ ] **Step 2: Update README for users**

Change the tagline to:

```markdown
> **多角色 AI 编码协作** — 强模型复用方案并核实证据，Coder 基于真实代码受约束实现，分歧由 Architect 或用户裁决。
```

Add `## 证据驱动 Coder`, `## 分歧如何裁决`, and `## 跨平台派发`. Include macOS/Linux, Windows, and WSL one-line copyable prompt examples. Retain installation and multi-reviewer bridge guidance.

- [ ] **Step 3: Replace the consolidated protocol spec**

Rewrite `docs/spec-architect-code.md` as implemented v2.0.0 documentation with this structure and no `待定项` section:

```markdown
# coder-delegation (cdel) — 技能插件设计规格

> 版本：v2.0.0
> 日期：2026-07-26
> 状态：已实施

## 1. 概述
## 2. 角色与边界
## 3. 双真相与证据模型
## 4. 核心流程
## 5. Coder v2 任务合同
## 6. 分歧分类与依赖感知暂停
## 7. Architect/用户/方案修订裁决
## 8. 跨平台绝对路径派发
## 9. 自测、自审与交付
## 10. 最终审查与 AC 桥接
## 11. v1.x 兼容
## 12. 文件结构
```

Use the approved design as the normative source; do not preserve v1 statements that require complete code for every step or prohibit all Coder implementation judgment.

- [ ] **Step 4: Add the v2.0.0 changelog entry**

Insert this entry above v1.3.0:

```markdown
## v2.0.0 (2026-07-26)

- **Breaking — Coder 角色升级**：从机械执行改为证据驱动的受约束实现；允许边界内自主决策，禁止擅自改变产品、接口、依赖、持久化和风险承诺。
- **双真相模型**：用户确认的 PRD/验收决定目标状态；代码、测试、配置、Schema 和依赖版本决定当前状态；任务文档属于待验证指令。
- **三种执行模式**：新增 `BOUNDED_AUTONOMY`、`EXACT_CHANGE`、`INVESTIGATE_FIRST`。
- **分歧裁决闭环**：新增结构化分歧单、依赖感知暂停、Architect 独立核实、用户交互决策、superpowers 方案修订和任务版本恢复门。
- **跨平台派发**：任务、报告和分歧路径使用 Coder 环境原生绝对路径；最终提示词仅输出“执行 + 文件绝对路径”，可直接复制。
- **审查加强**：影响交付的未关闭分歧、口头裁决未落入新任务版本、越过强制升级边界均为 BLOCKER。
- **兼容性**：既有 v1.x 任务按 `EXACT_CHANGE` 继续执行；已派发任务不静默迁移。
```

- [ ] **Step 5: Validate active documentation consistency**

Run:

```bash
rg -n 'v2.0.0|证据驱动|双真相|依赖感知暂停|原生绝对路径|EXACT_CHANGE' \
  AGENTS.md README.md docs/spec-architect-code.md docs/changelog.md
```

Expected: every active document describes the same v2 model and compatibility mode.

### Task 8: Upgrade Plugin Metadata to v2.0.0

**Files:**
- Modify: `marketplace.json:1`
- Modify: `.claude-plugin/plugin.json:1`
- Modify: `.claude-plugin/marketplace.json:1`
- Modify: `.codex-plugin/plugin.json:1`
- Inspect and modify only if supported: `.agents/plugins/marketplace.json:1`

- [ ] **Step 1: Replace stale mechanical descriptions**

Use this description wherever the schema provides a plugin description field:

```text
Multi-role AI coding collaboration: Architect reuses a superpowers-approved design, verifies repository evidence, defines a bounded-autonomy task contract, and arbitrates disputes; Coder reasons from code, tests, configuration, schema, and dependency facts, implements inside explicit boundaries, self-tests, and reports decisions. Optional multi-reviewer bridge for final blind audit.
```

Use this `.codex-plugin/plugin.json` short description:

```text
Evidence-driven bounded Coder with Architect arbitration.
```

Use this long description:

```text
Architect reuses a technical design and implementation plan produced through superpowers, verifies current repository facts, and creates a self-contained v2 task contract with explicit autonomy and escalation boundaries. Coder makes evidence-backed local implementation decisions, records structured disputes, pauses affected dependency branches, runs tests, and self-reviews. Architect independently verifies evidence, asks the user for product/contract/scope/risk decisions, and reviews the final result with an optional multi-reviewer bridge.
```

- [ ] **Step 2: Set version-bearing manifests to 2.0.0**

Set every existing plugin-level `version` field to the JSON string `2.0.0` in:

```text
.claude-plugin/plugin.json
.claude-plugin/marketplace.json
.codex-plugin/plugin.json
```

If `marketplace.json` gains a plugin-level version, use `2.0.0` only after confirming that the same local marketplace shape supports it. In `.agents/plugins/marketplace.json`, preserve its existing schema shape; add only fields supported by the local format and otherwise leave the catalog entry versionless.

- [ ] **Step 3: Add evidence-driven discovery keywords**

Where keyword or tag arrays exist, add:

```json
"evidence-driven",
"bounded-autonomy",
"dispute-arbitration"
```

Do not remove existing `cdel`, `coder-delegation`, `multi-role`, `code-review`, or runtime discovery terms.

- [ ] **Step 4: Parse every manifest**

Run:

```bash
node - <<'NODE'
const fs = require('fs');
const files = [
  'marketplace.json',
  '.claude-plugin/plugin.json',
  '.claude-plugin/marketplace.json',
  '.codex-plugin/plugin.json',
  '.agents/plugins/marketplace.json',
];
for (const file of files) {
  JSON.parse(fs.readFileSync(file, 'utf8'));
}
console.log(`validated ${files.length} manifests`);
NODE
```

Expected:

```text
validated 5 manifests
```

- [ ] **Step 5: Verify version and positioning consistency**

Run:

```bash
rg -n '"version": "2\.0\.0"|evidence-driven|bounded-autonomy|dispute-arbitration' \
  marketplace.json .claude-plugin/plugin.json .claude-plugin/marketplace.json \
  .codex-plugin/plugin.json .agents/plugins/marketplace.json
```

Expected: all version-bearing manifests report v2.0.0 and schemas with description/tag support use the new positioning.

### Task 9: Run Full Protocol Consistency Validation

**Files:**
- Test: all files changed in Tasks 1–8

- [ ] **Step 1: Check formatting and JSON syntax**

Run:

```bash
git diff --check
```

Expected: exit status `0` with no output.

Run the five-manifest Node.js parser from Task 8 again.

Expected: `validated 5 manifests`.

- [ ] **Step 2: Check new references are linked from the main skill**

Run:

```bash
for path in \
  references/evidence-dispute-arbitration.md \
  templates/coder-disputes.md \
  references/coder-hard-constraints.md \
  references/coder-prompt-spec.md \
  references/scan-delegation.md \
  references/review-ac-bridge.md \
  templates/coder-task.md \
  templates/codebase-scan-summary.md \
  templates/prd-code-mapping.md; do
  rg -F "$path" skills/coder-delegation/SKILL.md >/dev/null || {
    echo "missing reference: $path"
    exit 1
  }
done
echo 'all skill references linked'
```

Expected:

```text
all skill references linked
```

- [ ] **Step 3: Check active files no longer define Coder as mechanical**

Run:

```bash
if rg -n 'executes mechanically|mechanical coding|机械执行者|Coder 不思考|只需照抄|strictly executes the task doc' \
  AGENTS.md README.md skills/coder-delegation/SKILL.md \
  skills/coder-delegation/references skills/coder-delegation/templates \
  marketplace.json .claude-plugin .codex-plugin .agents/plugins; then
  echo 'stale mechanical wording found'
  exit 1
fi
echo 'no stale mechanical role wording'
```

Expected:

```text
no stale mechanical role wording
```

Historical descriptions in `docs/changelog.md` and the approved design's v1.x problem statement may retain the term when explicitly describing prior behavior.

- [ ] **Step 4: Check protocol vocabulary is consistent**

Run:

```bash
for term in \
  BOUNDED_AUTONOMY EXACT_CHANGE INVESTIGATE_FIRST \
  FACT_MISMATCH PLAN_AMBIGUITY DESIGN_CONFLICT PRODUCT_AMBIGUITY \
  CONTRACT_OR_SCOPE SAFETY_RISK \
  EVIDENCE_READY ARCHITECT_VERIFYING USER_INPUT_REQUIRED \
  PLAN_REVISION_REQUIRED TASK_UPDATED RESUMED CLOSED; do
  rg -F "$term" \
    skills/coder-delegation/references/evidence-dispute-arbitration.md \
    skills/coder-delegation/references/coder-prompt-spec.md \
    skills/coder-delegation/templates/coder-disputes.md >/dev/null || {
      echo "missing protocol term: $term"
      exit 1
    }
done
echo 'protocol vocabulary consistent'
```

Expected:

```text
protocol vocabulary consistent
```

- [ ] **Step 5: Check cross-platform path requirements**

Run:

```bash
rg -n 'macOS|Linux|Windows|UNC|WSL|原生绝对路径|file://|环境变量|混合路径分隔符' \
  README.md skills/coder-delegation/SKILL.md \
  skills/coder-delegation/references/evidence-dispute-arbitration.md \
  skills/coder-delegation/references/coder-prompt-spec.md \
  skills/coder-delegation/templates/coder-task.md
```

Expected: platform detection, native absolute paths, and forbidden unresolved path forms are documented in every dispatch-defining layer.

- [ ] **Step 6: Check version and changelog consistency**

Run:

```bash
rg -n 'v2\.0\.0|2\.0\.0' \
  AGENTS.md README.md docs/spec-architect-code.md docs/changelog.md \
  skills/coder-delegation/SKILL.md skills/coder-delegation/references \
  skills/coder-delegation/templates .claude-plugin .codex-plugin marketplace.json
```

Expected: active protocol files identify v2.0.0 and version-bearing manifests use `2.0.0`.

- [ ] **Step 7: Review the final diff without modifying Git state**

Run:

```bash
git status --short
git diff --stat
git diff -- AGENTS.md README.md docs skills marketplace.json .claude-plugin .codex-plugin .agents/plugins
```

Expected: only files listed in this plan are modified or created; no generated files, dependency artifacts, or unrelated edits appear.

- [ ] **Step 8: Map every acceptance criterion to an artifact**

Use `docs/superpowers/specs/2026-07-26-evidence-driven-coder-design.md` §22 and confirm:

```text
AC 1–3   -> coder-hard-constraints + coder-prompt-spec + coder-task
AC 4–10  -> evidence-dispute-arbitration + coder-disputes + SKILL + review-ac-bridge
AC 11    -> coder-prompt-spec + docs/spec-architect-code + changelog
AC 12–13 -> SKILL + coder-prompt-spec + coder-task + README
AC 14    -> SKILL reference table
AC 15    -> manifests + changelog + active docs
```

Expected: no acceptance criterion lacks an implementation file and validation command.
