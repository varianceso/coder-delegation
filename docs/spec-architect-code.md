# coder-delegation (cdel) — 技能插件设计规格

> 版本：v1.1
> 日期：2026-06-24
> 状态：已实施

## 1. 概述

`coder-delegation`（别名 `cdel`）是一个 Claude Code / Codex / Zcode 等 CLI/桌面端编程工具的技能插件。它编排"强模型设计 + 弱模型编码 + 强模型审查"的串行工作流。

**核心思想**：用强模型（Claude + GLM 5.2 等）做需要判断力的工作（方案设计、计划制定、最终审查），用弱模型（Codex + 阉割版模型）做机械执行（按计划编码）。

## 2. 角色抽象

角色与具体工具解耦，用户在配置中自行映射：

| 角色 | 职责 | 模型要求 | 默认工具映射 |
|------|------|----------|-------------|
| **Architect** | 复用 superpowers 方案/执行计划 → 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 审查 Coder 产出 → 判定是否需 AC 盲审 | 强模型 | Claude Code |
| **Coder** | 按 Coder 任务文档编码 → 自测 → 自审 → 交付 diff+结果 | 弱模型 | Codex / Zcode |

> 技术方案 + 执行计划由 superpowers:brainstorming + writing-plans 产出，cdel 不自己出方案。cdel 独有的是 Coder 任务文档（机读化）+ 委托编码 + 审查。

## 3. 核心流程

```
用户提需求
    ↓
前置：superpowers:brainstorming 澄清需求 + writing-plans 出技术方案 + 执行计划
    ↓
Architect（强模型）复用方案/执行计划 + 扫码（大改动委托弱模型出摘要）→ 生成 Coder 编码任务文档（机读化）
    ↓
用户审阅确认 Coder 任务文档（交互式，可调整）
    ↓
Architect 派发 Coder 任务文档
    ↓
Coder（弱模型）加载文档 → 按文档编码 → 自测(编译+单测) → 自审(checklist)
    │   └─ 卡住（自测失败 / 计划偏差）→ 反馈 Architect → 调整任务文档 → 重派
    ↓
Architect 快速审查 + 阈值判定（按顺序短路）
    ├─ 有 BLOCKER → ❌ 直接打回 Coder 修复 → 修复后重新审查（不进 AC）
    ├─ 无 BLOCKER 但超阈值（文件>3 或 行>150 或 HIGH≥3）
    │   └─ 提示用户"建议触发 AC 盲审"，用户确认后调 multi-reviewer 模式 A
    └─ 全部阈值内 → ✅ 直接通过，告知用户
```

## 4. 技能触发

| 触发方式 | 示例 | 说明 |
|----------|------|------|
| 显式命令 | `/cdel 给 UserService 加权限校验` | 精确触发 |
| 短别名 | `cdel` | 等价于 `/cdel` |
| 对话关键词 | "帮我实现 XX 功能"、"改 XX"、"加 XX 接口" | 匹配实现/改/加功能等关键词 |

Architect 识别到触发后，先判断用户是否有 superpowers 产出的技术方案 + 执行计划——没有则引导先走 `superpowers:brainstorming`（澄清需求）+ `writing-plans`（出方案+执行计划），cdel 不自己出方案；有则进入扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 确认流程。

## 5. 执行计划

**执行计划由 superpowers:writing-plans 产出，cdel 不重新定义其格式。** cdel 的职责是把执行计划机读化为 Coder 编码任务文档（见 §6），执行步骤的动作语义（insert_after / insert_before / replace / delete / new_file）在 `coder-prompt-spec.md` §4 自包含定义。

大改动时，Architect 委托弱模型扫描出代码摘要（面向设计决策：模块职责/关键类签名/调用链/数据流/影响面），降强模型 token 成本。详见 `references/scan-delegation.md`。

## 6. Coder 编码提示词文档

Architect 为每个编码任务生成一份**独立、自包含、可执行**的 `.md` 文档。Coder 加载该文档后不需追问、不需补全设计，直接开干。

文档质量要求：Coder agent 读完后即具备完整上下文，可独立完成编码+自测+自审全流程。

### 文档结构

