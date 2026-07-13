# Gate 5 Checklist: Architecture Security Review

## Purpose
Gate 5 在 Stage 2.7 执行，对**架构层面**的安全设计进行审查。此时尚未生成代码，只能审查设计方案。

## Auto-Validation

### Backend Architecture Security Design
- [ ] backend_arch 包含 authentication 设计（认证方案：JWT/OAuth2/session？）
- [ ] backend_arch 包含 authorization 设计（RBAC/ABAC/权限模型？）
- [ ] backend_arch 包含 data_encryption 设计（传输加密 TLS、存储加密 AES/hash？）
- [ ] backend_arch 包含 audit_log 设计（操作日志记录策略）
- [ ] backend_arch 包含全局异常处理和错误码规范（不泄露敏感信息）
- [ ] backend_arch 包含输入校验策略（参数校验、SQL 注入防护、XSS 防护）
- [ ] backend_arch 包含 CORS/CSP 配置策略
- [ ] openapi.yaml 中标注了安全要求（securitySchemes）

### Frontend Architecture Security Design
- [ ] frontend_arch 包含 XSS 防护策略（模板转义、 CSP header）
- [ ] frontend_arch 包含 CSRF 防护策略（same-site cookie、token 验证）
- [ ] frontend_arch 包含敏感数据存储策略（不在 localStorage 存 token/密码）
- [ ] frontend_arch 包含 API 调用安全策略（HTTPS only、token 自动刷新）

### Non-Functional Security Requirements
- [ ] PRD 的非功能需求中包含安全要求（性能/安全/可用性/兼容性）
- [ ] 至少定义了密码策略（复杂度/过期/哈希算法）

## Human Review Prompts

建议用户重点看：
- 认证方案是否适合项目规模（微服务用 JWT/OAuth2，单体用 session 即可）
- 数据加密策略是否覆盖传输和存储两端
- 输入校验是否覆盖了所有用户可控入口

- 选项 A: 确认，进入代码生成
- 选项 B: 需要修改（指明哪部分安全设计）
- 选项 C: 重新做安全审查

## On Reject
- 选 B: 修订架构文档中的安全设计，版本递增，重新过 Gate 5
- 选 C: 清空 security_review.arch_review，重做 Stage 2.7

## Note
此 Gate 仅审查架构设计。代码级安全扫描将在 Stage 3 代码生成后、Final Gate 前执行（gate6_security_code）。
