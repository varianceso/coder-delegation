# coder-delegation (cdel) — Agent Instructions

> **这是本仓库的 AGENTS.md。** 任何 AI agent（Claude Code / Codex / Zcode / Cursor / Cline 等）
> 在加载本目录时，应首先阅读本文件以理解项目目的、自身角色与行为边界。

## 项目是什么

`coder-delegation`（别名 `cdel`）是一个 **多角色 AI 编码协作技能插件**，
编排"强模型设计 + 弱模型编码 + 强模型审查"的串行工作流。

**核心思想**：用强模型做需要判断力的工作（设计走 superpowers、审查），用弱模型做机械执行（编码、代码扫描摘要）。**技术方案 + 执行计划由 superpowers:brainstorming + writing-plans 产出，cdel 不自己出方案**，只负责机读化、委托编码、审查。

## 角色定义

本技能定义两个抽象角色，**不与具体工具绑定**（用户在配置中映射）：

| 角色 | 职责 | 模型要求 | 行为边界 |
|------|------|----------|----------|
| **Architect** | 复用 superpowers 方案/执行计划 → 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 最终审查 + AC 桥接判定 | 强 | 读代码、写 Coder 任务文档、审查 Coder 产出、判定是否需 AC 盲审；**不自己出技术方案/执行计划**（走 superpowers） |
| **Coder** | 按 Coder 任务文档编码 → 自测（编译+单测） → 自审（checklist） → 交付 diff+结果 | 弱 | 严格按任务文档编码、不偏离方案、不改非目标文件、自测自审通过后才交付 |

## 核心流程

```
用户提需求 "/cdel <需求描述>"
    ↓
前置：superpowers:brainstorming + writing-plans 出技术方案 + 执行计划
    ↓
Architect 复用方案/执行计划 + 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档（机读化）
    ↓
用户审阅确认 Coder 任务文档（交互式，可调整）
    ↓
Architect 派发 Coder 任务文档
    ↓
Coder 加载文档 → 按文档编码 → 自测 → 自审 → 交付（diff + 自测结果 + 自审 checklist）
    ↓
Architect 快速审查 + 阈值判定
    ├─ 有 BLOCKER → ❌ 直接打回 Coder 修复 → 修复后重新审查（不进 AC）
    ├─ 全部在阈值内（≤3文件 且 ≤150行 且 <3 HIGH 且 0 BLOCKER）→ ✅ 通过
    └─ 无 BLOCKER 但任一超阈值 → 提示用户"建议 AC 盲审"，用户确认后调 multi-reviewer QA 模式
```

## 安装（用户侧）

### Claude Code

```bash
/plugin marketplace add <本仓库 URL>
/plugin install coder-delegation
```

### Codex

```bash
codex plugin marketplace add <本仓库 URL>
codex plugin add coder-delegation@coder-delegation-marketplace
```

### 手动 clone

```bash
git clone <本仓库 URL> ~/.claude/skills/coder-delegation-src
ln -s ~/.claude/skills/coder-delegation-src/skills/coder-delegation ~/.claude/skills/coder-delegation
ln -s ~/.claude/skills/coder-delegation-src/skills/cdel ~/.claude/skills/cdel
```

## 触发方式

| 触发 | 示例 |
|------|------|
| 显式命令 | `/cdel 给 UserService 加权限校验` |
| 短别名 | `cdel` / `/cdel` |
| 对话关键词 | "帮我实现 XX"、"改 XX 功能"、"加 XX 接口" |

Architect 识别触发后，先判断用户是否有 superpowers 产出的技术方案 + 执行计划——没有则引导先走 `superpowers:brainstorming` + `writing-plans`，cdel 不自己出方案；有则进入扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 确认流程。

## 产出物

| 阶段 | 产出 | 作者 |
|------|------|------|
| 前置（superpowers） | 技术方案 + 执行计划（.md） | Architect（经 superpowers） |
| 扫描阶段（大改动可选） | 代码扫描摘要（.md） | 弱模型（委托） |
| 派发阶段 | Coder 编码任务文档（.md，自包含可执行） | Architect |
| 编码阶段 | 代码 diff + 自测结果 + 自审 checklist | Coder |
| 审查阶段 | 审查结论（通过 / 打回 Coder / 建议 AC 盲审） | Architect |
| 审查阶段 | QA 回归 prompt（可选，超阈值时） | Architect |

## Coder 硬约束

Coder 在编码时必须遵守以下约束，任何违反 = 不可交付：

| # | 约束 |
|---|------|
| 1 | 只改编码任务文档中列出的文件，不碰其他文件 |
| 2 | 不修改接口契约、异常类型、已有方法签名 |
| 3 | 不调编码任务文档未声明的外部服务/依赖；计划偏差（行号/方法名/锚点对不上）停止编码并反馈 Architect，不强行匹配 |
| 4 | 不动 git 写操作（add / commit / push） |
| 5 | 编码完成后必须跑自测（编译 + 单测），不通过不交付；自测失败不瞎改绕过，停止并反馈 Architect |
| 6 | 编码完成后必须过自审 checklist |
| 7 | 输出中脱敏（不含真实 token/姓名/邮箱） |
| 8 | 启动的进程验完立即 kill |

## 审查→AC 桥接阈值

| 维度 | 阈值 | 触发动作 | 说明 |
|------|------|----------|------|
| **BLOCKER** | ≥1 个 | ❌ 直接打回 Coder 修复 | 功能不可用/数据错误/安全漏洞；不进 AC |
| 文件数 | >3 个 | ⚠️ 建议 AC 盲审 | git diff --stat |
| 行数 | >150 行 | ⚠️ 建议 AC 盲审 | git diff --shortstat |
| HIGH 问题 | ≥3 个 | ⚠️ 建议 AC 盲审 | 边界/异常行为不正确、偏离计划 |

判定（按顺序短路）：有 BLOCKER → 直接打回 Coder 修复，修复后重新审查（不进 AC）；无 BLOCKER 且全部阈值内 → ✅ 通过；无 BLOCKER 但任一超阈值 → 提示用户，用户确认后调 `multi-reviewer` QA 模式。

## 与 multi-reviewer 的关系

| | multi-reviewer | coder-delegation |
|---|---|---|
| 核心任务 | 多 agent 独立盲审 | 强弱模型串行编码 |
| 角色 | N 个 reviewer | 1 Architect + 1 Coder |
| Coder 是否写代码 | ❌ 禁止 | ✅ 核心任务 |
| 触发时机 | 方案就绪 / 编码完成 | 用户有实现需求 |
| 协作点 | — | 审查超阈值时建议用户调 AC |

## 文件结构

```
coder-delegation/
├── AGENTS.md                       ← 本文件
├── README.md
├── LICENSE
├── .gitignore
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── .codex-plugin/
│   └── plugin.json
├── .agents/plugins/
│   └── marketplace.json
├── docs/
│   ├── spec-coder-delegation.md      ← 设计规格
│   └── changelog.md
└── skills/
    ├── cdel/
    │   └── SKILL.md                ← 短别名
    └── coder-delegation/
        ├── SKILL.md                ← 主入口
        ├── references/
        │   ├── coder-hard-constraints.md
        │   ├── coder-prompt-spec.md
        │   ├── scan-delegation.md
        │   └── review-mrcc-bridge.md
        └── templates/
            ├── coder-task.md
            └── codebase-scan-summary.md
```

## 参与贡献

- 新增/修改 `references/` 或 `templates/` 后需同步更新 `SKILL.md` 的引用
- 协议变更需更新 `docs/changelog.md`
- 本仓零第三方依赖，脚本使用 Node.js 内置模块

## License

MIT
