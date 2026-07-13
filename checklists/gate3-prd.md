# Gate 3 Checklist: PRD Sign-off

## Purpose
Gate 3 **仅验证 PRD 文档质量**。PRD 决定后续所有产出物的基础，需在此确认 PRD 本身完整无误。

**注意：** 前端架构、后端架构、测试用例的验证统一在 Gate 4 进行。本 Gate 不检查这些产出物。

## Auto-Validation

### 1. 文件完整性
- [ ] `docs/02-prd/prd-v1.md` 存在，含 YAML front-matter
- [ ] `depends_on` 指向 design 且其 status=confirmed

### 2. 内容完整性
- [ ] 文档信息（version、author、last_updated、status）已填写
- [ ] 功能总览清晰，模块关系明确
- [ ] 每个模块含 user_stories（≥1）和 acceptance_criteria（≥1，可验证）
- [ ] 数据模型完整，每个字段含 name/type/required/description
- [ ] 接口契约（api_contracts）与数据模型一致
- [ ] 业务规则明确，跨模块约束清晰
- [ ] 非功能需求覆盖性能/安全/可用性/兼容性
- [ ] 有状态对象定义了 state_machine
- [ ] 无 TODO/TBD/矛盾陈述

### 3. PRD 质量检查
- [ ] acceptance_criteria 含可验证条件（Given/When/Then 或等效表述）
- [ ] 每个数据模型的字段都有 type 和 required 标识
- [ ] api_contracts 引用的 data_model 均存在
- [ ] 模块间依赖关系无循环

## Human Review Prompts

建议用户重点看：
- 功能模块划分是否合理
- 用户故事和验收标准是否可测试
- 数据模型是否覆盖所有场景
- 非功能需求是否明确

- 选项 A: 确认，进入 Stage 2 产出物生成
- 选项 B: 需要修改（用户指明哪部分）
- 选项 C: 重新做 PRD

## On Reject
- 选 B: 标记 prd 为 `status: draft`，按用户方向修订，版本递增 v2，重新过 Gate 3
- 选 C: 清空 prd，重做 Stage 2.②
- 任一修改触发 change-propagation: 下游 prototype、frontend_arch、backend_arch、test_case 标记 stale
