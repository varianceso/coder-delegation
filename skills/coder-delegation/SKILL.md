---
name: coder-delegation
description: >
  Evidence-driven multi-role coding workflow: Architect reuses a superpowers-approved design and plan,
  verifies repository facts, defines bounded-autonomy Coder units, arbitrates disputes, and optionally
  bridges independent codex/opencode/mico review. Coder reasons from evidence, self-tests on a strict
  validation allowlist, self-reviews, and reports. Use for /cdel, coder-delegation, and implementation
  requests. Do not use for typo-only, README-only, pure rename, one-off script, or review-only work.
---

# coder-delegation / cdel

## 0. 一句话概览

> 强模型复用 superpowers 方案并核实证据，Coder 基于目标真相和当前代码事实受约束实现，分歧由 Architect 或用户裁决，独立 reviewer 做双盲方案评审或 QA 回归。

cdel 不替代 `superpowers:brainstorming` / `superpowers:writing-plans`，不把固定行号补丁当作唯一执行方式，也不让 Architect 自己充当最终 QA。

## 1. 角色与运行时映射

| 角色 | 默认映射 | 职责 | 边界 |
|------|----------|------|------|
| Architect | claude / 强模型 | 需求澄清、复用方案与计划、核实证据、定义自主边界、编码协同、审查和裁决 | 不自创替代设计；最终 QA 交给独立 reviewer |
| Coder | mico coder / codex / zcode | 读目标真相和代码证据，在任务边界内实现、自测、自审、交付 | 不改变产品、公共契约、依赖、持久化或风险承诺 |
| 独立 reviewer | codex、opencode、mico reviewer-* | 模式 A QA 回归或模式 B 方案评审，双盲并行 | 只读、不修源码、不看对方产出 |
| 接跑替补 | zcode | codex 连续断流后的同任务接跑 | 继承原任务版本、边界和报告路径 |
| 用户 | — | 需求、方案、PR、产品/契约/范围/风险分歧仲裁 | 只在材料决策需要时介入 |

详见 `references/workmode-protocol.md`。当前 session 疑似弱模型而需 Architect 时，提醒角色错配并让用户确认风险。

## 2. 何时使用

适用于明确的业务实现、接口、数据权限、跨模块或需要方案与编码分离的任务。纯 typo、README 文字、纯重命名、一次性脚本或已有完整 PR 的纯审阅应退出 cdel，使用更轻量的直接修改、TDD 或独立 reviewer。

## 3. 前置方案检查

1. 确认一句话目标、仓库、模块、特殊兼容约束和 Coder 环境。
2. 检查是否已有经 `superpowers:brainstorming` + `superpowers:writing-plans` 批准的 PRD/技术方案/执行计划。
3. 没有则引导先完成 superpowers；cdel 不自行设计方案。
4. 已有方案才进入扫码和任务合同生成。

## 4. 七步 v2 工作流

```text
Step 1: 前置 superpowers 方案/计划检查
Step 2: 扫码、双真相映射与证据核实
Step 3: 生成 v2 任务合同、分歧单路径与自主边界
Step 4: 用户审阅任务合同
Step 5: 输出跨平台原生绝对路径派发提示词
Step 6: Coder 证据驱动执行；分歧进入核实/用户/方案修订裁决环并更新任务版本
Step 7: Architect 最终审查与 AC 阈值判定
```

### Step 2：扫码与核实

小改动直接读代码；大改动按 `references/scan-delegation.md` 委托摘要。Architect 必须将目标真相与当前代码事实分开记录，双向映射 PRD 功能点、调用链、字段流、组件 API、配置和依赖。摘要不能单独作为裁决证据。

### Step 3：任务合同

使用 `templates/coder-task.md`，每个实现单元声明 `BOUNDED_AUTONOMY`、`EXACT_CHANGE` 或 `INVESTIGATE_FIRST`，允许文件必须是 Coder 环境原生绝对路径，Java 后端任务显式引用 `references/java-backend-standard.md` 并声明例外。分歧单使用 `templates/coder-disputes.md`。

### Step 5：精简派发

Architect 先验证目标 OS、共享文件系统和任务文件路径。派发输出只能是一个独立代码块，不在其前后添加说明，不内联源码、方案正文、工作目录、分支、报告路径或硬约束：

```text
请读取并执行编码任务文档：/absolute/path/to/.claude/<slug>/coder/coder-task-<name>.md
```

实际 prompt 不得含 `<...>` 模板变量、`~`、环境变量、相对路径、`file://` URI 或混合分隔符。结尾要求标准化可复制总结（≤200 字）；完整报告写入任务指定路径。

### Step 6：执行与分歧

Coder 依据当前代码证据在边界内自主实现，不能机械照抄过时锚点。只跑任务文档验证白名单：编译优先使用 `-pl <受影响模块> -am`，测试必须列具体测试类，禁止扩大到未改动链路。基线债和鉴权失败立即记录并停止扩大验证。发现分歧写入分歧单，暂停受影响分支；独立单元只有在写明依赖独立性后才可继续。

状态必须沿 `DISCOVERED → EVIDENCE_READY → ARCHITECT_VERIFYING → ARCHITECT_DECIDED / USER_INPUT_REQUIRED / PLAN_REVISION_REQUIRED → TASK_UPDATED → RESUMED → CLOSED` 推进。口头裁决不能恢复任务；设计冲突返回 superpowers；产品/契约/范围/风险决定询问用户。