```markdown
# 编码任务：{任务名}

> 生成时间：{YYYY-MM-DD HH:mm}
> 技术方案来源：{superpowers:writing-plans 产出的 tech-design 文档引用}
> 执行计划来源：{superpowers:writing-plans 产出的 execution-plan 文档引用}
> 执行者：Coder（只读本文件，不要偏离计划）

## 1. 任务概述
{一句话 + 一段，自然语言交代要做什么、为什么}

## 2. 硬约束
固定 8 条（从 coder-hard-constraints.md 复制，等价摘要）：
- 只改 §3.1 文件清单列出的文件，不碰其他文件
- 不动接口契约、不换异常类型、不改 DTO/VO 字段语义
- 不调 §3.2 未声明的外部服务/依赖；计划偏差（行号/方法名/锚点对不上）停止编码并反馈
- 不动 git 写操作（不 add / commit / push / reset --hard / checkout --）
- 编码完成后必须跑自测（编译 0 error + 单测全 PASS）；自测失败不瞎改绕过，停止反馈
- 编码完成后必须过自审 checklist，全部通过才交付
- 输出脱敏（不含真实 token/姓名/邮箱/工号/手机号/连接串）
- 启动的进程/服务验完立即 kill

## 3. 上下文
### 3.1 受影响文件清单
| 文件 | 动作 | 备注 |
|------|------|------|

### 3.2 关键依赖
- 方法签名、类路径、异常类型

## 4. 执行步骤
### Step N
- **文件**：完整路径
- **定位**：方法名 / 行号
- **动作**：insert_after / replace / insert_before / delete / new_file
- **代码**：完整代码块（含缩进）

## 5. 自测要求
### 编译
```bash
{具体编译命令}
```

### 跑单测
```bash
{具体单测命令}
```

### 验收标准
- 编译 0 error
- 所有单测 PASS（含新增 + 已有）

## 6. 自审 Checklist
编码完成后逐条确认：
- [ ] 编译通过，单测全绿
- [ ] 未修改本文档未列出的文件
- [ ] 未更改已有方法的签名
- [ ] 未引入新的外部依赖调用
- [ ] 输出中不含真实 token/姓名/邮箱
- [ ] 启动的进程已全部关闭
- [ ] 所有执行步骤已按顺序完成

## 7. 输出要求
提交以下三项：
1. **git diff**（所有改动文件的统一 diff）
2. **自测结果**（编译输出 + 单测通过数）
3. **自审 checklist**（§6 的完成状态）
```

## 7. 审查→AC 桥接

### 阈值定义

阈值分两类，触发不同动作：

| 维度 | 阈值 | 触发动作 | 说明 |
|------|------|----------|------|
| **BLOCKER** | ≥1 个 | ❌ 直接打回 Coder 修复 | 已知致命问题，不进 AC |
| 文件数 | >3 个 | ⚠️ 建议 AC 盲审 | git diff --stat 统计 |
| 行数 | >150 行 | ⚠️ 建议 AC 盲审 | git diff --shortstat 统计 |
| HIGH 问题 | ≥3 个 | ⚠️ 建议 AC 盲审 | 按 multi-reviewer 严重度分级 |

### 判定逻辑

```
Architect 审查 Coder 产出
    ↓
分级计数（BLOCKER / HIGH）+ 统计（文件数、行数）
    ↓
有 BLOCKER → ❌ 直接打回 Coder 修复 → 修复后重新审查（不进 AC）
    ↓（无 BLOCKER）
全部在阈值内 → ✅ 通过，告知用户
任一超阈值 → 提示用户"建议触发 AC 盲审" → 用户确认后调 multi-reviewer QA 模式
```

BLOCKER 不进入 AC 盲审——AC 是独立验证，不是修已知 bug。不自动触发 AC，始终经过用户确认。

### 严重度分级（复用 multi-reviewer）

| 级别 | 定义 |
|------|------|
| **BLOCKER** | 功能不可用 / 数据错误 / 安全漏洞 |
| **HIGH** | 边界/异常下行为不正确、偏离计划 |
| **MEDIUM** | 小偏差 / 健壮性不足 |
| **LOW** | 风格 / 命名 / 注释问题 |
| **NOTE** | 观察 / 未验证项 / 建议 |

## 8. 用户交互节奏

