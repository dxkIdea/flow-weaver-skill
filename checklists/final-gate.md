# Final Gate Checklist: Code Delivery Sign-off

## Purpose
Final gate 在 Stage 3 冒烟测试 100% 通过后执行。用户确认后正式交付。

## Auto-Validation

### 1. 代码完整性
- [ ] 后端代码已按 backend_arch 的 module_decomposition 全部生成
- [ ] 每个模块的代码结构符合分层架构(controller/service/mapper/entity/dto)
- [ ] 前端代码已按 frontend_arch 生成
- [ ] 前端代码的组件树与 architecture 一致

### 2. 测试结果
- [ ] 所有模块单元测试通过
- [ ] 冒烟测试 100% 通过(0 失败)
- [ ] `docs/workflow-state.yaml` 中 `stage3.smoke_test.last_result` = pass
- [ ] 无 known issue 被跳过(若有,需用户明确知晓)

### 3. 一致性
- [ ] 代码实现的功能与 PRD 模块一致
- [ ] 后端 API 实现与 openapi.yaml 一致
- [ ] 前端调用的接口与后端实现一致

### 4. 工程质量
- [ ] 代码已全部 git commit
- [ ] `pre-smoke-test` tag 存在
- [ ] 无未提交的改动
- [ ] 项目可正常构建(mvn compile 或 npm run build)
- [ ] 项目可正常启动

### 5. 代码质量检查
- [ ] 代码已格式化(Prettier/eslint/gofmt/black)
- [ ] 无严重代码规范问题(Checkstyle/golangci-lint/ESLint/flake8)
- [ ] 无安全漏洞(如 SQL 注入、XSS、硬编码密钥)
- [ ] 代码覆盖率达标(单元测试覆盖率 ≥ 60%)
- [ ] SonarQube 分析通过(无 Critical/Bug 级别问题)

**技术栈特定代码质量检查:**

**Java 项目:**
- [ ] Checkstyle 检查通过
- [ ] SpotBugs 无 Critical/Bug 级别问题
- [ ] Maven 构建无警告

**Go 项目:**
- [ ] golangci-lint 检查通过
- [ ] go vet 无问题
- [ ] go fmt 已执行

**Node.js 项目:**
- [ ] ESLint 检查通过
- [ ] TypeScript 编译无错误
- [ ] Prettier 格式化完成

**Python 项目:**
- [ ] flake8 检查通过
- [ ] mypy 类型检查通过
- [ ] black 格式化完成

## Delivery Summary (呈现给用户)

需要呈现以下信息供用户决策:
1. 代码统计:后端模块数、前端页面数、总代码行数
2. 测试统计:单元测试用例数、冒烟测试用例数、通过率
3. 产出物清单:8 个文档/原型/代码的位置
4. 已知问题(若有)

## Human Review Prompts

- 选项 A: 确认交付
- 选项 B: 需要调整某些部分(指明)
- 选项 C: 整体重做 Stage 3

## On Approve (选 A)
- 更新 `docs/workflow-state.yaml`: `workflow.status` = completed
- 输出最终交付清单
- 流程结束

## On Reject
- 选 B: 按用户指明部分调整,重跑相关测试
- 选 C: 回退到 `pre-smoke-test` tag,清空 Stage 3 已生成代码,重新生成
