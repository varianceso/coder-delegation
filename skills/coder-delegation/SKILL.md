---
name: coder-delegation
description: >
  Multi-role coding workflow: strong-model Architect takes a technical plan + execution plan (produced via superpowers:brainstorming + writing-plans) and turns it into a self-contained Coder task doc, weak-model Coder executes mechanically, Architect reviews with optional multi-reviewer blind audit bridge. Alias: cdel. Use for cdel, coder-delegation, /cdel, "帮我实现", "改XX功能", "帮我加个XX接口", implementing features with model-tier orchestration, separating design judgment from mechanical coding. Do NOT use for typo fixes, README edits, pure renames, one-off scripts, or when the user already has a complete PR ready for simple review — use multi-reviewer for pure review needs. The Architect does NOT code and does NOT design solutions (that is superpowers' job); the Coder does NOT design.
---

# coder-delegation

短别名：**cdel**。

## 0. 一句话概览

> ⚠️ **本 skill = 强弱模型串行编码环**。
>
> **是什么**：Architect（强模型）**复用 superpowers 产出的技术方案 + 执行计划** → 转成 Coder 任务文档（机读化）→ 用户确认 → Coder（弱模型）按文档编码自测自审 → Architect 审查 + AC 桥接判定。
>
> **不是什么**：**不是**多 agent 盲审（那是 `multi-reviewer` 的事）、**不是**纯编码助手（Architect 不出代码，Coder 不出方案）、**不是**需求脑暴/方案设计工具（先用 `superpowers:brainstorming` + `writing-plans`）。**cdel 不另起炉灶做方案设计**——技术方案和执行计划都走 superpowers，cdel 只负责把它们机读化、委托弱模型编码、审查。
>
> **核心价值**：把"判断力密集型"工作（设计走 superpowers、审查）交给强模型，把"机械执行"工作（按文档编码、代码扫描摘要）交给弱模型，降低整体 token 成本的同时保证设计质量。

**2 角色串行**：

| 角色 | 模型 | 干什么 | 不干什么 |
|------|------|--------|----------|
| **Architect** | 强 | 复用 superpowers 方案/执行计划 → 扫码（必要时委托弱模型出摘要）→ 生成 Coder 任务文档 → 审查 Coder 产出 → AC 桥接判定 | 不写业务代码、不改源码、**不自己出技术方案/执行计划**（走 superpowers） |
| **Coder** | 弱 | 读编码任务文档 → 四方比对（PRD+技术方案+任务文档+代码）→ 按文档编码 → 自测（编译+单测）→ 自审（checklist）→ 交付报告 | 不设计方案、不改计划、不擅自按 PRD 改、不改非目标文件 |

> **Architect 怎么确定**：用户在哪个 agent session 里触发本 skill，那个就是 Architect。用户应在**强模型 session**（如 Claude Code + 强模型）中触发。
>
> **Coder 怎么确定**：Architect 生成 Coder 编码任务文档后，用户将文档交给弱模型 session（如 Codex + 弱模型）执行。Coder 不主动触发本 skill——它只读任务文档。
>
> **角色绑定**：Architect 和 Coder 不与具体工具绑定，用户在配置中自行映射。默认假设 Architect = Claude Code + 强模型，Coder = Codex + 弱模型。
>
> **角色错配防护**：若 Architect 识别到当前 session 是弱模型（判定提示：当前 session 的模型/工具不在用户配置的"强模型 Architect"映射内，或无法确认模型层级——例如模型名不含已知强模型标识、或运行时为 Codex/Zcode 等通常映射为 Coder 的工具），应**提醒用户角色错配**——"当前 session 疑似弱模型，Architect 需强模型。请在强模型 session 触发 cdel，或确认接受设计质量下降的风险"。不静默以弱模型充当 Architect。

---

## 1. 何时用 / 何时不用

| 用 | 不用 |
|---|---|
| 有明确实现需求（"帮我实现 XX 功能"） | typo / README 文字调整 |
| 改动涉及业务逻辑 / 接口契约 / 数据权限 | 纯重命名重构（编译通过即可） |
| 新增接口 / 新增 SQL / 新增模块 | 已有功能纯字段补充（无语义变化） |
| 0→1 新能力开发 | 实验性 / 一次性临时脚本 |
| 需要强模型出设计但用弱模型执行降低成本 | 已经有完整 PR 只需 review（用 `multi-reviewer`） |
| 改动复杂到需要结构化执行计划 | 用户明确说"我自己写，不用 cdel" |

**判断口径**：用户需求涉及"设计 + 编码"两步且希望分离角色 → 用 cdel。但**设计阶段（技术方案 + 执行计划）走 superpowers:brainstorming + writing-plans**，cdel 接在其后做"机读化 + 委托编码 + 审查"。纯设计审阅用 `ac`，纯脑暴用 `superpowers:brainstorming`。

## 1.1 触发关键词

**强触发**（命中即自动触发本 skill）：

| 中文 | 英文 | 行为 |
|------|------|------|
| /cdel | cdel | 直接进入 §2 流程 |
| 帮我实现 / 帮我写 / 帮我改 / 帮我加 | implement / write code for / add feature | 进入 §2 流程 |
| 改 XX 功能 / 加 XX 接口 / 实现 XX 需求 | — | 进入 §2 流程 |

**弱信号**（可能触发，优先用 superpowers）：

| 关键词 | 优先 skill | 理由 |
|--------|-----------|------|
| 头脑风暴 / brainstorm / 想想怎么做 | `superpowers:brainstorming` | 需求还没定，先澄清 |
| 出个方案 / 设计方案 / 写设计文档 | `superpowers:brainstorming` + `writing-plans` | 只要设计不要编码，不需要 cdel |
| 审一下 / review / QA | `multi-reviewer` | 纯审阅不需要 cdel 的编码环 |

---

## 2. 完整工作流

### 流程图

```
用户触发（/cdel 或关键词）
    ↓
Step 1: 需求确认 + 前置检查（含豁免退出口）
    │     └─ 无 superpowers 产出的方案/执行计划 → 引导先走 brainstorming+writing-plans
    ↓
Step 2: 扫码 + 核实闭环 + 生成 Coder 任务文档（机读化）
    ↓
Step 3: 用户审阅确认 Coder 任务文档（交互式，可调整）
    ↓
Step 4: 用户将文档交给 Coder（弱模型 session）
    ↓
Step 5: Coder 按文档编码 → 自测 → 自审 → 交付
    │     ├─ 交付 → 进 Step 6
    │     └─ 卡住（自测失败 / 计划偏差）→ 反馈 Architect → 调整任务文档 → 重派
    ↓
Step 6: Architect 快速审查 + 阈值判定
    ├─ 有 BLOCKER → ❌ 打回 Coder 修复 → 修复后重新审查
    ├─ 无 BLOCKER 但超阈值 → 建议 AC 盲审 → 用户决定
    │     ├─ 确认 → AC QA 回归
    │     └─ 拒绝 → 标"已知风险，用户知悉"
    └─ 全部阈值内 → ✅ 通过，告知用户
```

### Step 1: 需求确认 + 前置检查

Architect 用自然对话确认：

- 用户要做什么？（一句话目标）
- **有没有 superpowers 产出的技术方案 + 执行计划？**（cdel 的前置——cdel 不自己出方案）
- 涉及哪个仓库/模块？
- 有没有特殊约束（不改接口、保持兼容等）？

**前置检查（关键）**：

- **已有 superpowers 产出的方案 + 执行计划** → 进 Step 2，cdel 做机读化
- **没有方案/执行计划** → **引导用户先走 `superpowers:brainstorming`（澄清需求）+ `superpowers:writing-plans`（出技术方案 + 执行计划）**，cdel 不自己另起炉灶设计方案。用户完成后再回 cdel
- **需求本身就不明确** → 同样先 `superpowers:brainstorming`，cdel 不做需求脑暴

**不要进入多轮需求脑暴或方案设计**——那应该先走 `superpowers:brainstorming` + `writing-plans`。cdel 的需求确认是轻量的，聚焦在"前置方案是否就绪、能不能开始机读化"。

**豁免退出口**：若确认后发现需求属于 §1 "不用"场景（纯重命名、纯字段补充、typo、一次性脚本等），Architect 应建议用户走更轻量方式（直接改 / `superpowers:test-driven-development`）并**退出 cdel**，不强行走完整流程。

### Step 2: 核实闭环 + 生成 Coder 任务文档

> ⚠️ **AC 通过 ≠ coder-task 正确**。multi-reviewer 评审的是技术方案的**设计层面**（接口契约、架构合理性、安全策略）；cdel Step 2 的机读化是**代码层面**（具体调用路径、字段值、组件 API、行号引用）。AC 通过后，cdel Step 2 仍必须对每个改动点扫码核实，不能因"方案已通过 AC 评审"而跳过核实。**凭印象生成的 coder-task 会导致 Coder 全量回退**。

Architect 基于 superpowers 产出的技术方案 + 执行计划，经过**核实闭环**后，生成一份**独立、自包含、可执行**的 Coder 编码任务文档（机读化）。

#### 2a. 扫码（按需委托弱模型）

- 小改动（改 1-2 个方法、单文件）→ Architect 直接读相关代码，不委托
- **大改动（跨模块 / 需理解调用链 / 数据流）→ 委托弱模型扫描出摘要**，Architect 只消费摘要做机读化判断。详见 `references/scan-delegation.md`
  - 扫描委托两条路径：① 当前环境有 workflow 子 agent 能力（如 Claude Code workflow）→ 启动弱模型子 agent 并行扫描出摘要；② 无 workflow 能力 → 询问用户是否用其他弱模型 session 扫描出代码概况摘要文档
  - 摘要面向设计决策：模块职责 / 关键类签名 / 调用链 / 数据流 / 改动影响面（模板见 `templates/codebase-scan-summary.md`）
  - 摘要落盘可复用：`<repo>/.claude/<slug>/codebase-scan-<name>.md`，后续会话/同类改动复用，已有摘要时只扫增量

#### 2b. 核实闭环（强制，机读化前置）

生成 coder-task 之前，Architect 必须完成以下 5 项核实——不能凭印象、不能仅靠扫描摘要推断、不能因"方案已通过 AC 评审"跳过：

| # | 核实项 | 方法 | 触发条件 |
|---|--------|------|----------|
| 1 | **PRD 需求项 ↔ 代码入口 双向映射** | 对 PRD 每个功能点（F-xxx），扫码确认对应代码入口/DTO/Service/DAO，落盘 `prd-code-mapping.md`（模板见 `templates/prd-code-mapping.md`）。反向核查：coder-task 列出的每个改动文件必须能追溯到 PRD 功能点 | 所有任务（强制） |
| 2 | **跨仓调用链实名核实** | 涉及前端→后端、CLI→后端跨仓调用时，扫码确认实际调用路径（Controller→Service→Handler 完整链路），不能凭印象或命名惯例推断。**常见错误**：以为 CLI 走通用 CRUD 接口（DeptUpsertReq），实际走 agent 专属路径（PreviewAdjustItemReq → AgentPlanPreviewService → CreateDeptHandler） | 跨仓任务 |
| 3 | **字段透传全链路完整性** | 字段透传类任务，必须追踪入口→校验→转换→执行→落库每一环，确认每环都已列入步骤。**常见遗漏**：补了入口层（Req/Param/Controller），漏了执行层（stepPayload + Handler setter），导致字段在中间环节丢失 | 字段透传/新增字段 |
| 4 | **框架/组件 API 项目实际用法确认** | coder-task 涉及框架组件（如 antd TextArea、Element UI）时，必须在项目内找到现有用法锚点（file:line），引用其 API 形态。**禁止凭通用知识写 API**——项目可能用 `autoSize` 而非 `autosize`，用 `onChange(e) => e.target.value` 而非 `onChange(val)` | 涉及框架组件 |
| 5 | **所有代码引用扫码核实** | coder-task 中每个行号/方法名/类名/字段名引用，必须通过 `git show origin/<base>:<file>` 或直接读文件核实存在。旁标核实状态（✅ 已核实 / ⚠️ 未核实待 Coder 验证），无 ✅ 标记的引用不得写入 coder-task | 所有引用 |

> **核实闭环 ≠ 全量代码审查**。核实聚焦在"机读化的每个事实点是否真实存在"——调用路径是否走通、行号是否正确、API 用法是否匹配项目实际、PRD 需求是否全员映射。架构合理性、接口设计正确性由前置的 superpowers + AC 评审保证，不在本步核实范围。

**核实产物**：
- `prd-code-mapping.md` — 落盘 `<repo>/.claude/<slug>/prd-code-mapping-<name>.md`
- 核实过程中发现的分歧/缺口，记入 coder-task §1.1"已知分歧"，同时反馈用户确认

#### 2c. 生成 Coder 任务文档

核实闭环完成后，把技术方案 + 执行计划 + 核实结果 → 机读化为 Coder 任务文档。

**格式规范**：见 `references/coder-prompt-spec.md`
**文档模板**：见 `templates/coder-task.md`

关键原则：
- Coder 读完后不需要追问，直接开干
- 编译命令、单测命令可直接复制粘贴运行
- 硬约束明确，Coder 不会误操作
- 所有代码引用旁标核实状态（✅/⚠️）
- PRD↔技术方案分歧显式标注在 §1.1，Coder 不会按 PRD 擅自修改

**文档落盘位置**：`<repo>/.claude/<slug>/coder/coder-task-doc

### Step 3: 用户审阅确认 Coder 任务文档

Architect 将 Coder 任务文档展示给用户。用户审阅：

- 任务文档是否正确落实了 superpowers 的技术方案 + 执行计划？
- 执行步骤是否合理、定位是否精确？
- 有没有遗漏的改动点？

用户可以要求调整任务文档。Architect 修改后重新展示，直到用户确认。

### Step 4: 用户派发

Architect 告知用户：

```
Coder 编码任务文档已生成：<路径>

请将此文件交给 Coder（Codex + 弱模型）。Coder 会按文档编码并写报告到指定文件。
Coder 完成后直接读报告文件 <报告路径>，我来审查。
```

**用户操作**：在弱模型 session 中加载该文档作为 prompt，让 Coder 执行。

**派发提示词语气**（约束全在任务文档内，提示词只给路径+执行+报告落盘）：

```
请按 <coder-task.md 路径> 执行编码任务。工作目录 <repo路径>，分支 <分支名>。完成后将交付报告写入 <报告路径>，写完后直接退出，无需回复。
```

> 所有硬约束、执行步骤、自测命令、交付要求已在 coder-task 文档内自包含。派发提示词**不重复**这些内容，Coder 读文档时自然获得完整指令。

### Step 5: Coder 执行（不在本 session 中）

Coder 在独立 session 中：

1. 加载编码任务文档
2. **编码前**：按硬约束第 0 条做 PRD+技术方案+任务文档+代码四方比对，发现未标注分歧→停止并反馈；技术方案自身问题→记录在报告供 Architect 修复
3. 按 §4 执行步骤逐条编码
4. 按 §5 跑自测（编译 + 单测）
5. 按 §6 过自审 checklist
6. 按 §7 将交付报告（git diff + 自测结果 + checklist 状态）**写入任务文档指定的报告文件路径**，写完即退出

**报告落盘规则**：任务文档 §7 写死报告文件路径（如 `<repo>/.claude/<slug>/coder/coder-report-<name>.md`）。Coder 将完整报告写入该文件后直接退出 session，**不回复"已写入 XX 文件"**，**不额外总结**——Architect 会直接读该文件进行审查。这减少用户在中转环节的操作成本。

**卡住时的反馈回路**（不静默硬扛）：

- **四方比对未标注分歧**：PRD/技术方案/任务文档之间存在未在 §1.1 标注的分歧 → **停止编码**，反馈 Architect 具体分歧点
- **技术方案自身问题**：设计缺陷、步骤遗漏等不影响当前编码的问题 → **不停止**，记录在报告 §技术方案问题反馈
- **自测失败**：编译报错 / 单测 FAIL → 不要瞎改绕过，停止并反馈 Architect（报错信息 + 已尝试），由 Architect 判断是调整任务文档还是 Coder 重试
- **计划偏差**：任务文档里的行号 / 方法名 / 锚点对不上实际代码 → **停止编码**，反馈 Architect"任务文档与实际代码不符，{具体不符点}"，不强行匹配
- **缺依赖**：需要任务文档 §3.2 未声明的依赖 → 标记自审不通过并反馈 Architect

Coder 硬约束见 `references/coder-hard-constraints.md`。

### Step 6: Architect 审查

Architect **直接读取 Coder 写入的报告文件**（不是等用户转发），做快速审查：

1. **独立 git diff 核实**：对目标仓库执行 `git diff`，不只看 Coder 报告中的 diff（Coder 可能漏报或截断）
2. **对任务文档执行步骤**：diff 中每个 step 是否完成
3. **对文件清单**：独立核实是否只改了任务文档列出的文件（不做"信 Coder 自报"的假设）
4. **对自测结果**：是否真实全部 PASS
5. **对自审 checklist**：是否有明显矛盾
6. **PRD 覆盖度核查**：对 PRD 每个功能点（F-xxx），确认 coder-task 有对应实现步骤，防止任务文档漏 PRD 需求
7. **代码质量**：按严重度分级（BLOCKER/HIGH/MEDIUM/LOW/NOTE）

**阈值判定**（详见 `references/review-mrcc-bridge.md`，按顺序短路）：

| 条件 | 结果 |
|------|------|
| 有 BLOCKER（≥1） | ❌ 直接打回 Coder 修复，修复后重新审查（不进 AC） |
| 无 BLOCKER，但 文件数>3 或 行数>150 或 HIGH≥3 | ⚠️ 建议 AC 盲审，用户决定 |
| 无 BLOCKER 且 全部阈值内 | ✅ 通过 |

**通过时的话术**（如实报 HIGH 数量，不宣称"未发现 HIGH"）：
```
✅ 审查通过。改动 {N} 文件 {M} 行，HIGH {x} 个（阈值 <3）、BLOCKER 0 个。代码已就绪。
```

**打回时的话术**（有 BLOCKER）：
```
❌ 审查发现 BLOCKER {Y} 个，直接打回 Coder 修复（不进入 AC 盲审）：
- {问题}：{位置} — {原因}，期望修复：{方向}
请将反馈交回 Coder，修复后重新提交，我重新审查。
```

**超阈值时的话术**（无 BLOCKER）：
```
⚠️ 审查发现超阈值项（无 BLOCKER）：文件数 {N}（阈值≤3）、HIGH {X} 个（阈值<3）。
建议触发 multi-reviewer (ac) 盲审。确认？
- 确认 → 我检测 AC 安装情况并生成 QA prompt
- 拒绝 → 标"已知超阈值风险，用户知悉"（不标"通过"）
```

用户确认桥接后，Architect **先检测 multi-reviewer 是否安装**（见 §3），已安装则生成 QA 回归提示词（符合 multi-reviewer 的 QA prompt 格式），用户贴给 codex/opencode 执行。

---

## 3. 审查→AC 桥接

当 Coder 改动**无 BLOCKER 但规模/HIGH 超阈值**时，Architect 建议用户桥接到 `multi-reviewer` 的 QA 回归模式（模式 A）。有 BLOCKER 时不走本节，直接打回 Coder（见 Step 6）。

**桥接条件**：用户确认 + multi-reviewer 已安装。

**AC 安装检测**（按顺序尝试，命中即停）：① 查当前会话可用 skill 列表是否含 `multi-reviewer` 或 `ac`；② Glob 已知 skills 目录是否存在其 SKILL.md（如 `~/.claude/skills/multi-reviewer/SKILL.md`）；③ Claude Code 查 plugin cache / `/plugin list`，Codex 查 `~/.codex/plugins/`。
- 已安装 → 生成 QA prompt 并交付用户
- 未安装 → 告知"未检测到 multi-reviewer，无法桥接。请先安装，或自行人工复核"，不强行桥接、不静默跳过

**Architect 操作**：
1. 基于 Coder 的 diff + 任务文档执行步骤，生成 QA 回归提示词
2. 提示词符合 multi-reviewer 的 `qa-regression-prompt.md` 模板格式
3. 告知用户"把这份 prompt 贴给 codex/opencode 跑 QA 回归"

详见 `references/review-mrcc-bridge.md`。

## 4. 与 superpowers / multi-reviewer 的协作

```
superpowers:brainstorming    ← 需求澄清
        ↓
superpowers:writing-plans    ← 出技术方案 + 执行计划（cdel 不替代这步，复用其产出）
        ↓
coder-delegation (cdel)         ← 本 skill：机读化 + 委托编码 + 审查（必要时委托弱模型扫描出摘要）
        ↓
multi-reviewer (ac)        ← 超阈值时：盲审 QA 回归
        ↓
superpowers:verification-before-completion  ← 最终确认
```

| 场景 | 用哪个 |
|------|--------|
| 需求不明确，要讨论 | `superpowers:brainstorming` |
| 出技术方案 / 执行计划 | `superpowers:brainstorming` + `writing-plans` |
| 方案+执行计划就绪，要强弱分离编码 | **cdel** |
| 代码改完了，只要 review | `multi-reviewer` (ac) |
| cdel 审查超阈值 | cdel 建议 → `multi-reviewer` QA 模式 |

## 5. 产出物清单

| 阶段 | 产出 | 落盘位置 | 作者 |
|------|------|----------|------|
| 前置（superpowers） | 技术方案 + 执行计划 | `<repo>/.claude/<slug>/tech-design-<name>.md` 等 | Architect（经 superpowers） |
| Step 2a | 代码扫描摘要（大改动时，可选） | `<repo>/.claude/<slug>/codebase-scan-<name>.md` | 弱模型（委托） |
| Step 2b | PRD↔代码双向映射（强制） | `<repo>/.claude/<slug>/prd-code-mapping-<name>.md` | Architect |
| Step 2c | Coder 编码任务文档 | `<repo>/.claude/<slug>/coder/coder-task-doc | Architect |
| Step 5 | Coder 交付报告（git diff + 自测 + checklist） | `<repo>/.claude/<slug>/coder/coder-report-<name>.md`（任务文档 §7 指定） | Coder |
| Step 6 | 审查结论 | 当前 session 中输出 | Architect |
| Step 6 | QA 回归 prompt（可选） | `<repo>/.claude/<slug>/qa-regression-prompt.md` | Architect |

> 技术方案 + 执行计划由 superpowers:writing-plans 产出，非 cdel 独有产出，cdel 复用之。

## 6. 参考文件

| 文件 | 内容 |
|------|------|
| `references/coder-hard-constraints.md` | Coder 硬约束（含第 0 条四方比对 + PRD 只读） |
| `references/coder-prompt-spec.md` | Coder 编码任务文档规范（含执行步骤动作语义、已知分歧、核实状态） |
| `references/scan-delegation.md` | 代码扫描委托规范（触发条件 / 两路径 / 摘要结构 / 复用 / 跨仓调用链核实） |
| `references/review-mrcc-bridge.md` | 审查→AC 桥接规范（含 PRD 覆盖度核查） |
| `templates/coder-task.md` | Coder 编码任务文档模板 |
| `templates/codebase-scan-summary.md` | 代码扫描摘要模板 |
| `templates/prd-code-mapping.md` | PRD 功能点↔代码入口双向映射模板 |

## 7. 硬约束（Architect 侧）

- Architect **不写业务代码**——只做机读化和审查
- Architect **不自己出技术方案/执行计划**——走 superpowers:brainstorming + writing-plans，cdel 复用其产出
- Architect **不自动触发 AC**——始终经用户确认
- Architect 产出的 Coder 任务文档**必须自包含**——Coder 不应需要追问
- Architect **机读化时所有代码引用 + PRD 需求映射 + 跨仓调用链 + 组件 API 必须扫码核实**——凭印象生成的 coder-task 会导致 Coder 全量回退
- Architect 审查时**独立 git diff 核实**——不只看 Coder 自报的 diff
- Architect 审查时**如实报告超阈值项**——不因"改动看起来不严重"而跳过 AC 建议
- Architect 发现 **BLOCKER 必须直接打回 Coder**——不进 AC 盲审、不因用户拒绝而标"通过"
- Architect 通过话术**如实报 HIGH 数量**——不宣称"未发现 HIGH"（阈值允许 ≤2 HIGH 通过）
- Architect **大改动扫描委托弱模型**——降强模型 token 成本；小改动直接读，不为降本反增协调成本

## 8. 常见问题

### cdel 和 ac 怎么区分？

| | cdel | ac |
|---|---|---|
| 做什么 | 设计 + 委托编码 + 审查 | 多 agent 盲审 |
| 谁写代码 | Coder（弱模型） | 没人写代码（只审） |
| 什么时候用 | "帮我实现 XX" | "帮我审一下这个方案/代码" |
| 角色数 | 2 (Architect + Coder) | N+1 (主 agent + N reviewer) |

### Coder 写错了怎么办？

Coder 自测+自审应该拦住大部分问题。如果还有遗漏，Architect 在 Step 6 审查时发现，超阈值则建议 AC 盲审补验。

### 可以跳过 Coder 直接用 Architect 编码吗？

cdel 的核心价值就是角色分离。如果不需要分离，直接用 `superpowers:test-driven-development` 更快。

### 执行计划要多细？

执行计划由 superpowers:writing-plans 产出，cdel 不重新定义其粒度。cdel 关心的是：把执行计划机读化成 Coder 任务文档时，执行步骤要精确到文件/定位/动作/代码块，让弱模型不需要追问。机读化粒度见 `references/coder-prompt-spec.md` §4。

### 扫描代码量太大怎么办？

大改动时 Architect **委托弱模型扫描出摘要**，自己只消费摘要做机读化判断，降强模型 token 成本。两条路径：① 有 workflow 子 agent 能力（如 Claude Code workflow）→ 启动弱模型子 agent 并行扫描；② 无 workflow 能力 → 询问用户是否用其他弱模型 session 扫描出代码概况摘要文档。摘要面向设计决策（模块职责/调用链/数据流/影响面），落盘可复用。详见 `references/scan-delegation.md`。

小改动（1-2 个方法）Architect 直接读，不委托——委托本身有协调成本，改动小时不值得。
