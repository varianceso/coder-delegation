# coder-delegation (cdel)

> **多角色 AI 编码协作** — 强模型复用方案并核实证据，Coder 基于真实代码受约束实现，分歧由 Architect 或用户裁决。

支持 Claude Code、Codex、OpenCode、mico、Zcode、Cursor、Cline 等运行时。技术方案与计划由 superpowers 产出；cdel 将其变成证据驱动的任务合同，组织实现、自测、审查和独立双盲验证。

## 核心变化

- Coder 不再机械照抄完整补丁步骤，而是在明确边界内自主选择局部实现。
- 目标真相来自用户/验收/PRD，当前真相来自代码、测试、配置、Schema 和依赖证据。
- 事实不符、契约不明或安全风险必须写入分歧单，并暂停受影响依赖分支。
- Java 后端任务内置命名、参数、150 字符行限、分层、异常、日志、SQL 和 MySQL 规范。
- 自测与 reviewer 回归只跑验证白名单；基线债和鉴权失败不绕过。
- 支持 codex/opencode/mico 双盲 review，以及 codex 断流后切 zcode。

## 安装

Claude Code：

```text
/plugin marketplace add https://github.com/varianceso/coder-delegation.git
/plugin install coder-delegation
```

Codex：

```bash
codex plugin marketplace add https://github.com/varianceso/coder-delegation.git
codex plugin add coder-delegation@coder-delegation-marketplace
```

## 使用

```text
/cdel 给 UserService 加权限校验
```

完整流程：

```text
superpowers 方案/计划
  → Architect 双真相映射与证据核实
  → v2 Coder 任务合同
  → Coder 有界自主实现 + 白名单自测 + 自审
  → 分歧裁决和任务版本恢复
  → Architect 独立审查
  → 可选模式 A 双盲 QA
```

## 证据驱动 Coder

每个实现单元使用一种模式：

- `BOUNDED_AUTONOMY`：默认业务实现，Coder 在文件、契约、依赖和验收边界内自主决定。
- `EXACT_CHANGE`：安全常量、协议文本、迁移和 v1.x 兼容任务，按精确内容执行。
- `INVESTIGATE_FIRST`：先回答证据问题，符合假设才继续，否则开分歧单。

任务文档、方案和扫描摘要都不是不可质疑事实。决定性结论需要源码、配置、Schema 或可复现命令证据。

## 分歧如何裁决

Coder 将分歧写入文件并暂停受影响单元。Architect 独立核实事实；局部技术事实由 Architect 裁决，产品/公共契约/范围/依赖/持久化/风险由用户决定，设计冲突返回 superpowers。裁决必须写入新任务版本，Coder 才能恢复。影响交付的分歧未 `CLOSED` 时不能完成。

## 两种独立验证

- 模式 A：编码后 QA 回归。codex 与 opencode 独立跑指定接口、curl 或 CLI，落两份报告，Architect 对比裁决。
- 模式 B：编码前方案评审。codex 与 opencode 独立读方案并核查源码事实；B1 产品方案可单 reviewer。

两者不看对方报告，也不读主 agent 的 summary/决策史。Architect 不把自己的判断性测试结论写进 prompt，不自己做最终 QA。

## 验证与归档

验证白名单必须给出模块化编译命令和具体测试类。Maven 使用 `-pl <模块> -am` 缩小 reactor；多个测试类用逗号分隔。不得“顺便回归”未改动链路。

```text
<repo>/.claude/<slug>/<YYYY-MM-DD>/
<repo>/.claude/<slug>/coder/<YYYY-MM-DD>/
<repo>/.<reviewer>/<slug>/reviewer/<YYYY-MM-DD>/
```

主 agent 产物始终在 `.claude/`，交叉验证产出只落主仓。每次编码/评审范围最多 5 个 commit。

## 跨平台派发

实际派发只输出一个任务文档原生绝对路径，不内联正文：

```text
请读取并执行编码任务文档：/Users/name/project/.claude/user-auth/coder/2026-08-12/coder-task-user-auth.md
```

```text
请读取并执行编码任务文档：C:\Users\name\project\.claude\user-auth\coder\2026-08-12\coder-task-user-auth.md
```

```text
请读取并执行编码任务文档：/mnt/c/Users/name/project/.claude/user-auth/coder/2026-08-12/coder-task-user-auth.md
```

不允许 `~`、环境变量、相对路径、`file://`、未解析变量或混合分隔符。

## AC 阈值

未关闭分歧、越界实现或 reviewer 硬约束违反先计 BLOCKER 并暂停。无 BLOCKER 时，文件 >3、行 >150 或 HIGH ≥3 建议用户确认后进行独立盲审；其余可由 Architect 审查通过。

## License

MIT，见 [LICENSE](./LICENSE)。
