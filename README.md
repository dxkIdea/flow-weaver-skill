# Flow Weaver Skill

A meta-skill that orchestrates a complete delivery pipeline from one-line idea to shipped code.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                       Flow Weaver Skill                          │
├─────────────────────────────────────────────────────────────────┤
│  SKILL.md (Main definition)                                     │
│  ├── 14 Rules (govern pipeline behavior)                       │
│  ├── 10 Stages (env-check → Stage 1-3 → delivery)              │
│  └── Gate confirmation points                                  │
├─────────────────────────────────────────────────────────────────┤
│  rules/ (Detailed rule implementations)                         │
│  ├── env-check.md              # Environment & skill pre-check  │
│  ├── change-propagation.md     # Stale artifact handling        │
│  ├── retry-policy.md           # Rollback & retry rules         │
│  ├── logging.md                # Execution log & audit rules    │
│  ├── progress-visualization.md # Progress dashboard rules       │
│  ├── progress-template.html    # HTML dashboard template        │
│  └── progress-template.json    # JSON data template             │
├─────────────────────────────────────────────────────────────────┤
│  schemas/ (Per-artifact validation schemas)                     │
│  ├── requirement-schema.yaml   # Requirement document           │
│  ├── product-design-schema.yaml# Product design                 │
│  ├── prd-schema.yaml           # PRD                            │
│  ├── prototype-schema.yaml     # UI prototype                   │
│  ├── frontend-arch-schema.yaml # Frontend architecture           │
│  ├── backend-arch-schema.yaml  # Backend architecture            │
│  └── test-case-schema.yaml     # Test cases                     │
├─────────────────────────────────────────────────────────────────┤
│  checklists/ (Gate validation checklists)                       │
│  ├── gate4-artifacts.md        # Stage 2 artifact validation   │
│  └── final-gate.md             # Final delivery validation     │
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
| `/rules/` | timeout-retry-config.yaml | Per-stage timeout thresholds and fallback config |
| `/rules/` | logging.md | Execution log format, audit requirements |
| `/rules/` | progress-visualization.md | Progress dashboard generation rules |
| `/rules/` | progress-template.html | Real-time progress dashboard template |
| `/rules/` | progress-template.json | Dashboard data template |
| `/schemas/` | 7 *.yaml | Per-artifact validation schemas |
| `/checklists/` | gate4-artifacts.md | Stage 2 artifact validation checklist |
| `/checklists/` | final-gate.md | Final delivery validation checklist |

## Feature Matrix

| Feature | Status | Description |
|---------|--------|-------------|
| **Open-Source Sub-skills** | ✅ | All stages delegate to real open-source skills |
| **Multi Tech Stack** | ✅ | Java/Go/Node/Python backend, React/Vue frontend |
| **Parallel Execution** | ✅ | Stage 2 Group A/B/C parallel groups via dispatching-parallel-agents |
| **Progress Visualization** | ✅ | Real-time dashboard with module-level progress |
| **Project Type Templates** | ✅ | Microservice/Monolith/Frontend-only/API-only |
| **Security Review** | ✅ | Stage 2.7 using security-and-hardening skill |
| **Cross-Review** | ✅ | Stage 3 pre-delivery using doubt-driven-development |
| **Smart Tech Stack Recommendation** | ✅ | AI-powered recommendation based on requirements |
| **Cost & Risk Assessment** | ✅ | Stage 1.6 development cost and risk evaluation |

## Pipeline Stages

```
[env-check] → Stage 1 (Brainstorming) → Gate 1
    → Stage 1.3 (Project Type) → Stage 1.5 (Tech Stack)
    → Stage 1.6 (Assessment) → Stage 2 (Artifacts)
        → ① product_design (ui-ux-pro-max) → Gate 2
        → ② prd (prd-development) → Gate 3
        → Group A: prototype (design-taste-frontend) + backend_arch (built-in)
        → Group B: frontend_arch (frontend-design)
        → Group C: test_case (test-driven-development) → Gate 4
        → 2.7 Security Review (security-and-hardening) → Gate 5
    → Stage 3 (Code + Smoke Test + Cross-Review) → [Final Gate] → Deliver
```

## Change Log

### v3.0 - Open-Source Refactoring
- Renamed to Flow Weaver (was: vibe-coding-workflow)
- All sub-skills replaced with real open-source skills (obra/superpowers, anthropics, nextlevelbuilder, pm-skills, Leonxlnx)
- Added Stage 2.7 Security Review (security-and-hardening)
- Added cross-review before final gate (doubt-driven-development)
- Removed incremental build support; replaced with sequential re-generation
- Simplified schemas, checklists, logging, and change-propagation rules

### v1.0 - Initial Release
- Basic pipeline flow (env-check → Stage 1-3 → delivery)
- Single tech stack (Java Spring Boot + React)
- Core rules for stale propagation and rollback

### v1.1 - Sub-skill Enhancement
- Structured skill configuration (skill_id, fallback, alternatives, type)
- Environment pre-check with tech-stack-specific tools
- Skill availability detection and fallback mechanisms

### v1.2 - Multi Tech Stack
- Support for Java/Go/Node/Python backend
- Support for React/Vue frontend
- Dynamic skill routing based on tech stack selection

### v1.3 - Parallel Optimization
- Stage 2 parallel groups (A: prototype+backend, B: frontend, C: tests)
- Parallel group state tracking
- Concurrent execution support

### v1.4 - Progress Visualization
- Real-time HTML dashboard
- Progress percentage calculation
- Stage timeline with color coding

### v1.5 - Incremental Build
- Change detection and scope calculation
- Cache management with content hashes
- Partial regeneration support

### v1.6 - Project Type Templates
- Microservice/Monolith/Frontend-only/API-only project types
- Stage adjustment based on project type
- Customized artifact generation per type

### v1.7 - Code Quality
- Tech-stack-specific quality tools (Checkstyle, ESLint, golangci-lint)
- Final gate code quality validation
- Environment tool pre-check for quality tools

### v1.8 - Smart Recommendations
- AI-powered tech stack recommendation
- Feature-based recommendation rules
- Context-aware suggestion engine

### v1.9 - Risk Assessment
- Development cost estimation
- Multi-dimensional risk assessment
- Stage 1.6 cost & risk evaluation

### v2.0 - Reliability & Observability
- Per-stage timeout configuration
- Configurable retry policies with exponential backoff
- Detailed execution logging and audit trail
- Module-level progress tracking
- Execution timing statistics

## Usage

1. **Start workflow**: Initialize from `rules/workflow-state-template.yaml`
2. **Env check**: Run pre-checks for tools and sub-skills
3. **Stage 1**: Brainstorm requirements with user (skill: brainstorming)
4. **Stage 1.3**: Select project type (built-in)
5. **Stage 1.5**: Select tech stack with AI recommendation (built-in)
6. **Stage 1.6**: Evaluate cost and risks (built-in)
7. **Stage 2**: Generate artifacts (parallel groups via dispatching-parallel-agents)
8. **Stage 2.7**: Security review (skill: security-and-hardening)
9. **Stage 3**: Generate code with TDD + cross-review (skills: executing-plans, test-driven-development, doubt-driven-development)
10. **Final gate**: Validate and deliver

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
| security-and-hardening | opencode built-in | Stage 2.⑦ | Security review |
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
- Git 2.30+ (rollback)
- Playwright (smoke test browser automation)