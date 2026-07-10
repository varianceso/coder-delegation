# coder-delegation (cdel)

> **多角色 AI 编码协作** — 强模型把方案转成任务文档，弱模型照文档写代码，强模型审查验收。

Claude Code / Codex / Zcode / Cursor / Cline 多运行时技能插件，编排串行工作流：
**Architect**（强模型）复用 `superpowers` 产出的技术方案 + 执行计划 → 转为 Coder 任务文档
→ **Coder**（弱模型）按文档编码 → **Architect** 审查，可选桥接到
[multi-reviewer](https://github.com/varianceso/multi-reviewer) 双盲审计。

## 为什么用

- 强模型做机械编码太贵
- 弱模型做架构判断和方案设计靠不住
- 把"判断力密集"工作给强模型，"机械执行"给弱模型，降本不降质
- 大改动时委托弱模型扫描出代码摘要，进一步降低强模型 token 消耗

## 角色

| 角色 | 模型 | 职责 |
|------|------|------|
| **Architect** | 强 | 复用 superpowers 方案/执行计划 → 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档 → 审查 |
| **Coder** | 弱 | 读任务文档 → 编码 → 自测 → 自审 checklist → 交付 diff |

## 快速开始

### 安装

> 私有仓库 — 使用 SSH git URL（Claude Code / Codex 内部通过 `git clone` 拉取）。

**Claude Code：**
```
/plugin marketplace add https://github.com/varianceso/coder-delegation.git
/plugin install coder-delegation
```

**Codex：**
```bash
codex plugin marketplace add https://github.com/varianceso/coder-delegation.git
codex plugin add coder-delegation@coder-delegation-marketplace
```

### 使用

```
/cdel 给 UserService 加权限校验
```

对话中直接说："帮我实现 XX 功能" 也可触发。

## 工作流

```
用户：/cdel <需求>
  → （前置）superpowers:brainstorming + writing-plans 出技术方案 + 执行计划
  → Architect：复用方案 + 扫码（大改动委托弱模型出摘要）→ 生成 Coder 任务文档
  → 用户：审阅确认
  → Coder：编码 → 自测 → 自审 → 交付
  → Architect：审查 → 通过 / 打回 Coder 修复（BLOCKER） / 建议 AC 盲审
```

> cdel **不自己出方案** — 技术方案 + 执行计划由 `superpowers:brainstorming` + `superpowers:writing-plans` 产出。
> cdel 只负责把它们转为 Coder 任务文档、委托弱模型编码、审查。

## 与 multi-reviewer 的协作

Architect 审查 Coder 交付物时：

- 发现 **BLOCKER**（功能不可用 / 数据错误 / 安全漏洞）→ **直接打回 Coder 修复**，不进 AC（AC 是独立验证，不是修已知 bug）
- 无 BLOCKER 但超阈值（>3 文件 / >150 行 / ≥3 个 HIGH 问题）→ 建议桥接到 [multi-reviewer](https://github.com/varianceso/multi-reviewer) 做独立双盲审计

## 文件结构

```
coder-delegation/
├── AGENTS.md                   ← Agent 指令（优先读）
├── README.md                   ← 本文件
├── LICENSE
├── marketplace.json            ← 根级市场入口（SSH 安装用）
├── .claude-plugin/             ← Claude Code 插件清单
├── .codex-plugin/              ← Codex 插件清单
├── .agents/plugins/            ← 跨运行时市场
├── docs/
│   ├── spec-coder-delegation.md  ← 设计规格
│   └── changelog.md            ← 变更日志
└── skills/
    ├── cdel/SKILL.md            ← 短别名
    └── coder-delegation/
        ├── SKILL.md            ← 主入口
        ├── references/         ← 协议深水区
        └── templates/          ← 文档模板
```

## License

MIT — 详见 [LICENSE](./LICENSE)。
