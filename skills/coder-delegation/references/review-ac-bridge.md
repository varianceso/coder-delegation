# 审查→AC 桥接规范

- 版本：v1.1 (2026-06-26)
- 适用：Architect 角色审查 Coder 产出后，判定是**打回**、**通过**、还是**桥接**到 multi-reviewer 盲审
- 依赖：multi-reviewer 技能插件（独立安装，可选）

## 桥接决策流程

```
Architect 审查 Coder 产出
    ↓
按严重度分级计数（BLOCKER / HIGH / MEDIUM / LOW / NOTE）
    ↓
有 BLOCKER？──是──→ ❌ 直接打回 Coder 修复（不进 AC）
    │                   └─ Coder 修复后重新提交，重新走本流程
    否
    ↓
规模/HIGH 超阈值？──是──→ ⚠️ 建议触发 AC 盲审，原因：{具体超阈值项}
    │                       ├─ 用户确认 → 检测 AC 是否安装 → 调 multi-reviewer QA 模式（模式 A）
    │                       └─ 用户拒绝 → 标"已知风险，用户知悉"（不标"通过"）
    否
    ↓
✅ 全部在阈值内 → 通过，告知用户"编码完成，审查通过"
```

**关键区分**：

- **BLOCKER** = 已知致命问题（功能不可用 / 数据错误 / 安全漏洞）。Architect 审查时**已经发现**，正确动作是**直接打回 Coder 修复**，而不是去 AC 盲审。AC 的定位是"独立验证"，不是"修已知 bug"——让 reviewer 盲审一个已知含 BLOCKER 的产物既浪费又偏离 AC 价值。
- **规模/HIGH 超阈值** = 改动大或高危问题多，值得独立视角复核。这才是 AC 盲审的适用场景。

## 阈值定义

阈值分两类，触发**不同**动作：

| 维度 | 阈值 | 触发动作 | 统计方式 |
|------|------|----------|----------|
| **BLOCKER** | ≥1 个 | ❌ 直接打回 Coder 修复 | Architect 审查发现的致命问题数 |
| 文件数 | >3 个 | ⚠️ 建议 AC 盲审 | `git diff --stat` 统计修改文件数 |
| 行数 | >150 行 | ⚠️ 建议 AC 盲审 | `git diff --shortstat` 增删行数（insertions + deletions） |
| HIGH 问题 | ≥3 个 | ⚠️ 建议 AC 盲审 | Architect 审查发现的高严重度问题数 |

**判定规则**（按顺序短路）：

1. **先看 BLOCKER**：有任何一个 → 直接打回，不进入后续判定
2. **无 BLOCKER 时看规模/HIGH**：文件数、行数、HIGH 任一超阈值 → 建议 AC 盲审
3. **全部在阈值内** → 通过

> 注：纯删除大量废代码也可能让"行数"超阈值触发 AC 建议——这是有意的保守策略，用户可拒绝并标"已知风险"。

## AC 安装检测

建议 AC 盲审前，Architect 须先确认 multi-reviewer 是否可用：

- **检测方式**（按顺序尝试，命中即停）：
  1. 查当前会话可用 skill 列表是否含 `multi-reviewer` 或 `ac`
  2. Glob 已知 skills 目录是否存在其 SKILL.md，如 `~/.claude/skills/multi-reviewer/SKILL.md`、`~/.codex/.../multi-reviewer/SKILL.md`
  3. Claude Code session 可查 plugin cache / `/plugin list`；Codex session 可查 `~/.codex/plugins/`
- **已安装** → 生成 QA 回归 prompt，告知用户贴给 codex/opencode 执行
- **未安装** → 告知用户"未检测到 multi-reviewer，无法桥接盲审。请先安装，或自行人工复核超阈值风险"，**不强行桥接、不静默跳过**

## 严重度分级

复用 multi-reviewer 的严重度体系：

