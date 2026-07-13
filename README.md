# Flow Weaver Skill

A meta-skill that orchestrates a complete delivery pipeline from one-line idea to shipped code.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                       Flow Weaver Skill                          │
├─────────────────────────────────────────────────────────────────┤
│  SKILL.md (Main definition)                                     │
│  ├── 14+ Rules (govern pipeline behavior)                      │
│  ├── 10+ Stages (env-check → Stage 1-3 → delivery)             │
│  └── 7 Gate confirmation points                                │
├─────────────────────────────────────────────────────────────────┤
│  rules/ (Detailed rule implementations)                         │
│  ├── env-check.md              # Environment & skill pre-check  │
│  ├── change-propagation.md     # Stale artifact handling        │
│  ├── retry-policy.md           # Rollback & retry rules         │
│  ├── logging.md                # Execution log & audit rules    │
│  ├── progress-visualization.md # Progress dashboard rules       │
│  ├── timeout-retry-config.yaml # Per-stage timeout thresholds   │
│  ├── progress-template.html    # HTML dashboard template        │
│  └── progress-template.json    # JSON data template             │
├─────────────────────────────────────────────────────────────────┤
│  schemas/ (Per-artifact validation schemas)                     │
│  ├── requirement-schema.yaml   # Requirement document           │
│  ├── product-design-schema.yaml# Product design                 │
│  ├── prd-schema.yaml           # PRD                            │
│  ├── prototype-schema.yaml     # UI prototype                   │
│  ├── frontend-arch-schema.yaml # Frontend architecture          │
│  ├── backend-arch-schema.yaml  # Backend architecture           │
│  └── test-case-schema.yaml     # Test cases (+ execution format)│
├─────────────────────────────────────────────────────────────────┤
│  checklists/ (Gate validation checklists)                       │
│  ├── gate1-requirement.md      # Requirement sign-off          │
│  ├── gate2-design.md           # Product design sign-off       │
│  ├── gate3-prd.md              # PRD sign-off (PRD only)       │
│  ├── gate4-artifacts.md        # Stage 2 artifacts batch       │
│  ├── gate5-security-arch.md    # Architecture security review  │
│  ├── gate6-security-code.md    # Code-level security scan      │
│  └── final-gate.md             # Final delivery validation     │
├─────────────────────────────────────────────────────────────────┤
│  templates/ (Project scaffolds per tech stack)                  │
│  ├── backend-spring-boot/      # Java 17 + Spring Boot 3.x     │
│  ├── backend-go-gin/           # Go 1.21+ + Gin                │
│  ├── backend-nestjs/           # Node.js + NestJS              │
│  ├── backend-fastapi/          # Python + FastAPI              │
│  ├── frontend-react-next/      # React + Next.js               │
│  ├── frontend-vue-nuxt/        # Vue 3 + Nuxt 3                │
│  └── openapi-default.yaml      # Empty OpenAPI 3.x spec        │
├─────────────────────────────────────────────────────────────────┤
│  rules/workflow-state-template.yaml                             │
│  └── Single source of truth for pipeline state                  │
└─────────────────────────────────────────────────────────────────┘
```

## File Structure

| Directory | Files | Purpose |
|-----------|-------|---------|
| `/` | SKILL.md | Main skill definition, pipeline rules, stage details |
| `/` | README.md | Architecture overview, file list, changelog |
| `/rules/` | env-check.md | Environment and sub-skill pre-check rules |
| `/rules/` | change-propagation.md | Stale artifact propagation and dependency rules |
| `/rules/` | retry-policy.md | Rollback, retry, and error handling rules |
| `/rules/` | logging.md | Execution log format, audit requirements |
| `/rules/` | progress-visualization.md | Progress dashboard generation rules |
| `/rules/` | timeout-retry-config.yaml | Per-stage timeout thresholds and fallback config |
| `/rules/` | progress-template.html | Real-time progress dashboard template |
| `/rules/` | progress-template.json | Dashboard data template |
| `/schemas/` | 7 *.yaml | Per-artifact validation schemas |
| `/checklists/` | 7 *.md | Gate validation checklists |
| `/templates/` | 8 dirs/files | Project scaffolds per tech stack |
| `/rules/workflow-state-template.yaml` | State template | Single source of truth |

## Features

| Feature | Description |
|---------|-------------|
| **Two Modes** | Full (all stages) or Quick (trimmed by project type) |
| **Open-Source Sub-skills** | All stages delegate to real open-source skills |
| **Multi Tech Stack** | Java/Go/Node/Python backend, React/Vue frontend |
| **Parallel Execution** | Stage 2 parallel groups with git-branch isolation |
| **Version Locking** | Sub-skill versions recorded and health-checked |
| **Write-Path Isolation** | Sub-agents restricted to assigned directories |
| **Two-Layer Security** | Architecture review (Gate 5) + Code scanning (Gate 6) |
| **Clear Gate Boundaries** | Gate 3 = PRD only; Gate 4 = all Stage 2 artifacts |
| **Smoke Test Formats** | Auto-selected per tech stack (curl/bash/Python/REST-Assured) |
| **Project Templates** | Buildable skeletons for each tech stack combination |
| **Progress Visualization** | Real-time HTML dashboard with stage timeline |
| **Retry & Rollback** | 3-level failure handling with configurable timeouts |
| **Audit Trail** | Execution log, iteration history, fallback tracking |

## Pipeline Flow

```
env-check → Stage 1 → Gate 1 → Stage 1.3 → Stage 1.5 → [Stage 1.6]
  → Stage 2.① → Gate 2 → Stage 2.② → Gate 3
  → Stage 2 Group A → Stage 2 Group B → Stage 2 Group C → Gate 4
  → Stage 2.7 Security (Arch) → Gate 5
  → Stage 3 Code Gen + Smoke Test
  → Security (Code) → Gate 6
  → Cross-Review → Final Gate → Deliver
