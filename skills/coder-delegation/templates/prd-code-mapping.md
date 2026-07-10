# PRD↔代码双向映射：{{任务名}}

> 生成时间：{{YYYY-MM-DD HH:mm}}
> 映射人：Architect
> PRD 来源：{{<CALENDAR_PLATFORM>文档转 MD 路径/链接}}（⚠️ 只读，不可修改——产品 source of truth）
> 技术方案来源：{{superpowers:writing-plans 产出的 tech-design 文档路径}}
> 用途：coder-task 机读化前置依据——确保 PRD 每个功能点有对应代码入口，coder-task 每个改动文件能追溯到 PRD 功能点

## 1. PRD → 代码（正向映射）

PRD 每个功能点（F-xxx）对应的代码入口/DTO/Service/DAO：

| PRD 功能点 | 功能简述 | 代码入口 | 涉及 DTO/Req | 涉及 Service | 涉及 DAO/Mapper | 核实状态 |
|-----------|---------|---------|-------------|-------------|----------------|---------|
| F-001 | {{简述}} | `{{Controller.method}}` @ `{{file:line}}` | `{{XxxReq}}` | `{{XxxService.method}}` | `{{XxxMapper.method}}` | ✅ / ⚠️ |
| F-002 | {{简述}} | `{{Controller.method}}` @ `{{file:line}}` | `{{XxxReq}}` | `{{XxxService.method}}` | — | ✅ / ⚠️ |

> 每行一个功能点。若一个功能点有多个入口（如前端+CLI），分行列出。

## 2. 代码 → PRD（反向映射）

coder-task 列出的每个改动文件必须能追溯到 PRD 功能点，找不到映射 = 越权改动或遗漏：

| 改动文件 | 改动动作 | 追溯 PRD 功能点 | 是否映射 |
|---------|---------|---------------|---------|
| `{{完整相对路径}}` | {{修改/新增/删除}} | F-00x | ✅ / ⚠️ 未映射 |
| `{{完整相对路径}}` | {{修改/新增/删除}} | F-00x | ✅ |

> 出现 ⚠️ 未映射 → 停止，确认文件是否真的需要改（可能是越权改动），或 PRD 遗漏该功能点。

## 3. 跨仓调用链核实

> 若任务涉及跨仓调用（前端→后端、CLI→后端等），必填本节。无跨仓可删。

| 调用方（仓） | 实际调用路径（扫码确认，非推断） | 接口/DTO | 核实状态 |
|------------|------------------------------|---------|---------|
| CLI | `PreviewAdjustItemReq → AgentPlanPreviewService → CreateDeptHandler.handle()` | `PreviewAdjustItemReq` | ✅ |
| 前端 | `POST /api/dept/upsert → DeptController.upsert() → DeptServiceImpl.upsertDept()` | `DeptUpsertReq` | ✅ |

**常见陷阱**：
- CLI 不走通用 CRUD 接口，走 agent 专属路径——不要凭"CLI 也是改部门"推断调用路径
- 跨仓接口可能有适配层（DTO 转换），不要漏掉

## 4. 字段透传链路完整性

> 若任务涉及字段透传/新增字段，必填本节。无字段透传可删。

```
字段: {{fieldName}} ({{fieldType}})

入口层          → 校验层          → 转换层          → 执行层              → 落库层
{{Req.field}}     {{Validator}}     {{Converter}}     {{Service.process()}}   {{Mapper.insert()}}
                                                     {{Handler.setField()}}
```

每环标注对应改动文件和 Step 编号。空白环 = 遗漏，必须在 coder-task 补全。

## 5. 框架/组件 API 用法参考

> 若任务涉及框架组件（antd、Element UI 等），必填本节。无框架组件可删。

| 组件 | 项目内用法锚点 | API 要点 |
|------|--------------|---------|
| `TextArea` | `src/pages/XxxEdit.tsx:128` | `autoSize`（非 `autosize`），`onChange(e => e.target.value)`（非 `onChange(val)`） |
| `Select` | `src/components/XxxSelect.tsx:45` | `onChange={(value, option) => ...}`，`labelInValue` |

> 每个 API 引用必须来自项目内实际代码（扫码确认），不能凭通用知识填写。

## 6. 核实方法说明

| 核实项 | 核实方法 |
|--------|---------|
| 行号引用 | `git show origin/<base>:<file>` 确认目标行内容匹配 |
| 方法签名 | 读实际文件确认方法名+参数+返回值 |
| 调用链 | 读 Controller → Service → Handler 实际代码路径 |
| 组件 API | 在项目中搜索组件用法，找现有 file:line 锚点 |
| DTO 字段 | 读 DTO 类确认字段名+类型+注解 |

## 附录：未覆盖/不确定项

{{列出映射过程中无法确认的 PRD 功能点或代码入口，说明原因和后续处置}}

- F-00x "{{功能简述}}"：代码入口未找到，可能涉及新模块——已反馈用户确认
- `{{XxxHandler}}` 调用链：跨仓代码不可达——标 ⚠️ 待 Coder 验证
