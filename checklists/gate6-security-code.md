# Gate 6 Checklist: Code-Level Security Scan

## Purpose
Gate 6 在 Stage 3 代码生成完成后、Final Gate 前执行。对**已生成代码**进行静态安全扫描。

## Auto-Validation

### Static Analysis Results
- [ ] 已运行对应技术栈的安全扫描工具：
  - Java: SpotBugs / SonarQube（无 Critical/Bug 级别安全问题）
  - Go: golangci-lint + gosec（无 HIGH 级别问题）
  - Node.js: ESLint security plugin / Semgrep（无 HIGH 级别问题）
  - Python: Bandit / flake8-security（无 HIGH 级别问题）
- [ ] 扫描结果已记录到 `security_review.code_review.findings`
- [ ] 无未修复的 Critical/High 级别安全问题

### Common Security Checks
- [ ] 无硬编码密钥/密码/API key
- [ ] 无 SQL 拼接（使用参数化查询/ORM）
- [ ] 无明文存储密码（使用 bcrypt/argon2）
- [ ] 无不安全的反序列化
- [ ] 无调试端点暴露在生产配置中
- [ ] 依赖无已知 CVE 漏洞（可选：运行 `npm audit` / `mvn dependency-check` / `pip-audit`）

## Human Review Prompts

- 选项 A: 全部通过，进入 Final Gate
- 选项 B: 发现安全问题，修复后重扫
- 选项 C: 有已知风险需接受（记录为 known issue）

## On Reject
- 选 B: 修复安全问题后重新扫描
- 选 C: 记录为 known issue，进入 Final Gate 时告知用户

## Note
此 Gate 是代码级安全审查。Gate 5（架构安全审查）已在 Stage 2.7 完成。
