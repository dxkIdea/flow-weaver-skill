# Gate 3 Checklist: PRD Sign-off

## Purpose
Gate 3 在 Stage 2.② 完成后检查 PRD 质量,再提交人工确认。PRD 决定后续所有产出。

## Auto-Validation

### 1. 文件完整性
- [ ] `docs/02-prd/prd-v1.md` 存在,含 YAML front-matter
- [ ] `depends_on` 指向 design 且其 status=confirmed

### 2. 内容完整性
- [ ] 功能总览、模块说明(含 user_stories + acceptance_criteria)、数据模型、接口契约均已填写
- [ ] 无 TODO/TBD
- [ ] PRD 模块覆盖 design 中所有 mvp=true 的功能

## Human Review Prompts

PRD 是后续所有产出物的源头,建议用户重点看:
- 数据模型是否合理
- 接口契约是否完整
- 验收标准是否可测

- 选项 A: 确认,进入原型+架构+测试用例生成
- 选项 B: 需要修改(指明模块)
- 选项 C: 重新做 PRD

## On Reject
- 选 B: 修订 PRD,版本递增,触发 change-propagation(prototype/arch/test_case 标记 stale)
- 选 C: 清空 PRD,重做 2.②
