# Environment Pre-Check

## Purpose
验证开发环境和子技能满足后续所有阶段的要求。分两阶段执行:
1. Stage 1 前:基础环境检查(所有技术栈通用)
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

### 3. 子技能可用性检查

| 子技能 | 来源 | 用途 | 依赖 |
|---|---|---|---|
| brainstorming | obra/superpowers | 需求探索与确认 | 通用 |
| ui-ux-pro-max | nextlevelbuilder | 产品设计 | 通用 |
| prd-development | pm-skills | PRD生成 | 通用 |
| design-taste-frontend | Leonxlnx/taste-skill | UI原型设计 | 通用 |
| frontend-design | anthropics/skills | 前端架构 | 通用 |
| test-driven-development | obra/superpowers | 测试用例 | 通用 |
| security-and-hardening | opencode built-in | 安全审查 | 通用 |
| executing-plans | obra/superpowers | 代码生成执行 | 通用 |
| doubt-driven-development | obra/superpowers | 交叉审查 | 通用 |

后端架构为编排层内建(built-in),无需外部技能。

### 4. 代码质量工具检查(Stage 3 前执行)

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

#### Node.js 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| ESLint | JavaScript/TypeScript 代码规范 | 8.50+ | `npx eslint --version` |

#### Python 项目
| 工具 | 用途 | 最低版本 | 检查命令 |
|---|---|---|---|
| flake8 | Python 代码规范检查 | 6.0+ | `flake8 --version` |
| mypy | Python 类型检查 | 1.5+ | `mypy --version` |

## Execution Steps

### 步骤 1: 基础环境检查(Stage 1 前)
1. 依次执行基础工具的检查命令
2. 解析输出,提取版本号
3. 与最低版本对比
4. 将结果写入 `docs/workflow-state.yaml` 的 `env_check` 段
5. 全部通过 → 进入 Stage 1
6. 任一缺失/版本不足 → 列出缺失项,询问用户是否手动安装后继续

### 步骤 2: 子技能可用性检查(Stage 1 前)
1. 遍历 `skills` 段中所有技能
2. 尝试加载每个技能,记录可用性状态
3. 对于不可用的技能,检查是否有备选技能(alternatives)
4. 将结果写入 `docs/workflow-state.yaml` 的 `skill_check` 段:
   - `skill_check.passed`: 所有技能可用或有 fallback
   - `skill_check.unavailable_skills`: 完全不可用的技能列表
   - `skill_check.fallback_required`: 需要使用 fallback 的技能列表

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

## 输出示例

### Stage 1 前(基础检查)
```
[env-check] 检测基础环境...
  ✓ Node.js 18.18.0
  ✓ npm 9.8.1
  ✓ Playwright 1.42.0
  ✓ Git 2.42.0

[skill-check] 检测子技能...
  ✓ brainstorming (obra/superpowers)
  ✓ ui-ux-pro-max (nextlevelbuilder)
  ✓ prd-development (pm-skills)
  ✓ design-taste-frontend (Leonxlnx)
  ✓ frontend-design (anthropics)
  ✓ test-driven-development (obra/superpowers)
  ✓ security-and-hardening (opencode built-in)
  ✓ executing-plans (obra/superpowers)
  ✓ doubt-driven-development (obra/superpowers)

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
