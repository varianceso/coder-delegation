# Java 后端编码规范

- 版本：v2.0.0 (2026-08-12)
- 适用：Java 后端新增或修改代码的 Coder 任务；纯 typo、注释文字调整、纯重命名和一次性脚本可豁免
- 规则语义：`必须` 强制，`禁止` 不可，`建议` 推荐，`避免` 尽量不

## 1. 命名

- 类/接口/枚举使用 `UpperCamelCase`，方法/变量/参数使用 `lowerCamelCase`，常量使用 `UPPER_WITH_UNDERSCORE`，包名全小写。
- 禁止拼音、非常规缩写和以下划线/美元符开头或结尾；通用缩写可使用 `Func`、`IO`、`Tcp`、`Xml`。
- 接口使用 `*Provider` 或 `*Repository`，实现使用 `*Impl`；DO 使用 `*DO`；API 模型使用 `*VO`、`*Dto`、`*Req`；API 转换器使用 `*Convertor`，Infra 转换器使用 `*Converter`。
- 方法使用明确动作：单项 `get`、列表 `list`、统计 `count`、写入 `insert/save`、删除 `remove/delete`、修改 `update`；禁止 `doSomething`、`process`、`handle` 等模糊命名。
- 布尔变量优先用语义名（如 `deleted`）；枚举一般不加 `Enum` 后缀。

## 2. 方法、参数与排版

- 方法入参最多 5 个；超出时使用 `Command`、`Query`、`Param` 等对象封装并通过 converter 转换。
- 公共分页查询 Req 继承 common 层 `BaseReq`，由基类收敛分页、排序校验和归一化；列表上限按场景设置。
- 排序值使用 domain 层模块专用枚举，Convertor 负责 String 到枚举转换和非法值默认降级。
- 方法体建议不超过 80 行，嵌套不超过 3 层，使用卫语句保持主干清晰；集合返回不得为 null。
- 单行硬限 150 字符，120 字符以内优先；4 空格缩进、Unix 换行、K&R 花括号、一行一语句、禁止通配符 import 和 FQN（反射字符串等必要例外除外）。能自然放入一行且不超 150 字符的调用不要拆行。

## 3. 分层红线

| 层 | 允许职责 | 禁止事项 |
|----|----------|----------|
| API | 协议、基础校验、可信上下文提取、转换、异常映射 | 业务规则 |
| Application | 用例编排、事务、权限、日志上下文和协调 | 直接调用 Mapper、DO 或 SDK |
| Domain | 业务状态、规则、列表行为和端口 | 依赖基础设施实现 |
| Infra | 实现 Domain 端口，隔离 DO、Redis、UC、FDS、Groovy、SDK | 将外部类型泄漏到上层 |
| Repository | 数据源、DO、Mapper/XML、事务保护和 DDL | 业务判断 |
| Common | 无业务语义且真实复用的基础能力 | 依赖业务模块 |

DO 不得离开 Repository/Infra；API 模型不得进入 Repository；外部 SDK 类型不得进入 Domain/Application/API。过滤、排序、校验、标准化、匹配、降级、分组和状态判断优先放 Domain。

## 4. 数据、并发与事务

- Mapper 禁止 `SELECT *`；UPDATE/DELETE 必须有可走索引的 WHERE；列表必须分页或限量；循环内禁止逐条查 DB，批量取数后内存组装。
- 集合使用接口类型；批量集合按预期容量初始化；Map 遍历使用 `entrySet`；并发场景使用 `ConcurrentHashMap`。
- 线程资源必须由 `ThreadPoolExecutor` 和拒绝策略提供，禁止 `Executors` 和显式创建线程；日期使用 `DateTimeFormatter` 或隔离后的安全实现。
- `@Transactional` 明确事务边界和 `rollbackFor`，只读查询标 `readOnly=true`，异常必须显式抛出或处理。BigDecimal 使用 `compareTo`。
- MySQL 表使用 snake_case、单数表名、BIGINT 主键、小数 DECIMAL，LIMIT 配 ORDER BY，深分页使用游标；禁止外键级联；hrod-plus 建表统一 `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci`。

## 5. 异常、日志与注释

- `@Override` 必须添加；`equals` 与 `hashCode` 成对重写；工具类禁止暴露公共构造器。
- `switch` 必须有 `default`（穷举 enum 例外）；字符串比较使用 `equals`/`Objects.equals`；不得吞异常、抛通用 `Exception`/`RuntimeException`/`Throwable` 或在 finally 中控制流程。
- 业务异常使用 `BizException`/`LegacyException` 和模块错误码；GlobalExceptionHandler 统一映射 400/403/404/500，并保持 HTTP status 与 body code 一致。
- 资源使用 try-with-resources；日志记录完整堆栈但脱敏，禁止 `System.out`、`printStackTrace`、真实 token/身份/连接串。
- 公共 API 和核心领域逻辑写说明意图、设计理由、状态迁移和步骤骨架；源码注释、Javadoc、日志禁止内部任务编号（HR、Phase、AC、ARC 等），使用业务语言。

## 6. 任务执行要求

Java 任务文档必须声明本 reference 是否适用，并在 §3 自主边界中列出适用例外。Coder 发现现有代码与规范冲突时，不做无关存量治理；只对本任务新增/修改代码遵守本规范，必要的例外写入决策摘要或分歧单。自测命令仍受验证白名单约束。

## 版本记录

- v2.0.0 (2026-08-12)：将用户提供的 Java 后端命名、参数、排版、分层、数据、异常、日志、注释和 MySQL 规则纳入 cdel Coder 任务协议。
