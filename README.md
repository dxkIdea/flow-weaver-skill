# Flow Weaver Skill

> From one-line idea to shipped code with smoke tests — a meta-skill that orchestrates the entire delivery pipeline.
> 从一句话想法到通过冒烟测试的已交付代码 —— 一个编排完整交付流水线的元技能。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-3.2-green.svg)](.)
[![Codex Compatible](https://img.shields.io/badge/codex-compatible-brightgreen.svg)](.)

---

<!-- TOC -->
- [What Is Flow Weaver / 这是什么](#what-is-flow-weaver--这是什么)
- [Quick Start / 快速开始](#quick-start--快速开始)
- [Pipeline Overview / 流水线概览](#pipeline-overview--流水线概览)
  - [Full Mode (Complete Pipeline) / 完整模式](#full-mode-complete-pipeline--完整模式)
  - [Quick Mode (Dynamic Trimming) / 快速模式](#quick-mode-dynamic-trimming--快速模式)
  - [Gate Summary / 门禁汇总](#gate-summary--门禁汇总)
- [Execution Rules / 执行规则](#execution-rules--执行规则)
- [Sub-Skill Registry / 子技能注册表](#sub-skill-registry--子技能注册表)
- [Tech Stack Support / 技术栈支持](#tech-stack-support--技术栈支持)
- [Templates / 项目脚手架](#templates--项目脚手架)
- [File Structure / 文件结构](#file-structure--文件结构)
- [State Management / 状态管理](#state-management--状态管理)
- [Integration with AI Agents / AI Agent 集成](#integration-with-ai-agents--ai-agent-集成)
  - [Codex CLI (OpenAI Codex Desktop) / Codex CLI](#codex-cli-openai-codex-desktop--codex-cli)
  - [Claude Code / Claude Code](#claude-code--claude-code)
  - [OpenCode / OpenCode](#opencode--opencode)
  - [Cursor / VS Code / Cursor / VS Code]
  - [Other Agents / 其他 Agent](#other-agents--其他-agent)
- [Example Invocation / 示例调用](#example-invocation--示例调用)
- [Advanced Features / 高级特性](#advanced-features--高级特性)
- [Prerequisites / 前置要求](#prerequisites--前置要求)
- [Changelog / 更新日志](#changelog--更新日志)
- [License / 许可证](#license--许可证)
<!-- /TOC -->

---

## What Is Flow Weaver / 这是什么

Flow Weaver is an **orchestrator meta-skill** for AI coding agents. It turns a rough one-line idea into fully tested, shipped code by coordinating 10+ stages across 7 human confirmation gates, delegating each stage to specialized sub-skills, and persisting state throughout.

Flow Weaver 是一个面向 AI 编程 Agent 的**编排型元技能**。它将一个粗糙的一句话想法转化为经过完整测试、可通过冒烟测试的已交付代码 —— 通过协调 10+ 个阶段、7 个人工确认门禁、将每个阶段委派给专用子技能，并在整个过程中持久化状态。

Think of it as a **CI/CD pipeline for the ideation-to-code journey** — but driven by AI agents with human oversight at every critical decision point.

把它想象成一条**从想法到代码的 CI/CD 流水线** —— 由 AI Agent 驱动，但在每个关键决策点都需要人工确认。

---

## Quick Start / 快速开始

```
User: 启动 flow-weaver, 我想做一个团队周报自动汇总工具
```

That's it. The pipeline handles the rest: environment check → requirements → design → PRD → prototype → architecture → test cases → security review → code generation → smoke test → delivery.

就这样。流水线会自动处理后续所有步骤：环境检查 → 需求 → 设计 → PRD → 原型 → 架构 → 测试用例 → 安全审查 → 代码生成 → 冒烟测试 → 交付。

Or for a faster path:
更快的路径：
```
User: 启动 flow-weaver --quick
```

---

## Pipeline Overview / 流水线概览

### Full Mode (Complete Pipeline) / 完整模式

```
[env-check] Environment + Skills ready?
  ↓
Stage 1: Requirement brainstorming (skill: brainstorming)
  ↓ GATE 1: Requirement sign-off
Stage 1.3: Project Type Selection (microservice|monolith|frontend-only|api-only)
  ↓
Stage 1.5: Tech Stack Selection
  ↓
Stage 1.6: Cost & Risk Assessment (skippable)
  ↓
Stage 2: Artifact generation (parallel groups)
  ① product_design (skill: ui-ux-pro-max) → GATE 2: Design sign-off
  ② prd (skill: prd-development)          → GATE 3: PRD sign-off
  ┌─ ③ prototype (skill: design-taste-frontend) ┐
  └─ ⑤ backend_arch (built-in) ─────────────────┘→ Group A (parallel, git-branch-isolated)
                                               ↓
  ④ frontend_arch (skill: frontend-design)    → Group B (depends on prototype)
                                               ↓
  ⑥ test_case (skill: test-driven-development)→ Group C (depends on backend_arch)
                                               ↓ GATE 4: Artifacts sign-off
Stage 2.7: Security Review - Architecture Layer (security-and-hardening) → GATE 5
  ↓
Stage 3: Template instantiate → Code generation + smoke test
  Per-module generation → unit test per module → smoke test
  ↓
  Post-generation: Security Review - Code Layer (static analysis) → GATE 6
  ↓ Final Gate
[Final gate] → Deliver
```

### Quick Mode (Dynamic Trimming) / 快速模式

When `workflow.mode = quick`, stages are trimmed based on `project_type`:

当 `workflow.mode = quick` 时，根据 `project_type` 动态裁剪阶段：

| Project Type / 项目类型 | Skipped Stages / 跳过阶段 | Skipped Gates / 跳过门禁 | Schema Changes / 验证调整 |
|---|---|---|---|
| `frontend-only` | product_design, backend_arch, backend_code_generation, security_arch | gate2_design | test_case smoke_cases optional |
| `api-only` | prototype, frontend_arch, frontend_code_generation | — | prototype-schema skip, frontend-arch skip |
| `microservice` | product_design | gate2_design | prd depends on requirement (not product_design) |
| `monolith` | (none — full pipeline) | (none) | — |

User can also force quick mode: `"启动 flow-weaver --quick"`
用户也可以强制使用快速模式：`"启动 flow-weaver --quick"`

### Gate Summary / 门禁汇总

| Gate / 门禁 | Stage | Validates / 验证内容 | Checklist / 检查清单 |
|---|---|---|---|
| Gate 1 | Stage 1 | Requirement document quality | `checklists/gate1-requirement.md` |
| Gate 2 | Stage 2.① | Product design completeness | `checklists/gate2-design.md` |
| Gate 3 | Stage 2.② | PRD only (functional overview, module specs, data models, API contracts) | `checklists/gate3-prd.md` |
| Gate 4 | Stage 2 end | All Stage 2 artifacts batch-wise (prototype, frontend_arch, backend_arch, test_case) + cross-artifact consistency | `checklists/gate4-artifacts.md` |
| Gate 5 | Stage 2.7 | Architecture-level security design (auth, encryption, input validation) | `checklists/gate5-security-arch.md` |
| Gate 6 | Post-Stage 3 | Code-level static analysis (no Critical/High findings) | `checklists/gate6-security-code.md` |
| Final Gate | — | Code completeness, test pass rate, consistency, quality, security, delivery summary | `checklists/final-gate.md` |

---

## Execution Rules / 执行规则

| Rule | Name / 名称 | Key Point / 要点 |
|------|---|---|
| 0 | Explicit trigger only / 显式触发 | Never auto-invoke; must be explicitly requested |
| 1 | State persistence / 状态持久化 | `docs/workflow-state.yaml` is the single source of truth |
| 1.5 | Quick mode trimming / 快速模式裁剪 | Apply `quick_mode_trim` after project type selection |
| 2 | Parallel execution + write-path isolation / 并行执行 + 写入路径隔离 | Git-branch isolation for Group A; sub-agents can only write to assigned paths |
| 3 | Subagent isolation / 子代理隔离 | Each stage runs in an independent subagent |
| 4 | Artifact contract / 产物契约 | Every artifact includes YAML front-matter (type, version, dependencies, schema) |
| 5 | Sub-skill version locking + health check / 子技能版本锁定 + 健康检查 | Record version + run health check for every community skill |
| 6 | Smoke test execution format / 冒烟测试执行格式 | Auto-selected per tech stack (curl/bash, Python, REST-Assured, Playwright) |
| 7 | Two-layer security review / 双层安全审查 | Gate 5 (architecture) + Gate 6 (code-level static analysis) |
| 8 | Gate 3/4 separation / Gate 3/4 职责分离 | Gate 3 = PRD only; Gate 4 = all Stage 2 artifacts |
| 9 | Template instantiation / 模板实例化 | Instantiate project scaffolds before code generation |
| 10 | Manual edit detection / 手动编辑检测 | Detect user edits at each gate; prompt to preserve or regenerate |
| 11 | Execution log archival / 执行日志归档 | Archive oldest 500 entries when `execution_log` exceeds 1000 |
| 12 | Resume checkpoint / 断点恢复 | Resume from last checkpoint with prerequisite validation |

---

## Sub-Skill Registry / 子技能注册表

| Artifact / 产物 | Skill / 技能 | Source / 来源 | Type / 类型 |
|---|---|---|---|
| requirement | brainstorming | [obra/superpowers](https://github.com/obra/superpowers) | community |
| product_design | ui-ux-pro-max | [nextlevelbuilder/skills](https://github.com/nextlevelbuilder/skills) ⭐24.7k | community |
| prd | prd-development | [pm-skills](https://github.com/dingxingkai/pm-skills) | community |
| prototype | design-taste-frontend | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | community |
| frontend_arch | frontend-design | [anthropics/skills](https://github.com/anthropics/skills) ⭐160k | community |
| backend_arch | *(built-in)* | — | built-in |
| test_case | test-driven-development | [obra/superpowers](https://github.com/obra/superpowers) | community |
| security_arch | security-and-hardening | OpenCode built-in | community |
| code_generation | executing-plans | [obra/superpowers](https://github.com/obra/superpowers) | community |
| cross_review | doubt-driven-development | [obra/superpowers](https://github.com/obra/superpowers) | community |
| parallel_dispatch | dispatching-parallel-agents | [obra/superpowers](https://github.com/obra/superpowers) | community |

**Banned / 禁用:** `api-and-interface-design` — explicitly excluded from this workflow.

---

## Tech Stack Support / 技术栈支持

| Category / 类别 | Options / 选项 |
|---|---|
| Backend / 后端 | Java 17 + Spring Boot 3.x · Go 1.21+ + Gin · Node.js 18+ + NestJS · Python 3.11+ + FastAPI |
| Frontend / 前端 | React + Next.js + TypeScript + Tailwind · Vue 3 + Nuxt 3 + TypeScript + Tailwind |
| Database / 数据库 | PostgreSQL · MySQL · SQLite · MongoDB |

---

## Templates / 项目脚手架

During Stage 3, Flow Weaver instantiates the appropriate template from `templates/`:

在 Stage 3 代码生成前，Flow Weaver 会从 `templates/` 中实例化对应的技术栈模板：

| Template / 模板 | Language/Framework / 语言框架 | Structure / 目录结构 |
|---|---|---|
| `templates/backend-spring-boot/` | Java 17 + Spring Boot 3.x + Maven | controller/service/mapper/entity/dto |
| `templates/backend-go-gin/` | Go 1.21+ + Gin | handler/service/repository/model/dto |
| `templates/backend-nestjs/` | Node.js + NestJS + TypeScript | controller/service/module/dto/entity |
| `templates/backend-fastapi/` | Python + FastAPI | api/core/models/schemas/services |
| `templates/frontend-react-next/` | React + Next.js + TS + Tailwind | app/components/lib/styles |
| `templates/frontend-vue-nuxt/` | Vue 3 + Nuxt 3 + TS + Tailwind | app/components/composables/stores/layouts |
| `templates/openapi-default.yaml` | OpenAPI 3.x | Starter spec with health endpoint |

All templates include a buildable skeleton with health endpoint, standard layered directory structure, and stack-appropriate `.gitignore`.
所有模板均包含可构建的骨架代码（带健康检查端点）、标准的分层目录结构和对应技术栈的 `.gitignore`。

---

## File Structure / 文件结构

```
flow-weaver-skill/
├── SKILL.md                           # Main definition, 12 rules, pipeline overview
├── README.md                          # This file
├── rules/
│   ├── workflow-state-template.yaml   # State file template (single source of truth)
│   ├── env-check.md                   # Environment pre-check + sub-skill health checks
│   ├── change-propagation.md          # Stale artifact handling + dependency graph
│   ├── retry-policy.md                # 3-level failure handling + rollback rules
│   ├── timeout-retry-config.yaml      # Per-stage timeout thresholds
│   ├── logging.md                     # Execution log format + audit rules
│   ├── progress-visualization.md      # Progress dashboard rules
│   ├── progress-template.html         # HTML dashboard template
│   └── progress-template.json         # JSON data template for dashboard
├── schemas/                           # Per-artifact validation schemas
│   ├── requirement-schema.yaml        # Requirement confirmation
│   ├── product-design-schema.yaml     # Product design
│   ├── prd-schema.yaml                # PRD
│   ├── prototype-schema.yaml          # UI prototype
│   ├── frontend-arch-schema.yaml      # Frontend architecture
│   ├── backend-arch-schema.yaml       # Backend architecture (Java example)
│   └── test-case-schema.yaml          # Test cases + execution formats
├── checklists/                        # Gate validation checklists
│   ├── gate1-requirement.md           # Requirement sign-off
│   ├── gate2-design.md                # Product design sign-off
│   ├── gate3-prd.md                   # PRD sign-off (PRD only)
│   ├── gate4-artifacts.md             # Stage 2 artifacts batch
│   ├── gate5-security-arch.md         # Architecture security review
│   ├── gate6-security-code.md         # Code-level security scan
│   └── final-gate.md                  # Final delivery sign-off
├── templates/                         # Project scaffolds per tech stack
│   ├── backend-spring-boot/
│   ├── backend-go-gin/
│   ├── backend-nestjs/
│   ├── backend-fastapi/
│   ├── frontend-react-next/
│   ├── frontend-vue-nuxt/
│   └── openapi-default.yaml
└── templates/README.md                # Template usage guide
```

---

## State Management / 状态管理

All pipeline state is persisted in `docs/workflow-state.yaml` (copied from `rules/workflow-state-template.yaml` at start).

所有流水线状态持久化在 `docs/workflow-state.yaml` 中（启动时从 `rules/workflow-state-template.yaml` 复制）。

| Section / 字段 | Purpose / 用途 |
|---|---|
| `workflow` | Pipeline ID, mode (full/quick), current stage, status |
| `quick_mode_trim` | Stage trimming rules per project type |
| `skills` | Sub-skill config with version locking + health check results |
| `write_paths` | Allowed write directories per artifact (isolation) |
| `artifacts` | Registry of all produced artifacts with version/status |
| `gates` | Gate confirmation records (pending/confirmed/rejected) |
| `stage2.parallel_groups` | Group A/B/C execution state + git branches |
| `stage3` | Template instantiation, module progress, smoke test results |
| `security_review` | Two-layer security: arch_review + code_review |
| `resume_checkpoint` | Resume target stage + prerequisites + stale artifacts |
| `manual_edits` | Detection of user manual modifications to artifacts |
| `stage_execution` | Per-stage timing, retries, timeout tracking |
| `fallbacks` | Record of skill fallbacks |
| `iterations` | Version history for all artifacts |
| `execution_log` | Audit trail (archived to file when >1000 entries) |

---

## Integration with AI Agents / AI Agent 集成

Flow Weaver is designed as a **skill file** (`SKILL.md`) that can be integrated into any AI coding agent that supports the OpenAI Codex skill format. The skill format is portable and agent-agnostic.

Flow Weaver 设计为标准 skill 文件（`SKILL.md`），可集成到任何支持 OpenAI Codex skill 格式的 AI 编程 Agent 中。该格式具有可移植性，不绑定特定 Agent。

### Codex CLI (OpenAI Codex Desktop) / Codex CLI

**Supported: ✅ Fully compatible / 支持：✅ 完全兼容**

Codex CLI is the native home for skills. Flow Weaver integrates seamlessly.

Codex CLI 是 skill 的原生运行环境，Flow Weaver 可直接无缝集成。

**Installation / 安装:**

```bash
# Method 1: Symlink to Codex skills directory (recommended)
ln -s /path/to/flow-weaver-skill \
  ~/.codex/skills/flow-weaver

# Method 2: Symlink to agents skills directory
ln -s /path/to/flow-weaver-skill \
  ~/.agents/skills/flow-weaver

# Method 3: Copy into skills directory
cp -r /path/to/flow-weaver-skill \
  ~/.codex/skills/flow-weaver
```

After installation, restart Codex CLI. The skill will appear in the sidebar and can be triggered with:
安装后重启 Codex CLI，skill 将出现在侧边栏中，可通过以下命令触发：
```
启动 flow-weaver, [your idea / 你的想法]
```

**Prerequisites / 前置条件:** Codex CLI must have the required sub-skills installed (brainstorming, ui-ux-pro-max, prd-development, etc.).

### Claude Code (Anthropic) / Claude Code

**Supported: ✅ Compatible / 支持：✅ 兼容**

Claude Code supports skill files in the same Codex-compatible format. The `SKILL.md` with YAML front-matter (`name`, `description`) is recognized by Claude Code's skill loading mechanism.

Claude Code 支持与 Codex 兼容的 skill 文件格式。带有 YAML front-matter（`name`、`description`）的 `SKILL.md` 可被 Claude Code 自动识别。

**Installation / 安装:**

```bash
# Clone or copy into Claude Code's skills directory
mkdir -p ~/.claude/skills
cp -r /path/to/flow-weaver-skill ~/.claude/skills/flow-weaver

# Or symlink
ln -s /path/to/flow-weaver-skill \
  ~/.claude/skills/flow-weaver
```

Alternatively, place the skill in your project directory — Claude Code auto-discovers `SKILL.md` files in the workspace and parent directories.
或者将 skill 放在项目根目录 —— Claude Code 会自动发现工作区和父目录中的 `SKILL.md` 文件。

**Usage / 用法:**
```
> 启动 flow-weaver, 我想做一个任务调度系统
```

**Prerequisites / 前置条件:** Claude Code needs the same sub-skills installed. Community skills can be installed via Claude Code's skill registry or placed directly in the skills directory.

### OpenCode / OpenCode

**Supported: ✅ Compatible / 支持：✅ 兼容**

OpenCode uses the same skill format and auto-discovers `SKILL.md` files. Flow Weaver's built-in skills (backend_arch, project type selection, tech stack recommendation) work natively without external dependencies.

OpenCode 使用相同的 skill 格式，自动发现 `SKILL.md` 文件。Flow Weaver 的内建技能（后端架构、项目类型选择、技术栈推荐）无需外部依赖即可原生运行。

**Installation / 安装:**

```bash
# Place in project root (auto-discovered)
cp -r /path/to/flow-weaver-skill ./flow-weaver-skill

# Or in OpenCode's global skills directory
mkdir -p ~/.opencode/skills
ln -s /path/to/flow-weaver-skill \
  ~/.opencode/skills/flow-weaver
```

**Usage / 用法:**
```
> 启动 flow-weaver, 我想做一个电商后台管理系统
```

**Note / 注意:** OpenCode's built-in `security-and-hardening` skill is automatically available. Community sub-skills may need to be installed separately.

### Cursor / VS Code / Cursor / VS Code

**Supported: ✅ Via Codex extension / 支持：✅ 通过 Codex 扩展**

If you use the Codex CLI extension in Cursor or VS Code, Flow Weaver works identically to Codex CLI — install the skill in `~/.codex/skills/` and it will be available through the Codex panel.

如果你在使用 Cursor 或 VS Code 的 Codex CLI 扩展，Flow Weaver 的使用方式与 Codex CLI 完全一致 —— 将 skill 安装到 `~/.codex/skills/` 后，即可通过 Codex 面板使用。

### Other Agents / 其他 Agent

Flow Weaver's core format is a **markdown file with YAML front-matter**, which is a widely adopted convention. Any agent that supports the following can potentially integrate Flow Weaver:

Flow Weaver 的核心格式是**带 YAML front-matter 的 Markdown 文件**，这是一种广泛采用的约定。支持以下功能的 Agent 理论上都可以集成 Flow Weaver：

1. **Skill loading**: 能够读取和解析 `SKILL.md`
2. **Sub-agent support**: Flow Weaver 将每个阶段委派给独立的子代理执行
3. **File system access**: 读写 `docs/workflow-state.yaml` 和生成产物
4. **Shell execution**: 环境检查、冒烟测试、git 操作

---

## Example Invocation / 示例调用

```
User: 启动 flow-weaver, 我想做一个团队周报自动汇总工具

AI: [env-check] 检测环境...
    ✓ JDK 17.0.8
    ✓ Maven 3.9.5
    ✓ Node.js 18.18.0
    ✓ Playwright 1.42.0
    ✓ Git 2.42.0
    [skill-health] 子技能健康检查...
    ✓ brainstorming (v1.2.0, health OK)
    ✓ ui-ux-pro-max (v3.1.0, health OK)
    ✓ prd-development (abc1234, health OK)
    ✓ design-taste-frontend (v2.0.0, health OK)
    ✓ frontend-design (v1.0.0, health OK)
    ✓ test-driven-development (v4.0.0, health OK)
    ✓ security-and-hardening (built-in, health OK)
    ✓ executing-plans (v2.1.0, health OK)
    ✓ doubt-driven-development (v3.0.0, health OK)
    
    [Stage 1] 启动 brainstorming, 基于 "团队周报自动汇总工具" 探索需求...
    [发散 → 收敛 → 需求确认书]
    [Gate 1] 请确认需求确认书...
    
    [Stage 1.3] 项目类型选择...
    请选择项目类型:
    A - monolith (单体应用, 完整流水线)
    B - microservice (微服务, 跳过产品设计)
    C - frontend-only (前端项目, 跳过后端)
    D - api-only (后端API, 跳过前端)
    
    User: A
    
    [Stage 1.5] 技术选型...
    请选择技术栈:
    A - 企业级: Java 17 + Spring Boot 3.x + React + Next.js + PostgreSQL
    B - 轻量级: Go 1.21 + Gin + Vue 3 + Nuxt + SQLite
    C - 全栈 JS: Node.js + NestJS + React + Next.js + MongoDB
    D - Python: Python 3.11 + FastAPI + React + Next.js + PostgreSQL
    
    User: A
    
    [Stage 2] 启动产出物生成...
    [Group A] dispatching-parallel-agents → 并行启动 prototype 和 backend_arch...
    [Git branches: fw-prototype-<uuid>, fw-backend-<uuid>]
    
    [Stage 3] 实例化模板...
    ✓ Backend: templates/backend-spring-boot/ → project root
    ✓ Frontend: templates/frontend-react-next/ → project root
    ✓ OpenAPI: templates/openapi-default.yaml → docs/05-backend/openapi.yaml
    [Git baseline committed]
    → 逐模块生成代码...
```

---

## Advanced Features / 高级特性

### Parallel Execution with Git Branch Isolation / 带 Git 分支隔离的并行执行

Group A (prototype + backend_arch) runs concurrently on separate git branches. After both complete, branches are merged. Conflicts are detected and stale artifacts are flagged.

Group A（原型 + 后端架构）在独立的 git 分支上并行运行。两者完成后合并分支，冲突会被检测并将相关产物标记为 stale。

### Change Propagation / 变更传播

When an upstream artifact is modified (e.g., user rejects PRD and requests changes), all downstream artifacts are automatically marked as `stale`. Affected artifacts are listed for the user to decide: regenerate all, regenerate selectively, or cancel the change.

当上游产物被修改时（例如用户拒绝 PRD 并要求修改），所有下游产物会自动标记为 `stale`。受影响产物会列出供用户决策：全部重新生成、选择性重新生成或取消变更。

### Retry & Rollback / 重试与回滚

Three-tier failure handling:

三级故障处理：
- **Level 1 / 一级**: Single module test failure → auto-diagnose → fix → retry (max 3) → rollback only that module
  单个模块测试失败 → 自动诊断 → 修复 → 重试（最多 3 次）→ 仅回退该模块
- **Level 2 / 二级**: Smoke test failure → auto-diagnose → fix → retry (max 3) → rollback to `pre-smoke-test` git tag
  冒烟测试失败 → 自动诊断 → 修复 → 重试（最多 3 次）→ 回退到 `pre-smoke-test` git tag
- **Level 3 / 三级**: Still failing → diagnostic report + human intervention (user gives direction, manual fix, skip as known issue, or retreat to Gate 4)
  仍失败 → 诊断报告 + 人工介入（用户给出修复方向、手动修复、跳过为已知问题、或退回 Gate 4）

### Resume from Interruption / 中断恢复

If the workflow is interrupted (agent disconnect, session timeout, etc.), Flow Weaver remembers exactly where it left off. On next invocation, it offers to resume from the last unconfirmed gate with prerequisite validation.

如果流程被中断（Agent 断开连接、会话超时等），Flow Weaver 会记住确切的断点。下次调用时，它会从最后一个未确认的门禁处恢复，并验证前置条件。

### Manual Edit Detection / 手动编辑检测

At each gate, Flow Weaver checks if the user manually edited any artifact since it was generated. If so, it prompts: "检测到手动修改，是否重新生成版本？"
在每个门禁处，Flow Weaver 检查用户是否手动编辑过产物。如果是，会提示："检测到手动修改，是否重新生成版本？"

### Two-Layer Security / 双层安全审查

- **Gate 5 (Architecture) / 架构层**: Reviews auth design, encryption strategy, input validation approach in architecture documents
  审查架构文档中的认证设计、加密策略、输入验证方案
- **Gate 6 (Code) / 代码层**: Runs tech-stack-specific static analyzers (SpotBugs, golangci-lint+gosec, Semgrep, Bandit) on generated code
  在生成的代码上运行技术栈特定的静态分析工具（SpotBugs、golangci-lint+gosec、Semgrep、Bandit）

### Progress Dashboard / 进度仪表盘

Real-time HTML dashboard (`docs/progress.html`) with color-coded stage timeline, per-module progress bars, timing statistics, and artifact version tracking. Auto-refreshes via AJAX polling.

实时 HTML 仪表盘（`docs/progress.html`），包含颜色编码的阶段时间线、逐模块进度条、耗时统计和产物版本追踪。通过 AJAX 轮询自动刷新。

---

## Prerequisites / 前置要求

### Required Tools / 必需工具

| Tool / 工具 | Minimum Version / 最低版本 | Purpose / 用途 |
|---|---|---|
| Node.js | 18+ | Frontend + smoke test |
| Git | 2.30+ | Rollback, branch isolation |
| JDK 17+ / Go 1.21+ / Python 3.11+ | — | Backend (depends on tech stack) |
| Playwright | Latest (optional) | Browser-based smoke tests |

### Required Sub-Skills / 必需子技能

All community sub-skills must be installed in the agent's skills directory before starting the pipeline. See [Sub-Skill Registry](#sub-skill-registry--子技能注册表) for sources.
所有社区子技能必须在启动流水线前安装到 Agent 的 skills 目录中。参见[子技能注册表](#sub-skill-registry--子技能注册表)获取来源。

### Optional Tools (Non-Blocking) / 可选工具（不阻塞流程）

| Tool / 工具 | Purpose / 用途 |
|---|---|
| SonarQube Scanner | Static code analysis |
| Prettier / ESLint / Checkstyle / golangci-lint / flake8 | Code quality |
| SpotBugs / gosec / Semgrep / Bandit | Security scanning |

---

## Changelog / 更新日志

### v3.2 (2026-07-13)
- **Quick mode data flow**: `modified_schemas` and `modified_dependencies` per project type
- **Health check definitions**: Concrete procedures for all 9 community skills
- **Template instantiation**: `stage3.template` section with source tracking
- **Manual edit detection**: `manual_edits` section + `last_modified_by_user` per artifact
- **Resume checkpoint**: `resume_checkpoint` with prerequisite validation
- **Execution log archival**: Archive to file when >1000 entries
- **Security scanning tools**: Added gosec, Semgrep, Bandit to env-check

### v3.1 (2026-07-13)
- **Quick mode**: Dynamic stage trimming by project type
- **Sub-skill version locking**: Version + health check per skill
- **Write-path isolation**: Sub-agent directory restrictions + git-branch isolation
- **Smoke test execution format**: Auto-selected per tech stack
- **Gate 3/4 separation**: Gate 3 = PRD only; Gate 4 = all Stage 2 artifacts
- **Two-layer security review**: Gate 5 (architecture) + Gate 6 (code)
- **Project templates**: 4 backends + 2 frontends + OpenAPI default

### v3.0 (2026-07-13)
- Renamed to Flow Weaver (was: vibe-coding-workflow)
- All sub-skills replaced with real open-source skills
- Added Stage 2.7 Security Review (security-and-hardening)
- Added cross-review before final gate (doubt-driven-development)
- Removed incremental build support

---

## License / 许可证

MIT
