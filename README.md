# Flow Weaver Skill

> From one-line idea to shipped code with smoke tests — a meta-skill that orchestrates the entire delivery pipeline.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-3.2-green.svg)](.)
[![Codex Compatible](https://img.shields.io/badge/codex-compatible-brightgreen.svg)](.)

## What Is Flow Weaver

Flow Weaver is an **orchestrator meta-skill** for AI coding agents. It turns a rough one-line idea into fully tested, shipped code by coordinating 10+ stages across 7 human confirmation gates, delegating each stage to specialized sub-skills, and persisting state throughout.

Think of it as a **CI/CD pipeline for the ideation-to-code journey** — but driven by AI agents with human oversight at every critical decision point.

## Quick Start

```
User: 启动 flow-weaver, 我想做一个团队周报自动汇总工具
```

That's it. The pipeline handles the rest: environment check → requirements → design → PRD → prototype → architecture → test cases → security review → code generation → smoke test → delivery.

Or for a faster path:
```
User: 启动 flow-weaver --quick
```

## Pipeline Overview

### Full Mode (Complete Pipeline)

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

### Quick Mode (Dynamic Trimming)

When `workflow.mode = quick`, stages are trimmed based on `project_type`:

| Project Type | Skipped Stages | Skipped Gates | Schema Changes |
|---|---|---|---|
| `frontend-only` | product_design, backend_arch, backend_code_generation, security_arch | gate2_design | test_case smoke_cases optional |
| `api-only` | prototype, frontend_arch, frontend_code_generation | — | prototype-schema skip, frontend-arch skip |
| `microservice` | product_design | gate2_design | prd depends on requirement (not product_design) |
| `monolith` | (none — full pipeline) | (none) | — |

User can also force quick mode: `"启动 flow-weaver --quick"`

### Gate Summary

| Gate | Stage | Validates | Checklist |
|------|-------|-----------|-----------|
| Gate 1 | Stage 1 | Requirement document quality | `checklists/gate1-requirement.md` |
| Gate 2 | Stage 2.① | Product design completeness | `checklists/gate2-design.md` |
| Gate 3 | Stage 2.② | PRD only (functional overview, module specs, data models, API contracts) | `checklists/gate3-prd.md` |
| Gate 4 | Stage 2 end | All Stage 2 artifacts batch-wise (prototype, frontend_arch, backend_arch, test_case) + cross-artifact consistency | `checklists/gate4-artifacts.md` |
| Gate 5 | Stage 2.7 | Architecture-level security design (auth, encryption, input validation) | `checklists/gate5-security-arch.md` |
| Gate 6 | Post-Stage 3 | Code-level static analysis (no Critical/High findings) | `checklists/gate6-security-code.md` |
| Final Gate | — | Code completeness, test pass rate, consistency, quality, security, delivery summary | `checklists/final-gate.md` |

## Execution Rules (12 Core Principles)

| Rule | Name | Key Point |
|------|------|-----------|
| 0 | Explicit trigger only | Never auto-invoke; must be explicitly requested |
| 1 | State persistence | `docs/workflow-state.yaml` is the single source of truth |
| 1.5 | Quick mode trimming | Apply `quick_mode_trim` after project type selection |
| 2 | Parallel execution + write-path isolation | Git-branch isolation for Group A; sub-agents can only write to assigned paths |
| 3 | Subagent isolation | Each stage runs in an independent subagent |
| 4 | Artifact contract | Every artifact includes YAML front-matter (type, version, dependencies, schema) |
| 5 | Sub-skill version locking + health check | Record version + run health check for every community skill |
| 6 | Smoke test execution format | Auto-selected per tech stack (curl/bash, Python, REST-Assured, Playwright) |
| 7 | Two-layer security review | Gate 5 (architecture) + Gate 6 (code-level static analysis) |
| 8 | Gate 3/4 separation | Gate 3 = PRD only; Gate 4 = all Stage 2 artifacts |
| 9 | Template instantiation | Instantiate project scaffolds before code generation |
| 10 | Manual edit detection | Detect user edits at each gate; prompt to preserve or regenerate |
| 11 | Execution log archival | Archive oldest 500 entries when `execution_log` exceeds 1000 |
| 12 | Resume checkpoint | Resume from last checkpoint with prerequisite validation |

## Sub-Skill Registry

| Artifact | Skill | Source | Type |
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

**Banned:** `api-and-interface-design` — explicitly excluded from this workflow.

## Tech Stack Support

| Category | Options |
|---|---|
| Backend | Java 17 + Spring Boot 3.x · Go 1.21+ + Gin · Node.js 18+ + NestJS · Python 3.11+ + FastAPI |
| Frontend | React + Next.js + TypeScript + Tailwind · Vue 3 + Nuxt 3 + TypeScript + Tailwind |
| Database | PostgreSQL · MySQL · SQLite · MongoDB |

## Templates (Project Scaffolds)

During Stage 3, Flow Weaver instantiates the appropriate template from `templates/`:

| Template | Language/Framework | Structure |
|---|---|---|
| `templates/backend-spring-boot/` | Java 17 + Spring Boot 3.x + Maven | controller/service/mapper/entity/dto |
| `templates/backend-go-gin/` | Go 1.21+ + Gin | handler/service/repository/model/dto |
| `templates/backend-nestjs/` | Node.js + NestJS + TypeScript | controller/service/module/dto/entity |
| `templates/backend-fastapi/` | Python + FastAPI | api/core/models/schemas/services |
| `templates/frontend-react-next/` | React + Next.js + TS + Tailwind | app/components/lib/styles |
| `templates/frontend-vue-nuxt/` | Vue 3 + Nuxt 3 + TS + Tailwind | app/components/composables/stores/layouts |
| `templates/openapi-default.yaml` | OpenAPI 3.x | Starter spec with health endpoint |

All templates include a buildable skeleton with health endpoint, standard layered directory structure, and stack-appropriate `.gitignore`.

## File Structure

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

## State Management

All pipeline state is persisted in `docs/workflow-state.yaml` (copied from `rules/workflow-state-template.yaml` at start). Key sections:

| Section | Purpose |
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

## Integration with AI Agents

Flow Weaver is designed as a **skill file** (`SKILL.md`) that can be integrated into any AI coding agent that supports the OpenAI Codex skill format. The skill format is portable and agent-agnostic.

### Codex CLI (OpenAI Codex Desktop)

**Supported: ✅ Fully compatible**

Codex CLI is the native home for skills. Flow Weaver integrates seamlessly.

**Installation:**

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
```
启动 flow-weaver, [your idea]
```

**Prerequisites:** Codex CLI must have the required sub-skills installed (brainstorming, ui-ux-pro-max, prd-development, etc.). See the [Sub-Skill Registry](#sub-skill-registry) above.

### Claude Code (Anthropic)

**Supported: ✅ Compatible**

Claude Code supports skill files in the same Codex-compatible format. The `SKILL.md` with YAML front-matter (`name`, `description`) is recognized by Claude Code's skill loading mechanism.

**Installation:**

```bash
# Clone or copy into Claude Code's skills directory
mkdir -p ~/.claude/skills
cp -r /path/to/flow-weaver-skill ~/.claude/skills/flow-weaver

# Or symlink
ln -s /path/to/flow-weaver-skill \
  ~/.claude/skills/flow-weaver
```

Alternatively, place the skill in your project directory — Claude Code auto-discovers `SKILL.md` files in the workspace and parent directories.

**Usage in Claude Code:**
```
> 启动 flow-weaver, 我想做一个任务调度系统
```

**Prerequisites:** Claude Code needs the same sub-skills (brainstorming, prd-development, etc.) installed. Community skills can be installed via Claude Code's skill registry or placed directly in the skills directory.

### OpenCode

**Supported: ✅ Compatible**

OpenCode uses the same skill format and auto-discovers `SKILL.md` files. Flow Weaver's built-in skills (backend_arch, project type selection, tech stack recommendation) work natively without external dependencies.

**Installation:**

```bash
# Place in project root (auto-discovered)
cp -r /path/to/flow-weaver-skill ./flow-weaver-skill

# Or in OpenCode's global skills directory
mkdir -p ~/.opencode/skills
ln -s /path/to/flow-weaver-skill \
  ~/.opencode/skills/flow-weaver
```

**Usage:**
```
> 启动 flow-weaver, 我想做一个电商后台管理系统
```

**Note:** OpenCode's built-in `security-and-hardening` skill is automatically available. Community sub-skills may need to be installed separately.

### Cursor / VS Code (via Codex extension)

**Supported: ✅ Via Codex extension**

If you use the Codex CLI extension in Cursor or VS Code, Flow Weaver works identically to Codex CLI — install the skill in `~/.codex/skills/` and it will be available through the Codex panel.

### Other Agents

Flow Weaver's core format is a **markdown file with YAML front-matter**, which is a widely adopted convention. Any agent that supports:

- YAML front-matter in markdown files
- External skill/tool definitions
- Sub-agent spawning for parallel execution

...can potentially integrate Flow Weaver. The main requirements are:

1. **Skill loading**: The agent must be able to read and parse `SKILL.md`
2. **Sub-agent support**: Flow Weaver delegates each stage to independent sub-agents
3. **File system access**: To read/write `docs/workflow-state.yaml` and generate artifacts
4. **Shell execution**: For environment checks, smoke tests, and git operations

## Example Invocation

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

## Advanced Features

### Parallel Execution with Git Branch Isolation

Group A (prototype + backend_arch) runs concurrently on separate git branches. After both complete, branches are merged. Conflicts are detected and stale artifacts are flagged.

### Change Propagation

When an upstream artifact is modified (e.g., user rejects PRD and requests changes), all downstream artifacts are automatically marked as `stale`. Affected artifacts are listed for the user to decide: regenerate all, regenerate selectively, or cancel the change.

### Retry & Rollback

Three-tier failure handling:
- **Level 1**: Single module test failure → auto-diagnose → fix → retry (max 3) → rollback only that module
- **Level 2**: Smoke test failure → auto-diagnose → fix → retry (max 3) → rollback to `pre-smoke-test` git tag
- **Level 3**: Still failing → diagnostic report + human intervention (user gives direction, manual fix, skip as known issue, or retreat to Gate 4)

### Resume from Interruption

If the workflow is interrupted (agent disconnect, session timeout, etc.), Flow Weaver remembers exactly where it left off. On next invocation, it offers to resume from the last unconfirmed gate with prerequisite validation.

### Manual Edit Detection

At each gate, Flow Weaver checks if the user manually edited any artifact since it was generated. If so, it prompts: "检测到手动修改，是否重新生成版本？"

### Two-Layer Security

- **Gate 5 (Architecture)**: Reviews auth design, encryption strategy, input validation approach in architecture documents
- **Gate 6 (Code)**: Runs tech-stack-specific static analyzers (SpotBugs, golangci-lint+gosec, Semgrep, Bandit) on generated code

### Progress Dashboard

Real-time HTML dashboard (`docs/progress.html`) with color-coded stage timeline, per-module progress bars, timing statistics, and artifact version tracking. Auto-refreshes via AJAX polling.

## Prerequisites

### Required Tools

| Tool | Minimum Version | Purpose |
|---|---|---|
| Node.js | 18+ | Frontend + smoke test |
| Git | 2.30+ | Rollback, branch isolation |
| JDK 17+ / Go 1.21+ / Python 3.11+ | — | Backend (depends on tech stack) |
| Playwright | Latest (optional) | Browser-based smoke tests |

### Required Sub-Skills

All community sub-skills must be installed in the agent's skills directory before starting the pipeline. See [Sub-Skill Registry](#sub-skill-registry) for sources.

### Optional Tools (Non-Blocking)

| Tool | Purpose |
|---|---|
| SonarQube Scanner | Static code analysis |
| Prettier / ESLint / Checkstyle / golangci-lint / flake8 | Code quality |
| SpotBugs / gosec / Semgrep / Bandit | Security scanning |

## Changelog

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

### v1.0 – v2.0
- Basic pipeline flow, multi tech stack, parallel optimization, progress visualization, code quality, smart recommendations, risk assessment, reliability & observability

## License

MIT
