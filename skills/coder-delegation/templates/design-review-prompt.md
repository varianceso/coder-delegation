# 模式 B：编码前方案评审任务（{{reviewer}}）

## 0. 任务状态

- **协议版本**：`v2.0.0`
- **评审类型**：`{{B1 产品方案 / B2 技术方案 / B3 测试方案 / B4 上线方案}}`
- **任务 ID/slug**：`{{kebab-case slug}}`
- **reviewer**：`{{codex / opencode / mico / zcode / cursor / cline / qoder / aider / trae}}`
- **仓库根目录**：`{{reviewer 环境原生绝对路径}}`
- **基线 commit**：`{{完整 SHA}}`
- **待核查 commit 范围**：`{{无 / 1–5 个 commit 的精确范围}}`
- **评审文档**：`{{reviewer 可访问的原生绝对路径}}`
- **报告路径**：`{{主仓 .claude/<slug>/<YYYY-MM-DD>/ 下的原生绝对路径}}`
- **私有过程目录**：`{{.<reviewer>/<slug>/reviewer/<YYYY-MM-DD>/ 原生绝对路径}}`

## 1. 独立性边界

- B2/B3/B4 默认由 codex 和 opencode 双盲并行；B1 产品方案允许单 reviewer。
- 不读取另一 reviewer 的任何报告，不读取 `<repo>/.claude/<slug>/` 的 `index`、`summary` 和决策史。
- 只读指定方案和为核查事实必要的源码、配置、Schema、测试与依赖元数据。
- 不修改方案或源码，不执行 Git 写操作，不清共享环境；问题写入 evaluation。
- 不把命名、历史说明或委托摘要当作当前事实，决定性结论必须有 `DIRECT` 或 `REPRODUCED` 锚点。

## 2. 评审目标

| ID | 核查问题 | 目标真相来源 | 需要核查的当前事实 |
|----|----------|--------------|--------------------|
| RV-001 | {{完整性/契约/安全/可测性问题}} | `{{文档条款}}` | `{{代码/配置/Schema 范围}}` |

## 3. 只读核查白名单

```bash
{{rg/git show/构建/具体测试类等允许命令}}
```

禁止扩大到无关模块。鉴权失败或基线债按未验证项记录，不修改配置绕过。

## 4. Evaluation 格式

将完整 evaluation 写入指定主仓报告路径：

```markdown
# Evaluation：{{reviewer}} / {{评审类型}}
## 1. 评审范围与独立性声明
## 2. 方案覆盖矩阵
## 3. 源码事实核查（DIRECT / REPRODUCED / INFERRED / UNVERIFIED）
## 4. Findings（BLOCKER / HIGH / MEDIUM / LOW / NOTE）
## 5. 分歧与需用户裁决项
## 6. 未验证项和风险
## 7. 硬约束自检
## 8. 可复制总结（≤200字）
```

报告落盘后，聊天只返回标准化、可直接复制的总结（≤200 字）。

## 5. 断流规则

codex/PPIO 单次请求最多重试 2 次；仍断流报告“未完成（断流）”。同一子任务连续两次断流，由主 agent 将同一输入版本交给 zcode 接跑。