| 级别 | 定义 | 示例 |
|------|------|------|
| **BLOCKER** | 功能不可用 / 数据错误 / 安全漏洞 | 权限校验漏了、SQL 字段写错、NPE 未处理导致服务崩溃 |
| **HIGH** | 边界/异常下行为不正确、偏离计划 | 空值未处理、超时未设默认值、与执行计划描述的行为不一致 |
| **MEDIUM** | 小偏差 / 健壮性不足 | 日志级别不当、缺少必要的 null check（已有上层兜底） |
| **LOW** | 风格 / 命名 / 注释问题 | 变量名不符合项目惯例、注释拼写错误 |
| **NOTE** | 观察 / 未验证项 / 建议 | "建议补一个边界测试"、"这个改动可能影响 X 模块，建议留意" |

## Architect 审查要点

审查不是完整 CR，而是**快速验证**集中在：

| 检查项 | 方法 |
|--------|------|
| **独立 git diff** | **自己对目标仓库跑 `git diff`**，不只看 Coder 报告中的 diff（Coder 可能漏报/截断） |
| 是否偏离执行计划 | diff 与执行计划 §4 逐节对照 |
| 是否漏做步骤 | 执行计划有几条 step，diff 是否都覆盖 |
| 是否有越权改动 | diff 文件清单 vs 任务文档 §3.1 文件清单 |
| 自测是否真实通过 | Coder 提交的自测结果是否含 FAIL/ERROR |
| 自审 checklist 是否诚实 | 逐条看一眼 checklist 状态是否有明显矛盾 |
| **PRD 覆盖度** | 对 PRD 每个功能点（F-xxx），确认 coder-task 有对应实现步骤，防止任务文档漏 PRD 需求 |

发现 BLOCKER 时，打回反馈中须明确：问题位置、严重度、为什么是 BLOCKER、期望 Coder 如何修。

### 独立 git diff 核查规则

- Architect 审查时 **先自己对仓库跑 `git diff`**，拿到真实 diff，然后与 Coder 报告中的 diff 比对
- 比对重点：文件数一致、改动行数偏差 <5%、没有 Coder 漏报的文件
- 差异 >5% → 标记为审查异常，要求 Coder 解释差异原因
- Coder 报告文件不存在/无法读取 → 要求用户确认 Coder 是否已执行完毕

## 提示用户的话术

### 通过时（无 BLOCKER 且全部阈值内）

```
✅ 审查通过。改动 {N} 文件 {M} 行，HIGH {x} 个（阈值 <3）、BLOCKER 0 个。代码已就绪。
```

> 通过允许 0~2 个 HIGH（<3 即通过）。话术如实报 HIGH 数量，**不宣称"未发现 HIGH"**——否则当存在 1~2 个 HIGH 时与实际矛盾。

### 打回时（有 BLOCKER）

```
❌ 审查发现 BLOCKER {Y} 个，直接打回 Coder 修复（不进入 AC 盲审）：
- {问题1}：{位置} — {为什么是 BLOCKER}，期望修复：{方向}
- {问题2}：...

请将以上反馈交回 Coder 修复，修复后重新提交 diff + 自测结果，我重新审查。
```

### 超阈值时（无 BLOCKER 但规模/HIGH 超阈值）

```
⚠️ 审查发现以下超阈值项（无 BLOCKER）：
- 文件数：{N}（阈值 ≤3）
- 行数：{M}（阈值 ≤150）
- HIGH 问题：{X} 个（阈值 <3）

建议触发 multi-reviewer (ac) 盲审进行独立回归。是否桥接？
- 确认 → 我检测 AC 安装情况并生成 QA prompt
- 拒绝 → 我将标注"已知超阈值风险，用户知悉"（注意：不会标"审查通过"）
```

## 与 multi-reviewer 的集成方式

用户确认桥接且 AC 已安装后，Architect 生成 multi-reviewer QA 回归的输入：

1. **QA prompt 内容**：基于 Coder 的 diff + 执行计划，生成一份标准 QA 回归提示词
2. **提示词格式**：符合 multi-reviewer 的 `qa-regression-prompt.md` 模板
3. **派发方式**：告知用户"把这份 prompt 贴给 codex/opencode 跑 QA 回归"

Architect **不自动调用 multi-reviewer**——生成 QA prompt 后交给用户，由用户在实际的 reviewer agent session 中使用。

## 版本记录

- v1.1 (2026-06-26)：审查要点增加"独立 git diff"和"PRD 覆盖度"两项；增加独立 git diff 核查规则
- v1.0 (2026-06-24)：首版
