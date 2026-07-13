# Environment Pre-Check

## Purpose
验证开发环境和子技能满足后续所有阶段的要求。分两阶段执行:
1. Stage 1 前:基础环境检查(所有技术栈通用) + 子技能健康检查
2. Stage 1.5 后:技术栈特定检查(根据用户选择的技术栈)

## Check Items

### 1. 基础环境检查(Stage 1 前执行)

| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| Node.js | 前端(design-taste-frontend/frontend-design) + 冒烟测试 | 18 | `node -version` |
| npm/pnpm | 前端依赖管理 | npm 9+ / pnpm 8+ | `npm -version` |
| Playwright | 冒烟测试浏览器自动化 | 最新 | `npx playwright --version` |
| Git | 提交/回滚 | 2.30+ | `git --version` |

### 2. 技术栈特定检查(Stage 1.5 后执行)

#### Java Spring Boot 后端
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| JDK | 后端编译 | 17 | `java -version` |
| Maven 或 Gradle | 后端构建 | Maven 3.6+ / Gradle 7+ | `mvn -version` 或 `gradle -version` |

#### Go Gin 后端
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| Go | 后端编译 | 1.21 | `go version` |

#### Node.js NestJS 后端
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| Node.js | 后端运行 | 18 | `node -version` |
| npm/pnpm | 依赖管理 | npm 9+ / pnpm 8+ | `npm -version` |

#### Python FastAPI 后端
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| Python | 后端运行 | 3.11 | `python --version` |
| pip/poetry | 依赖管理 | pip 23+ / poetry 1.6+ | `pip --version` |

### 3. 代码质量工具检查(Stage 3 前执行)

#### 通用工具
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| SonarQube Scanner | 静态代码分析 | 5.0+ | `sonar-scanner --version` |
| Prettier | 代码格式化 | 3.0+ | `npx prettier --version` |

#### Java 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| Checkstyle | Java 代码规范检查 | 10.0+ | `mvn checkstyle:check` |
| SpotBugs | Java 静态分析 | 4.8+ | `mvn spotbugs:check` |

#### Go 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| golangci-lint | Go 代码规范检查 | 1.55+ | `golangci-lint --version` |
| gosec | Go 安全扫描 | 2.x | `gosec --version` |

#### Node.js 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| ESLint | JavaScript/TypeScript 代码规范 | 8.50+ | `npx eslint --version` |
| Semgrep | 安全扫描 | 1.x | `semgrep --version` |

#### Python 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| flake8 | Python 代码规范检查 | 6.0+ | `flake8 --version` |
| mypy | Python 类型检查 | 1.5+ | `mypy --version` |
| Bandit | Python 安全扫描 | 1.7+ | `bandit --version` |

## Sub-Skill Health Check Definitions

Each community sub-skill has a specific health check procedure. Run these during env-check to verify the skill produces valid output.

### brainstorming (requirement)
- **Input**: A single topic word, e.g. "task-scheduler"
- **Expected output**: Markdown with YAML front-matter containing at least:
  - `artifact` field
  - `version` field (v1 or similar)
  - A `target_users` or `overview` section
- **Validation**:
  1. Parse YAML front-matter → must exist and be valid YAML
  2. Check front-matter has `artifact` key
  3. Check body contains at least 200 characters of text
  4. Check for `target_users` OR `overview` section heading
- **Pass criteria**: All 4 checks pass
- **Failure handling**: Log error to `skill_check.health_checks.brainstorming`, try alternatives, then fallback

### ui-ux-pro-max (product_design)
- **Input**: A short description, e.g. "A task scheduling dashboard with weekly summaries"
- **Expected output**: Markdown with YAML front-matter containing:
  - `artifact` field
  - `feature_list` section with ≥1 item
  - `user_journey` section with Mermaid flowchart syntax
- **Validation**:
  1. YAML front-matter parses successfully
  2. `feature_list` section exists with at least 1 item containing `name` and `priority`
  3. `user_journey` section contains a Mermaid code block (```mermaid ... ```)
- **Pass criteria**: All 3 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### prd-development (prd)
- **Input**: A requirement summary, e.g. "User management module with CRUD operations"
- **Expected output**: Markdown with YAML front-matter containing:
  - `artifact` field
  - `module_specs` section with ≥1 module
  - `data_models` section with ≥1 model
  - `api_contracts` section with ≥1 endpoint