### Step 7：Architect 审查

Architect 独立读取 Git diff、任务版本、基线、决策摘要、分歧单、测试结果和所有改动文件，不只相信 Coder 报告。先按 `references/review-ac-bridge.md` 检查未关闭分歧和越界，再做 BLOCKER/HIGH/MEDIUM/LOW/NOTE 分级。

## 5. 模式 A/B 独立交叉验证

### 模式 A：编码后 QA 回归

本地集成测试和冒烟完成后生成 `<repo>/.claude/<slug>/<YYYY-MM-DD>/qa-regression-prompt.md`，分别交给 codex 与 opencode。两者只读验证接口、curl、CLI 或验收路径，独立写 `qa-report`，不看对方过程或报告；Architect 对比后裁决，不自己作最终 QA。

### 模式 B：编码前方案评审

PRD、技术方案、测试或上线方案定稿后生成 `<repo>/.claude/<slug>/<YYYY-MM-DD>/tech-review-prompt.md`，分别交给 codex 与 opencode，只读文档并核查代码事实，写独立 `evaluation`。产品方案 B1 可只派一名 reviewer。

## 6. 产出与归档

```text
<repo>/.claude/<slug>/<YYYY-MM-DD>/                       主 agent 产物
<repo>/.claude/<slug>/coder/<YYYY-MM-DD>/                 Coder 任务、报告和分歧
<repo>/.<reviewer>/<slug>/reviewer/<YYYY-MM-DD>/          reviewer 私有过程
```

主 agent 目录始终 `.claude/`；交叉验证报告只落主仓。slug 用 kebab-case，跨会话决策史为 `<slug>/summary.md`。

## 7. 审查→AC 桥接阈值

按短路顺序：

1. 任一影响交付的未关闭分歧、未写入新任务版本的口头裁决、Coder 越过强制升级边界或 reviewer 违反硬约束 = **BLOCKER**，直接打回/暂停，不进 AC。
2. 无 BLOCKER 且文件数 >3、行数 >150 或 HIGH ≥3 = 建议用户确认后桥接 multi-reviewer 模式 A。
3. 无 BLOCKER 且全部阈值内 = 通过；如实报告任务版本、关闭分歧数、文件/行数和 HIGH 数量。

模式 A/B 产生的报告仍须独立、脱敏、只读和双盲；codex PPIO 单请求最多重试 2 次，连续两次断流切 zcode 接跑。

## 8. 参考文件

| 文件 | 内容 |
|------|------|
| `references/evidence-dispute-arbitration.md` | 双真相、证据、分歧、暂停、裁决和恢复 |
| `references/workmode-protocol.md` | 角色映射、模式 A/B、双盲、白名单、断流、归档、派发 |
| `references/java-backend-standard.md` | Java 命名、参数、排版、分层、数据、异常、日志、注释、MySQL |
| `references/coder-hard-constraints.md` | Coder 11 条硬约束和自审 |
| `references/coder-prompt-spec.md` | v2 任务合同规范 |
| `references/scan-delegation.md` | 证据感知扫描摘要 |
| `references/review-ac-bridge.md` | 独立审查、BLOCKER 和 AC 阈值 |
| `templates/coder-task.md` | Coder 任务合同/报告模板 |
| `templates/coder-disputes.md` | 分歧单模板 |
| `templates/codebase-scan-summary.md` | 扫描摘要模板 |
| `templates/prd-code-mapping.md` | 目标/现状映射模板 |
| `templates/qa-regression-prompt.md` | 模式 A 独立 QA 回归模板 |
| `templates/design-review-prompt.md` | 模式 B 独立方案评审模板 |

## 9. Architect 硬约束

- 复用 superpowers 方案，不自创替代设计。
- 区分目标真相和当前代码证据，独立核实决定性主张。
- 定义显式自主边界和升级条件，不为每个单元写死完整源码。
- 不把委托摘要当最终证据，不把自己当最终 QA。
- 产品、契约、范围、依赖、持久化和重大风险决定必须询问用户。
- 设计冲突回到 superpowers；裁决后先更新任务版本再恢复。
- 最终派发只输出一个可复制的原生绝对路径代码块。

## 10. 常见问题

### Coder 发现任务和代码对不上怎么办？

停止受影响单元，写分歧单和证据锚点；独立单元只有在写明无依赖后可继续。Architect 独立核实：事实问题可裁决，产品/契约/范围/风险问题问用户，设计冲突回 superpowers。裁决写入新任务版本后才恢复。

### Java 规则是否适用于所有任务？

只有 Java 后端任务选择 `java-backend-standard.md`；前端、脚本和插件文档任务不套用。适用任务只约束新增/修改代码，不借机治理无关存量违规。

### cdel 和独立 reviewer 怎么配合？

cdel 负责编码协同和 Architect 审查；模式 A/B 的 codex/opencode/mico reviewer 只读双盲验证，报告回到主仓，由 Architect 对比裁决。

## 版本记录

- v2.0.0 (2026-08-12)：接入证据驱动任务合同、当前工作模式、Java 后端规则和独立双盲验证。