```
1. 用户         "/cdel 给 UserService 加权限校验"
2. （前置）superpowers:brainstorming + writing-plans 出技术方案 + 执行计划
3. Architect    复用方案/执行计划 + 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 展示给用户
4. 用户         审阅确认 / 提出调整
5. Architect    派发 Coder 任务文档
6. Coder        加载文档 → 编码 → 自测 → 自审 → 返回 diff+结果
7. Architect    快速审查 → 统计+分级 → 出结论
   ├─ 有 BLOCKER → ❌ 打回 Coder 修复 → 修复后重新审查（不进 AC）
   ├─ 通过     → 告知用户 "编码完成，审查通过"
   └─ 超阈值（无 BLOCKER）→ 告知用户 "建议 AC 盲审，确认？"
        ├─ 确认 → 调 multi-reviewer QA 模式
        └─ 拒绝 → 标"已知风险，用户知悉"（不标"通过"）
```

## 9. 与 multi-reviewer 的关系

两个插件**独立部署**，但有协作点：

| | multi-reviewer | coder-delegation |
|---|---|---|
| 核心任务 | 多 agent 独立盲审 | 强弱模型串行编码 |
| 角色 | N 个 reviewer | 1 Architect + 1 Coder |
| Coder 写代码 | ❌ 禁止 | ✅ 核心任务 |
| 触发时机 | 方案就绪 / 编码完成 | 用户有实现需求 |
| 严重度 | BLOCKER/HIGH/MEDIUM/LOW/NOTE | 复用同一套 |
| 协作点 | — | 审查超阈值时建议用户调 AC |

### Coder 硬约束 vs AC Reviewer 硬约束

| 约束 | AC Reviewer | Coder |
|------|------------|-------|
| 只读不写源码 | ✅ 强制 | ❌（编码是核心任务） |
| 归档目录外不落盘 | ✅ 强制 | ✅ 强制 |
| 不动 git 写操作 | ✅ 强制 | ✅ 强制 |
| 数据脱敏 | ✅ 强制 | ✅ 强制 |
| 不动共享环境 | ✅ 强制 | ✅ 强制 |
| 启进程验完 kill | ✅ 强制 | ✅ 强制 |
| 鉴权失败不硬跑 | ✅ 强制 | ✅ 强制 |
| 不改方案/接口契约 | 不适用 | ✅ 强制 |
| 不改非目标文件 | 不适用 | ✅ 强制 |
| 自测+自审后交付 | 不适用 | ✅ 强制 |

## <REDACTED> 文件结构

```
coder-delegation/                        # 独立 Git 仓库
├── README.md                          # 插件概述与使用说明
├── LICENSE                            # MIT
├── .gitignore
├── .claude-plugin/
│   ├── plugin.json                    # Claude Code 插件清单
│   └── marketplace.json               # Claude Code 市场清单
├── .codex-plugin/
│   └── plugin.json                    # Codex 插件清单
├── .agents/plugins/
│   └── marketplace.json               # 跨运行时市场清单
├── docs/
│   ├── spec-coder-delegation.md         # 本文件
│   └── changelog.md
└── skills/
    ├── cdel/
    │   └── SKILL.md                   # 别名，指向 coder-delegation
    └── coder-delegation/
        ├── SKILL.md                   # 主入口：流程编排 + 决策树
        ├── references/
        │   ├── coder-hard-constraints.md      # Coder 硬约束
        │   ├── coder-prompt-spec.md           # Coder 任务文档规范（含执行步骤动作语义）
        │   ├── scan-delegation.md             # 代码扫描委托规范（降强模型成本）
        │   └── review-mrcc-bridge.md            # 审查→AC 桥接阈值与流程
        └── templates/
            ├── coder-task.md                  # Coder 编码任务文档模板
            └── codebase-scan-summary.md       # 代码扫描摘要模板
```

## 11. 待定项

以下内容在实施阶段细化（v1.0 实施后状态）：

- [x] SKILL.md 中 Architect agent 的决策树具体话术 — 已在 SKILL.md §2 流程图 + 话术落地
- [x] Coder 提示词文档中各节的精确措辞（确保弱模型不误解） — 已在 `coder-prompt-spec.md` 各节要点落地
- [x] 自审 checklist 是否需要按语言/框架定制 — 首版采用通用 7 条，暂不按语言定制
- [ ] 审查→AC 桥接时，AC QA prompt 的自动填充字段 — 依赖 multi-reviewer 模板，首版手工填充
- [x] 是否需要 `scripts/init.mjs` 做项目本地 bootstrap — 首版不需要（零第三方依赖）
- [x] plugin.json 的 keywords / category — 已在 `.claude-plugin/plugin.json` 落地