- **Validation**:
  1. YAML front-matter parses successfully
  2. `module_specs` section exists with at least 1 module having `module_name` and `user_stories`
  3. `data_models` section exists with at least 1 model having `name` and `fields`
  4. `api_contracts` section exists with at least 1 endpoint having `method` and `path`
- **Pass criteria**: All 4 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### design-taste-frontend (prototype)
- **Input**: A page description, e.g. "Login page with username/password fields and remember me checkbox"
- **Expected output**: An HTML file (or markdown with embedded HTML) containing:
  - Valid HTML structure (`<!DOCTYPE html>`)
  - At least one interactive element (`<a>`, `<button>`, or `<form>`)
  - Viewport meta tag
- **Validation**:
  1. Output contains `<!DOCTYPE html>` or `<html` tag
  2. Contains at least one `<button` or `<a ` or `<form` tag
  3. Contains `<meta` viewport tag
- **Pass criteria**: All 3 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### frontend-design (frontend_arch)
- **Input**: A feature description, e.g. "Admin panel with user management and settings pages"
- **Expected output**: Markdown with YAML front-matter containing:
  - `artifact` field
  - `component_tree` section with ≥1 component
  - `directory_structure` section
  - `routing` section with ≥1 route
- **Validation**:
  1. YAML front-matter parses successfully
  2. `component_tree` section exists with at least 1 component having `name` and `responsibility`
  3. `directory_structure` section exists and is non-empty
  4. `routing` section exists with at least 1 route entry
- **Pass criteria**: All 4 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### test-driven-development (test_case)
- **Input**: A module description, e.g. "User service with login and registration"
- **Expected output**: Markdown with YAML front-matter containing:
  - `artifact` field
  - `functional_cases` section with ≥1 case
  - `smoke_cases` section with ≥1 case
- **Validation**:
  1. YAML front-matter parses successfully
  2. `functional_cases` section exists with at least 1 case having `id`, `title`, `steps`, `expected_result`
  3. `smoke_cases` section exists with at least 1 case having `id`, `endpoint`, `method`, `expected_status`
- **Pass criteria**: All 3 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### security-and-hardening (security)
- **Input**: An architecture summary, e.g. "Spring Boot REST API with JWT auth and PostgreSQL"
- **Expected output**: Markdown with security recommendations containing:
  - Authentication/Authorization section
  - Input validation section
  - Data protection section
- **Validation**:
  1. Output is non-empty (>100 characters)
  2. Contains at least 2 of: "auth", "encrypt", "validate", "cors", "token", "jwt", "xss", "sql injection"
- **Pass criteria**: All 2 checks pass
- **Failure handling**: Log error, fallback to built-in security review

### executing-plans (code_generation)
- **Input**: A simple task, e.g. "Create a Hello World REST endpoint in Spring Boot"
- **Expected output**: Code files with:
  - At least one `.java` or `.kt` file (for Java) or `.go` file (for Go) etc.
  - Valid code structure (imports, class/function definition)
- **Validation**:
  1. Output contains at least one code block with language tag
  2. Code block has ≥10 lines
  3. Contains at least one function/method definition pattern
- **Pass criteria**: All 3 checks pass
- **Failure handling**: Log error, try alternatives, then fallback

### doubt-driven-development (cross_review)
- **Input**: A code snippet or architecture description
- **Expected output**: Markdown with critical review containing:
  - At least one identified risk or issue
  - A suggested improvement or alternative
- **Validation**:
  1. Output is non-empty (>200 characters)
  2. Contains at least one section with "risk", "issue", "concern", or "vulnerability"
  3. Contains at least one section with "improve", "alternative", "suggest", or "consider"
- **Pass criteria**: All 3 checks pass
- **Failure handling**: Log error, fallback to built-in review

### dispatching-parallel-agents (parallel_dispatch)
- **Input**: Two independent task descriptions
- **Expected output**: Confirmation that both tasks were dispatched, with task IDs
- **Validation**:
  1. Output contains at least 2 task references (IDs or names)
  2. Indicates concurrent/parallel execution
- **Pass criteria**: Both checks pass
- **Failure handling**: Fall back to sequential execution (built-in)

## Health Check Execution Procedure

1. For each community skill listed in `skills`:
   a. Load the skill definition
   b. Execute the skill with its health check input (see definitions above)
   c. Validate the output against the expected validation rules
   d. Record result in `skills.<key>.health_check_passed` (true/false)
   e. Record detailed result in `skills.<key>.health_check_result`
