# Gate 2 Checklist: Product Design Sign-off

## Purpose
Gate 2 在 Stage 2.① 完成后检查设计文档质量,再提交人工确认。

## Auto-Validation

### 1. 文件完整性
- [ ] `docs/01-product/design-v1.md` 存在,含 YAML front-matter
- [ ] `depends_on` 指向 requirement 且其 status=confirmed

### 2. 内容完整性
- [ ] 设计概述、功能清单、用户旅程、信息架构均已填写
- [ ] Mermaid 图表语法无错误

## Human Review Prompts

- 选项 A: 确认,进入 PRD 阶段
- 选项 B: 需要修改(用户指明哪部分)
- 选项 C: 重新做产品设计

## On Reject
- 选 B: 修订 design,版本递增 v2,重新过 Gate 2
- 选 C: 清空 design,重做 2.①
- 任一修改触发 change-propagation:下游 prd 等标记 stale