```

`[Stage 1.6]` = cost & risk assessment (skippable)

## Quick Mode

Automatically trims stages based on project type:

| Type | Skips |
|------|-------|
| `frontend-only` | product_design, backend_arch, backend_code, security_arch |
| `api-only` | prototype, frontend_arch, frontend_code |
| `microservice` | product_design |
| `monolith` | (nothing — full pipeline) |

## Usage

1. **Start workflow**: `启动 flow-weaver` or `启动 flow-weaver --quick`
2. **Initialize**: Copies `workflow-state-template.yaml` → `docs/workflow-state.yaml`
3. **Env check**: Run pre-checks for tools and sub-skills (with health checks)
4. **Stage 1**: Brainstorm requirements with user (skill: brainstorming)
5. **Stage 1.3**: Select project type (built-in) → apply quick mode trimming
6. **Stage 1.5**: Select tech stack with AI recommendation (built-in)
7. **Stage 1.6**: Evaluate cost and risks (built-in, skippable)
8. **Stage 2**: Generate artifacts (parallel groups with write-path isolation)
9. **Stage 2.7**: Architecture security review
10. **Stage 3**: Instantiate template → generate code with TDD → smoke test
11. **Post-stage3**: Code-level security scan
12. **Final gate**: Validate and deliver

## Dependencies

### Open-Source Sub-Skills (required)

| Skill | Source | Stage | Purpose |
|-------|--------|-------|---------|
| brainstorming | [obra/superpowers](https://github.com/obra/superpowers) | Stage 1 | Requirement exploration & convergence |
| ui-ux-pro-max | [nextlevelbuilder](https://github.com/nextlevelbuilder/skills) ⭐24.7k | Stage 2.① | Product design |
| prd-development | [pm-skills](https://github.com/dingxingkai/pm-skills) | Stage 2.② | PRD generation |
| design-taste-frontend | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | Stage 2.③ | UI prototype |
| frontend-design | [anthropics/skills](https://github.com/anthropics/skills) ⭐160k | Stage 2.④ | Frontend architecture |
| test-driven-development | [obra/superpowers](https://github.com/obra/superpowers) | Stage 2.⑥, Stage 3 | Test cases, TDD code gen |
| security-and-hardening | opencode built-in | Stage 2.⑦ | Architecture security review |
| executing-plans | [obra/superpowers](https://github.com/obra/superpowers) | Stage 3 | Code generation execution |
| doubt-driven-development | [obra/superpowers](https://github.com/obra/superpowers) | Stage 3 | Cross-review |
| dispatching-parallel-agents | [obra/superpowers](https://github.com/obra/superpowers) | Stage 2 Group A | Parallel dispatch |

### Built-in (no external skill)
- Project Type Selection (Stage 1.3)
- Tech Stack Recommendation (Stage 1.5)
- Cost & Risk Assessment (Stage 1.6)
- Backend Architecture (Stage 2.⑤)

### Banned Skills
- `api-and-interface-design` — explicitly excluded from this workflow

### Environment Tools
- Node.js 18+ (frontend + smoke test)
- JDK 17+ / Go 1.21+ / Python 3.11+ (depending on tech stack)
- Git 2.30+ (rollback, branch isolation)
- Playwright (optional, for browser-based smoke tests)

## Change Log

### v3.1 - Improvement Patch (2026-07-13)
- **Quick mode**: Dynamic stage trimming by project type (frontend-only, api-only, microservice, monolith)
- **Sub-skill version locking**: Each skill records version + health check result
- **Write-path isolation**: Sub-agents restricted to assigned directories; git-branch isolation for parallel Group A
- **Smoke test execution format**: Auto-selected per tech stack (curl/bash, Python, REST-Assured, Playwright)
- **Gate 3/4 separation**: Gate 3 validates PRD only; Gate 4 validates all Stage 2 artifacts
- **Two-layer security review**: Gate 5 (architecture design) + Gate 6 (code-level static analysis)
- **Project templates**: Buildable skeletons for 4 backends + 2 frontends + OpenAPI default
- Added `gate5-security-arch.md` and `gate6-security-code.md` checklists
- Added `templates/` directory with starter projects

### v3.0 - Open-Source Refactoring
- Renamed to Flow Weaver (was: vibe-coding-workflow)
- All sub-skills replaced with real open-source skills
- Added Stage 2.7 Security Review (security-and-hardening)
- Added cross-review before final gate (doubt-driven-development)
- Removed incremental build support; replaced with sequential re-generation

### v1.0 - Initial Release
- Basic pipeline flow (env-check → Stage 1-3 → delivery)
- Single tech stack (Java Spring Boot + React)

### v1.1 - Sub-skill Enhancement
- Structured skill configuration (skill_id, fallback, alternatives, type)
- Environment pre-check with tech-stack-specific tools

### v1.2 - Multi Tech Stack
- Support for Java/Go/Node/Python backend
- Support for React/Vue frontend

### v1.3 - Parallel Optimization
- Stage 2 parallel groups (A: prototype+backend, B: frontend, C: tests)

### v1.4 - Progress Visualization
- Real-time HTML dashboard

### v1.5 - Incremental Build
- Change detection and scope calculation

### v1.6 - Project Type Templates
- Microservice/Monolith/Frontend-only/API-only project types

### v1.7 - Code Quality
- Tech-stack-specific quality tools

### v1.8 - Smart Recommendations
- AI-powered tech stack recommendation

### v1.9 - Risk Assessment
- Development cost estimation and multi-dimensional risk assessment

### v2.0 - Reliability & Observability
- Per-stage timeout configuration
- Configurable retry policies with exponential backoff
- Detailed execution logging and audit trail