2. After all checks, compute `skill_check.passed`:
   - `true` if all skills have `health_check_passed: true`
   - `false` if any skill failed (but at least one fallback exists)
   - `failed` if a skill failed AND has no fallback
3. If `skill_check.passed` is `false`, log which skills failed and their fallbacks in `execution_log`

## Execution Steps

### 步骤 1: 基础环境检查(Stage 1 前)
1. 依次执行基础工具的检查命令
2. 解析输出,提取版本号
3. 与最低版本对比
4. 将结果写入 `docs/workflow-state.yaml` 的 `env_check.base` 段
5. 全部通过 → 进入子技能健康检查
6. 任一缺失/版本不足 → 列出缺失项,询问用户是否手动安装后继续

### 步骤 2: 子技能健康检查(Stage 1 前)
1. 遍历 `skills` 段中所有社区技能
2. 对每个技能执行对应的 health check（见上方定义）
3. 记录结果到 `skills.<key>.health_check_passed` 和 `health_check_result`
4. 将汇总结果写入 `skill_check.health_checks`
5. 全部通过 → 进入 Stage 1
6. 任一失败 → 尝试 alternatives，再失败则 fallback 到 built-in-chat，记录到 `fallbacks`

### 步骤 3: 技术栈特定检查(Stage 1.5 后)
1. 根据 `tech_stack.backend` 的值,执行对应的工具检查
2. 将结果写入 `docs/workflow-state.yaml` 的 `env_check.tech_specific` 段
3. 更新 `env_check.passed` 为最终结果
4. 如果缺失技术栈特定工具,列出缺失项,询问用户是否安装后继续

### 步骤 4: 代码质量工具检查(Stage 3 前执行)
1. 根据 `tech_stack.backend` 和 `tech_stack.frontend` 的值,执行对应的代码质量工具检查
2. 将结果写入 `docs/workflow-state.yaml` 的 `env_check.code_quality` 段
3. 如果代码质量工具缺失,列出警告(非阻塞),建议用户安装
4. 记录检查结果,用于 Stage 3 的代码质量验证

## Fallback 策略

### 环境工具跳过
- 如果用户明确表示此项目**不需要某个工具**,允许跳过对应检查
- 跳过需在 `env_check` 中标注 `skipped: true` 和 `reason`
- 技术栈特定工具检查会根据选择自动跳过无关项

### 子技能降级
- 当首选技能不可用时,按顺序尝试 alternatives 列表中的备选技能
- 如果所有备选都不可用,使用 fallback 指定的降级方案(通常是 built-in-chat)
- fallback 使用会在 `fallbacks` 日志中记录,并在对应 gate 时提醒用户注意审查
- 健康检查失败 ≠ 技能不可用：健康检查失败表示技能可用但输出不符合预期，应优先尝试 alternatives

## 输出示例

### Stage 1 前(基础检查)
```
[env-check] 检测基础环境...
  ✓ Node.js 18.18.0
  ✓ npm 9.8.1
  ✓ Playwright 1.42.0
  ✓ Git 2.42.0

[skill-health] 子技能健康检查...
  ✓ brainstorming (v1.2.0, health OK, output validated)
  ✓ ui-ux-pro-max (v3.1.0, health OK, output validated)
  ✓ prd-development (abc1234, health OK, output validated)
  ✓ design-taste-frontend (v2.0.0, health OK, output validated)
  ✓ frontend-design (v1.0.0, health OK, output validated)
  ✓ test-driven-development (v4.0.0, health OK, output validated)
  ✓ security-and-hardening (built-in, health OK)
  ✓ executing-plans (v2.1.0, health OK, output validated)
  ✓ doubt-driven-development (v3.0.0, health OK, output validated)

[env-check] 基础检查完成,进入 Stage 1...
```

### Stage 1.5 后(技术栈特定检查)
```
[env-check] 用户选择技术栈: Go 1.21 + Gin + Vue 3 + Nuxt + PostgreSQL

[env-check] 检测技术栈特定工具...
  ✓ Go 1.22.0
  ✓ Node.js 18.18.0 (已检查)

[env-check] 技术栈检查完成
  工具: ✓ 全部通过
  子技能: ✓ 全部可用(后端架构为 built-in)
是否继续? [Y/N]
```
