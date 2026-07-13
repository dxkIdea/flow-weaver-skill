# Gate 1 Checklist: Requirement Sign-off

## Purpose
Gate 1 在 Stage 1 完成后检查 requirement 文档质量,再提交人工确认。

## Auto-Validation

### 1. 文件完整性
- [ ] `docs/00-requirement/需求确认书.md` 存在,含 YAML front-matter
- [ ] status 为 `draft` 或 `confirmed`

### 2. 内容完整性
- [ ] 项目概述、目标用户、核心场景、成功标准均已填写
- [ ] 无 TODO/TBD/矛盾陈述

## Human Review Prompts

校验通过后,使用 `AskUserQuestion` 让用户确认:
- 选项 A: 确认定稿,进入 Stage 2
- 选项 B: 需要修改(用户提供修改方向)
- 选项 C: 重新做需求确认(回到 Stage 1 开头)

## On Reject
- 选 B: 标记 requirement 为 `status: draft`,按用户方向修订,重新过 Gate 1
- 选 C: 清空 requirement,重新执行 Stage 1
