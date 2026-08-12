# coder-delegation (cdel) — Agent Instructions

> 本仓库是多角色 AI 编码协同技能插件。进入仓库后先读本文件和 `skills/coder-delegation/SKILL.md`。

## 项目目的

`coder-delegation` v2.0.0 编排“superpowers 方案 → 证据驱动实现 → Architect 审查 → 独立双盲验证”。

**核心思想**：强模型负责复用 superpowers 方案、核实证据、定义自主边界与裁决；Coder 根据目标真相和当前代码事实进行受约束实现。Coder 不盲从任务文档，也不擅自改变产品目标；出现分歧时落盘证据，暂停受影响依赖分支，由 Architect 或用户裁决。

## 角色与边界

| 角色 | 常见映射 | 职责 | 禁止 |
|------|----------|------|------|
| Architect | claude / mico lead | 需求、superpowers 方案/计划、证据核实、任务合同、审查、裁决 | 自创替代设计；自己做最终 QA |
| Coder | mico coder / codex / zcode | 在显式自主边界内实现、跑白名单、自审、报告 | 改产品语义、公共契约、依赖、持久化、架构或范围 |
| 独立 reviewer | codex / opencode / mico reviewer-* | 模式 A QA 或模式 B 方案评审，双盲核查 | 写源码、看对方报告、改共享环境、Git 写操作 |
| 用户 | — | 需求/方案/PR 审阅，产品、契约、范围和风险仲裁 | — |

codex 连续两次 PPIO 断流后由 zcode 接跑。mico 的 lead/coder/qa/reviewer-codex/reviewer-opencode/spec-reviewer/doc-reviewer 等按角色边界执行；工作虾只有获得明确授权时才改飞书文档。

## 核心流程

```text
superpowers:brainstorming + writing-plans
  → Architect 扫码、双真相映射、证据核实
  → v2 任务合同（BOUNDED_AUTONOMY / EXACT_CHANGE / INVESTIGATE_FIRST）
  → 用户确认
  → Coder 受约束实现、验证白名单、自审
  → 分歧落盘、依赖感知暂停、Architect/用户/superpowers 裁决
  → 新任务版本恢复并关闭分歧
  → Architect 独立审查
  → 模式 A codex + opencode 双盲 QA（按阈值/用户决定）
```

技术方案和执行计划由 superpowers 产出，cdel 不自行设计。已派发的 v1.x 任务按 `EXACT_CHANGE` 执行，不静默迁移。

## 触发与安装

- 显式命令：`/cdel <需求>`、`cdel` 或 `coder-delegation`。
- 自然语言实现需求可路由到主 skill；纯 review 使用独立 reviewer。
- Claude Code：`/plugin marketplace add <仓库 URL>` 后 `/plugin install coder-delegation`。
- Codex：`codex plugin marketplace add <仓库 URL>` 后 `codex plugin add coder-delegation@coder-delegation-marketplace`。

## Coder 硬约束

1. 编码前核实任务版本、目标真相、基线和当前代码事实。
2. 只在允许文件和自主边界内工作。
3. 边界内自主实现，不机械照抄过时行号或提示。
4. 不擅自改变产品、公共契约、持久化、依赖、架构或风险承诺。
5. 非平凡自主决策记录证据和摘要。
6. 分歧必须落盘，暂停受影响单元和下游依赖。
7. 只从含明确裁决的新任务版本恢复。
8. 不执行 Git 写操作。
9. 只跑任务文档验证白名单；基线债和鉴权失败不绕过。
10. 自测失败如实报告，不伪装通过。
11. 完成自审、脱敏并关闭启动进程后才交付。

Java 后端任务必须引用 `references/java-backend-standard.md`；只约束本任务新增/修改代码，不扩大治理存量。

## 双盲与审查

- 模式 A：编码后 QA，codex 与 opencode 独立跑接口/curl/CLI，Architect 对比报告。
- 模式 B：编码前方案评审，codex 与 opencode 独立核查方案和源码事实；B1 产品方案可单 reviewer。
- Reviewer 不读 `<repo>/.claude/<slug>/` 的 index/summary/PRD/方案决策史，不看对方报告。
- BLOCKER ≥1 直接暂停/打回；无 BLOCKER 且文件 >3、行 >150 或 HIGH ≥3 时建议 AC；否则通过。
- 影响交付的未关闭分歧、口头裁决未写入新任务版本、越界实现和 reviewer 硬约束违规都属于 BLOCKER。

## 验证白名单

Maven 编译优先使用 `-pl <模块> -am`；测试必须指定测试类，多个类用逗号分隔，禁止 `+` 导致 surefire 空跑。不得扩大到“相关测试”或未改动链路。只有 Architect 明确授权才能扩大验证。

## 归档结构

```text
<repo>/.claude/<slug>/<YYYY-MM-DD>/
<repo>/.claude/<slug>/coder/<YYYY-MM-DD>/
<repo>/.<reviewer>/<slug>/reviewer/<YYYY-MM-DD>/
```

主 agent 目录始终 `.claude/`，交叉验证产出只落主仓。派发 prompt 只给已验证的 Coder 环境原生绝对任务路径；完整报告落盘，聊天返回 ≤200 字可复制总结。

## 仓库文件

```text
skills/coder-delegation/
├── SKILL.md
├── references/
│   ├── evidence-dispute-arbitration.md
│   ├── workmode-protocol.md
│   ├── java-backend-standard.md
│   ├── coder-hard-constraints.md
│   ├── coder-prompt-spec.md
│   ├── scan-delegation.md
│   └── review-ac-bridge.md
└── templates/
    ├── coder-task.md
    ├── coder-disputes.md
    ├── codebase-scan-summary.md
    ├── prd-code-mapping.md
    ├── qa-regression-prompt.md
    └── design-review-prompt.md
```

## 贡献约束

- 新增/修改 reference 或 template 后同步 `SKILL.md` 引用和 `docs/changelog.md`。
- 本仓零第三方依赖，验证使用 Node.js 内置模块、ripgrep、Git 只读命令。
- 不执行 `git add`、`commit`、`push` 或其他 Git 写操作，除非用户明确授权。
