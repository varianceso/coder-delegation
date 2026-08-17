# coder-delegation (cdel) — 技能插件设计规格

> 版本：v2.0.0
> 日期：2026-08-12
> 状态：已实施

## 1. 概述

cdel 编排 superpowers 方案、证据驱动 Coder、Architect 裁决和独立双盲 reviewer。插件以 Markdown 文件为跨运行时协议，不依赖实时 agent 消息或第三方运行时代码。

## 2. 角色与边界

Architect 负责复用批准方案、核实仓库事实、定义自主边界、审查和裁决；不得在 cdel 内独立重设计。Coder 在显式边界内自主实现并记录证据，不得改变产品、公共契约、依赖、持久化、架构或风险。codex/opencode/mico reviewer 只读核查，不改源码；zcode 是 codex 连续断流后的替补。用户裁决产品、契约、范围和重大风险。

当前工作模式允许 claude 作为指挥和实施者，但最终 QA 必须交给独立 reviewer，避免自己验证自己。

## 3. 双真相与证据模型

目标真相由用户决定、验收、PRD 和批准方案确定；当前真相由代码、测试、配置、Schema、迁移和依赖确定。证据分 `DIRECT`、`REPRODUCED`、`INFERRED`、`UNVERIFIED`。行号只是辅助，稳定符号和可复现命令优先。委托扫描摘要不构成最终裁决证明。

## 4. 核心流程

```text
Step 1 superpowers 方案/计划检查
Step 2 扫码、双真相映射和证据核实
Step 3 生成任务合同、自主边界、分歧/报告路径
Step 4 用户审阅
Step 5 原生绝对路径派发
Step 6 Coder 实现、白名单自测、分歧裁决、任务版本恢复
Step 7 Architect 独立审查和可选双盲 QA
```

## 5. Coder v2 任务合同

任务合同包含文档状态、目标真相、当前事实、自主边界、实现单元、验证白名单、分歧协议、自审、报告和 Architect 决策记录。实现单元使用 `BOUNDED_AUTONOMY`、`EXACT_CHANGE` 或 `INVESTIGATE_FIRST`。Java 后端任务引用 `java-backend-standard.md`，只治理本任务新增/修改代码。

## 6. 分歧分类与依赖感知暂停

分歧分类为 `LOCAL_CHOICE`、`FACT_MISMATCH`、`PLAN_AMBIGUITY`、`DESIGN_CONFLICT`、`PRODUCT_AMBIGUITY`、`CONTRACT_OR_SCOPE`、`SAFETY_RISK`。Coder 暂停受影响单元及依赖；独立单元只有写明无依赖证据时才可继续。

## 7. Architect/用户/方案修订裁决

Architect 独立核实事实和边界内选择；产品、契约、范围、依赖、持久化和风险由用户决定；设计变化返回 superpowers。状态从 `DISCOVERED` 经证据和裁决进入 `TASK_UPDATED`、`RESUMED`、`CLOSED`。口头裁决无效，任务文档必须升级版本。

## 8. 跨平台绝对路径派发

任务、报告和分歧使用 Coder 环境原生绝对路径，支持 macOS/Linux、Windows、WSL。禁止 `~`、环境变量、相对路径、`file://`、未解析模板变量和混合分隔符。最终派发仅输出“请读取并执行编码任务文档：<已验证原生绝对路径>”的独立代码块。

## 9. 自测、自审与交付

Coder 只运行验证白名单：构建限制在受影响模块，测试指定具体类，禁止扩大到未改动链路。基线债和鉴权失败停止扩大并如实报告。完整交付落盘，聊天只返回 ≤200 字可复制总结。交付需包含版本/基线、Unit 结果、决策摘要、分歧状态、diff、测试、自审和阻塞项。

## 10. 最终审查与 AC 桥接

未关闭交付分歧、口头裁决未落入新版本、越界实现和 reviewer 硬约束违反均为 BLOCKER。无 BLOCKER 时，文件 >3、行 >150 或 HIGH ≥3 建议用户确认后进行模式 A 双盲 QA。模式 B 用于编码前方案评审；B1 产品方案可单 reviewer。codex PPIO 最多重试 2 次，连续两次断流切 zcode。

## 11. v1.x 兼容

已派发 v1.x 补丁脚本按 `EXACT_CHANGE` 继续执行，不静默迁移。新任务默认 v2 合同。

## 12. 文件结构

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
