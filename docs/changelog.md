# Changelog

## v2.0.0 (2026-08-12)

- **Breaking — Coder 角色升级**：从机械执行改为证据驱动的受约束实现；允许边界内自主决策，禁止擅自改变产品、接口、依赖、持久化、架构和风险承诺。
- **双真相模型**：用户确认的 PRD/验收决定目标状态；代码、测试、配置、Schema 和依赖决定当前状态；任务文档属于待验证指令。
- **三种执行模式**：新增 `BOUNDED_AUTONOMY`、`EXACT_CHANGE`、`INVESTIGATE_FIRST`。
- **分歧裁决闭环**：新增结构化分歧单、依赖感知暂停、Architect 独立核实、用户交互决策、superpowers 方案修订和任务版本恢复门。
- **当前工作模式**：接入 claude/codex/opencode/mico/zcode 角色、模式 A/B 双盲验证、验证白名单、codex 断流替补、最多 5 commit 和 ≤200 字派发总结。
- **Java 后端规范**：新增命名、参数、排版、分层、异常、日志、SQL 和 MySQL 规则 reference，Java 任务显式选择并声明例外。
- **归档与派发**：归档升级为 `<slug>/<YYYY-MM-DD>/`、`coder/<YYYY-MM-DD>/`、`reviewer/<YYYY-MM-DD>/`；使用 Coder 环境原生绝对路径和精简 dispatch。
- **审查加强**：未关闭交付分歧、口头裁决未落入新任务版本、越过强制升级边界和 reviewer 硬约束违反均为 BLOCKER。
- **兼容性**：既有 v1.x 任务按 `EXACT_CHANGE` 继续执行，已派发任务不静默迁移。

## v1.3.0 (2026-06-26)

文件归档结构重构：

- **归档结构从 `<repo>/.claude/<YYYY-MM-DD>/` 改为 `<repo>/.claude/<slug>/`**，引入需求/任务 slug 维度
- Coder 专属产出放入 `<repo>/.claude/<slug>/coder/`（coder-task、coder-report）
- 共用产出（tech-design、requirement）放 `.claude/<slug>/` 下，一份即可
- Architect 产出（codebase-scan、prd-code-mapping、qa-regression-prompt、tech-review-prompt）放 `.claude/<slug>/` 下
- SKILL.md、references/、templates/ 全部产出物清单和落盘路径同步更新
- marketplace.json、plugin.json 版本号同步至 1.3.0

## v1.0.1 (2026-06-24)

边界澄清 + 扫描降本能力：

**边界澄清（核心）**：
- **技术方案 + 执行计划改走 superpowers:brainstorming + writing-plans**，cdel 不自己出方案。cdel 的独有职责收窄为：复用 superpowers 产出 → 机读化为 Coder 任务文档 → 委托弱模型编码 → 审查
- 删除 `references/execution-plan-spec.md` + `templates/execution-plan.md`（执行计划格式由 superpowers 管）
- `coder-prompt-spec.md` §4 动作语义声明自包含（insert_after/before/replace/delete/new_file），不再依赖 execution-plan-spec
- SKILL.md §0/§2/§4/§5/§6/§7/§8、spec §2/§3/§5/§10、AGENTS 角色表/流程/产出物/文件结构、README Why/Roles/Flow 全面同步边界

**扫描降本能力（新增）**：
- 新增 `references/scan-delegation.md`：代码扫描委托规范（触发条件 / 能力探测 / 两路径 / 摘要结构 / 落盘复用）
- 新增 `templates/codebase-scan-summary.md`：代码扫描摘要模板（模块职责/关键类签名/调用链/数据流/影响面，面向设计决策）
- 大改动委托弱模型扫描出摘要，强模型只消费摘要做机读化，降 token 成本
- 两路径：有 workflow 子 agent 能力 → 启动弱模型子 agent 并行扫描；无 → 询问用户用其他弱模型 session 出摘要
- 小改动（1-2 方法）Architect 直接读，不委托（避免为降本反增协调成本）

**round-2 AC 双盲交叉验证补全**（codex + opencode 评审后修复）：

- F1/F2：spec §4/§8 残留"Architect 出方案"旧边界 → 同步为"无方案引导 superpowers"
- F3：3 份 manifest description（plugin.json/marketplace.json/codex-plugin）残留"Architect produces/designs plan" → 全部改为"reuses superpowers plan → machine-readize"
- F4：SKILL.md FAQ "Step 7" 残留 → 改 Step 6（7→6 步顺延遗漏）
- F5-F8：scan-delegation 补全——能力探测具体方法（工具列表/skill 列表/运行时特征）、路径 A 失败兜底（启动失败/部分失败/缺段质量门）、增量扫描判定机制（git diff 比对覆盖列表+重扫阈值）、重写 superpowers 边界段消除"cdel 独有 vs superpowers 也可用"矛盾
- F9：coder-prompt-spec §4 补 superpowers→cdel 动作语义适配说明（Architect 机读化时补动作字段）
- F10/F11：coder-prompt-spec + spec 文件头版本号升 v1.1（与版本记录对齐）
- F12：coder-task 模板 + spec §6 示例"方案来源"字段改为"superpowers 技术方案/执行计划来源"
- codebase-scan-summary 模板附录补"5 段完整性自查"checkbox + 覆盖文件列表字段

## v1.0.0 (2026-06-24)

首版发布。

**核心能力**：

- Architect + Coder 双角色串行编码工作流（强弱模型分工：强设计 + 弱执行）
- 执行计划混合格式（自然语言目标 + 结构化步骤）
- Coder 编码任务文档（独立、自包含、可执行）
- Coder 8 条硬约束（含计划偏差出口、自测失败反馈回路）
- Coder 自测 + 自审 checklist（7 条统一）机制
- 审查→AC 桥接（BLOCKER 直接打回 Coder；无 BLOCKER 超阈值才建议 multi-reviewer QA 模式）
- 角色抽象（Architect / Coder 不与具体工具绑定）+ 角色错配防护
- 触发方式：显式命令 `/cdel` + 对话关键词 + 短别名
- 多运行时插件清单（Claude Code / Codex / 跨运行时市场）

**工作流闭环设计**：

- BLOCKER 处理：审查发现 BLOCKER → 直接打回 Coder 修复 → 修复后重新审查（不进 AC）；AC 是独立验证，非修已知 bug
- Coder 失败回路：自测失败 / 计划偏差 / 缺依赖 → 停止并反馈 Architect → 调整计划 → 重派
- 审查通过话术：如实报 HIGH 数量（阈值允许 ≤2 HIGH 通过），不宣称"未发现 HIGH"
- 豁免退出口：Step 1 识别到纯重命名 / typo 等豁免场景时建议轻量方式并退出 cdel

**已知待优化项**（留后续版本）：

- spec 内部版本号（v1.x）与插件版本号独立，文档内补充说明
- 计划偏差出口现挂于硬约束 3（不调未声明依赖）下，语义关联不直接，未来版本考虑独立为约束 9
