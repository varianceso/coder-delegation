# 当前工作模式协议

- 版本：v2.0.0 (2026-08-12)
- 适用：Architect/Coder 协同与独立交叉验证 reviewer
- 目标：让插件原生支持当前工作模式，不依赖项目私有 rules 打补丁

## 1. 角色映射

| 角色/运行时 | 职责 | 写入边界 |
|-------------|------|----------|
| `claude` / Architect | 需求澄清、superpowers 方案与计划、编码协同、单测、本地集成测试、生成评审 prompt、收集报告并裁决 | 可按当前工作模式实施；不得把自己作为最终 QA reviewer |
| `codex` | 独立方案评审（模式 B）和编码后 QA 回归（模式 A），经 PPIO 运行 gpt-5.6 | 只读源码/文档/环境，报告只写指定归档目录 |
| `opencode` | 与 codex 同等的独立方案评审和 QA 回归 | 同 codex，双盲并行 |
| `mico` | `.claude/mico/` 项目自有 lead/coder/qa/reviewer-* 等角色 | 按角色只读评审非 AC commit；工作虾可按授权修改飞书文档 |
| `zcode` | codex 断流后的弱模型接跑替补 | 继承被替换任务的全部边界 |
| 用户 | 需求、方案、PR、产品/契约/范围/风险分歧仲裁 | 不读取不应暴露的 reviewer 私有过程 |

Architect/Coder 是抽象角色，可由 mico lead/coder、codex 或 zcode 承担；具体绑定以任务文档声明为准。

## 2. 交叉验证模式

### 模式 A：编码后回归

实现、单测和本地冒烟完成后，Architect 生成 `qa-regression-prompt`，分别交给 codex 和 opencode 双盲执行接口、curl 或 CLI 验证。每个 reviewer 独立写 `qa-report`，Architect 收集两份报告后对比裁决。Architect 不把自己的判断性自测结论写入 prompt，只提供可复现事实、测试角色名、数据规模和已批准契约。

### 模式 B：编码前方案评审

PRD、技术方案、测试方案或上线方案定稿后，Architect 生成对应 `review-prompt`，分别交给 codex 和 opencode，只读文档并核查源码事实，输出独立 `evaluation`。产品方案评审 B1 可按用户指定只派一个 reviewer，不伪装成双盲多数意见。

## 3. 双盲原则

- codex 与 opencode 不读取对方同期或历史报告，不提前接收对方结论。
- reviewer 可以回看自己上一轮同类报告，但不得看对方报告。
- `<repo>/.claude/<slug>/` 下的 `index`、`summary`、PRD、方案和决策史是主 agent 主权区，reviewer 不读不写。
- 交叉验证产出只落主仓归档，不镜像到 reviewer 私有仓。
- reviewer prompt 禁止内联大段源码和 Architect 判断结论，使用路径、事实和核查目标。

## 4. 验证白名单

Coder 自测和 reviewer 回归均只运行任务文档明确列出的白名单命令：编译命令必须带 `-pl <受影响模块> -am`（若项目构建工具支持），测试命令必须列出具体测试类，多个类用逗号分隔，禁止用 `+` 连接造成 surefire 空跑假绿。

禁止扩大到“相关测试”“顺便回归”或未改动链路。额外测试暴露基线债（例如引用已删除类导致 compile 失败）时，立即停止扩大验证，报告阻断/风险；只有 Architect 明确授权才可扩大。鉴权失败不得修改配置、关闭安全校验、伪造 token 或绕过服务，标记为“未验证（鉴权失败 + 原因）”并继续不依赖鉴权的检查。

## 5. 断流与交付节奏

codex 经 PPIO 调用 gpt-5.6 时，单个请求最多重试 2 次；仍断流即记录“未完成（断流）”。同一子任务连续两次断流，切换 zcode 接跑，zcode 继承原任务版本、边界和报告路径。单轮盲审或编码交付最多涉及 5 个 commit；插件本身和 Coder 均禁止执行 Git 写操作。

所有派发给 codex、opencode、mico 或 Coder 的 prompt 结尾必须要求返回标准化、可直接复制的总结，正文不超过 200 字。完整报告仍必须写入任务指定的归档文件。

## 6. 归档路径与所有权

```text
<repo>/.claude/<slug>/<YYYY-MM-DD>/                       主 agent 产物
<repo>/.claude/<slug>/coder/<YYYY-MM-DD>/                 Coder 产物
<repo>/.<reviewer>/<slug>/reviewer/<YYYY-MM-DD>/          reviewer 私有过程
```

主 agent 归档目录始终是 `.claude/`；跨验证报告只写主仓的 `<repo>/.claude/<slug>/<YYYY-MM-DD>/`。reviewer 私有目录只承载自身临时过程和独立 evaluation，不得复制主 agent 决策史。slug 使用 kebab-case，跨会话以 `<slug>/summary.md` 维护决策史。

## 7. 派发协议

派发 prompt 不内联源码、方案正文、工作目录、分支、报告路径或硬约束，只提供一个已验证的 Coder 环境原生绝对任务文档路径和执行要求。实际输出必须是一个独立代码块：

```text
请读取并执行编码任务文档：/absolute/path/to/.claude/<slug>/coder/coder-task-<name>.md
```

不要在代码块前后添加说明。环境或共享文件系统不明确时先询问用户。路径不可使用 `~`、环境变量、相对路径、`file://` URI、未解析模板变量或混合分隔符。

## 8. Reviewer 硬约束

以下任一违反即为 BLOCKER：

1. 只读不写源码、配置、pom、package.json；发现问题写报告，不自行修复。
2. 归档目录外不落盘；临时文件使用 `/tmp` 或 `%TEMP%`。
3. 不执行 `git add`、`commit`、`push`、`reset --hard`、`checkout --`。
4. 交付脱敏，不输出真实 token、姓名、工号、邮箱、手机号或连接串。
5. 不清理 `~/.m2`、`node_modules` 或用户级环境变量，不动共享环境；启动进程验证后立即 kill。

## 9. 版本记录

- v2.0.0 (2026-08-12)：接入当前工作模式的角色、双盲、白名单、归档、派发和断流协议。
