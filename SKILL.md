---
name: "flow-weaver"
description: "Flow-weaver orchestrates a full delivery pipeline from a one-line idea to shipped code with smoke tests. Supports two modes: 'full' (all 10+ stages) and 'quick' (dynamically trimmed by project type). Invoke when user says '启动 flow-weaver' / '启动流水线' / 'flow-weaver' or asks to turn a rough idea into a complete delivery. Orchestrates open-source sub-skills across stages with human gates."
---

# Flow Weaver (Meta Skill)

## Description

Flow Weaver orchestrates a complete delivery pipeline: from a one-line idea to shipped code that passes automated smoke tests. It coordinates stages, delegates each to a designated open-source sub-skill, persists state across stages, and pauses for human confirmation at explicit gates.

**Trigger conditions (explicit trigger only — do NOT auto-invoke):**
- User explicitly says "启动 flow-weaver", "启动流水线", or "flow-weaver"
- User asks to turn a rough idea into a full delivery covering design, PRD, prototype, architecture, test cases, code, and smoke tests
- User references this skill by name

**Do NOT invoke when:**
- User only wants a single artifact (e.g., just a PRD, just a prototype)
- User wants to directly edit code without going through the full pipeline
- User is doing a quick prototype or throwaway experiment

## When to Use

- One-line idea → complete deliverable pipeline
- User explicitly requests the full workflow
- New feature/product greenfield development where requirements are still rough

## Pipeline Overview

### Full Mode (default)

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
Stage 3: Code generation + smoke test (skill: executing-plans + subagent: TDD)
  Per-module generation → unit test per module → smoke test
  Failure handling: auto-diagnose → fix → retry (max 3) → dialog intervention
  ↓
  Post-generation: Security Review - Code Layer (static analysis) → GATE 6
  ↓ GATE 7: Final delivery sign-off
[Final gate] → Deliver
```

### Quick Mode (fast path)

When `workflow.mode = quick`, stages are trimmed based on `project_type`:

| Project Type | Skipped Stages | Skipped Gates |
|---|---|---|
| `frontend-only` | product_design, backend_arch, backend_code_generation, security_arch | gate2_design |
| `api-only` | prototype, frontend_arch, frontend_code_generation | — |
| `microservice` | product_design | gate2_design |
| `monolith` | (none — full pipeline) | (none) |

User can also force quick mode: "启动 flow-weaver --quick"

## Execution Rules

### Rule 0: Explicit trigger only
This skill must be explicitly triggered. Never auto-invoke based on vague phrasing. If unsure, ask the user to confirm whether they want to start the full workflow.

### Rule 1: State persistence is mandatory
Before starting any stage, read `docs/workflow-state.yaml`. After completing any stage, update it. This is the single source of truth for progress, versions, and confirmations. If the file doesn't exist, initialize it from the template in `rules/workflow-state-template.yaml`.

**Resume logic:** On invocation, first check `docs/workflow-state.yaml`. If a previous run exists, offer to resume from the last unconfirmed gate rather than starting over.

### Rule 1.5: Quick mode trimming
After Stage 1.3 (project type selection), apply trimming rules from `workflow-state-template.yaml` to determine which stages/gates to skip. Update `workflow.mode` accordingly.

### Rule 2: Parallel group execution with write-path isolation
Stage 2 executes in parallel groups based on dependencies:
- **Group A**: prototype + backend_arch (both depend only on PRD, can run concurrently)
- **Group B**: frontend_arch (depends on prototype)
- **Group C**: test_case (depends on backend_arch)

Within each group, tasks run sequentially. Group B starts only after prototype completes. Group C starts only after backend_arch completes. Use `dispatching-parallel-agents` to dispatch Group A's independent tasks.

**Write-path isolation:** Each artifact has a designated `write_paths` entry in the state file. Sub-agents MUST NOT write outside their assigned directory. Group A tasks are created on separate git branches (`fw-prototype-<uuid>`, `fw-backend-<uuid>`) and merged after completion. Conflicts are resolved by marking stale artifacts.

See `rules/workflow-state-template.yaml` for parallel_group and write_paths definitions.

### Rule 3: Subagent isolation per stage
Each stage (1, 2.①–2.⑦, 3) must be executed in an independent subagent to protect main-context window. The main session only retains:
- `docs/workflow-state.yaml` content
- The current stage's input/output artifact paths
- Gate confirmation results

Subagents must NOT carry over context from unrelated stages.

### Rule 4: Artifact contract enforcement
Every produced artifact must include a YAML front-matter block declaring its type, version, dependencies, and schema:

```yaml
---
artifact: <type>
version: v1
depends_on:
  - artifact: requirement
    version: v1
    status: confirmed
schema: requirement-schema.yaml
generated_by: brainstorming
generated_at: <ISO-8601>
---
```

### Rule 5: Sub-skill version locking and health check
Each sub-skill has a `version` and `health_check_passed` field in the state file. During env-check:
1. Record the installed version/commit hash of each community skill
2. Run a smoke test against each skill (generate a tiny dummy artifact and validate its structure)
3. If a skill fails health check, try `alternatives` list first, then fallback to `built-in-chat`
4. Record all results in `skill_check.health_checks`

Never use a community skill without a recorded version.

### Rule 6: Smoke test execution format
Smoke cases use the `execution_format` defined in `test-case-schema.yaml`. The format is auto-selected based on `tech_stack.backend`:
- `java-spring-boot` → `rest_assured`
- `go-gin` → `http_bash`
- `node-nestjs` → `http_bash` (or `playwright` for full e2e)
- `python-fastapi` → `http_python`

Before running smoke tests, ensure the backend is running on the expected port.

### Rule 7: Two-layer security review
Security review is split into two gates:
- **Gate 5 (Stage 2.7)**: Architecture-level security design review. Checks authentication, authorization, encryption, input validation strategies in backend_arch and frontend_arch.
- **Gate 6 (post-stage3)**: Code-level static analysis. Runs tech-stack-specific scanners (SpotBugs, golangci-lint+gosec, Semgrep, Bandit). No Critical/High findings allowed.

See `checklists/gate5-security-arch.md` and `checklists/gate6-security-code.md`.

### Rule 8: Separation of Gate 3 and Gate 4
- **Gate 3** validates ONLY the PRD document (functional overview, module specs, data models, API contracts, business rules, non-functional requirements).
- **Gate 4** validates ALL Stage 2 artifacts batch-wise (prototype, frontend_arch, backend_arch, test_case) plus cross-artifact consistency.

PRD is NOT re-checked at Gate 4.

## Sub-Skill Registry

| Artifact | Skill | Source | Type |
|---|---|---|---|
| requirement | brainstorming | obra/superpowers | community |
| product_design | ui-ux-pro-max | nextlevelbuilder/skills | community |
| prd | prd-development | pm-skills | community |
| prototype | design-taste-frontend | Leonxlnx/taste-skill | community |
| frontend_arch | frontend-design | anthropics/skills | community |
| backend_arch | (built-in) | — | built-in |
| test_case | test-driven-development | obra/superpowers | community |
| security_arch | security-and-hardening | opencode built-in | community |
| code_generation | executing-plans | obra/superpowers | community |
| cross_review | doubt-driven-development | obra/superpowers | community |
| parallel_dispatch | dispatching-parallel-agents | obra/superpowers | community |

**Tech Stack Configuration:**

```yaml
tech_stack:
  backend: java-spring-boot              # java-spring-boot / go-gin / node-nestjs / python-fastapi
  frontend: react-next                   # react-next / vue-nuxt
  database: postgresql                   # postgresql / mysql / sqlite / mongodb
```

## Templates (Project Scaffolds)

During Stage 3, the orchestrator instantiates the appropriate template from `templates/` based on the selected tech stack:

| Template | Purpose |
|---|---|
| `templates/backend-spring-boot/` | Java 17 + Spring Boot 3.x + Maven layered structure |
| `templates/backend-go-gin/` | Go 1.21+ + Gin layered structure |
| `templates/backend-nestjs/` | Node.js 18+ + NestJS + TypeScript |
| `templates/backend-fastapi/` | Python 3.11+ + FastAPI |
| `templates/frontend-react-next/` | React + Next.js + TypeScript + Tailwind |
| `templates/frontend-vue-nuxt/` | Vue 3 + Nuxt 3 + TypeScript + Tailwind |
| `templates/openapi-default.yaml` | Empty OpenAPI 3.x starter spec |

Templates provide a buildable skeleton with health endpoint, standard directory structure, and stack-appropriate `.gitignore`.

## Example Invocation

```
User: 启动 flow-weaver, 我想做一个团队周报自动汇总工具

AI: [env-check] 检测环境...
    ✓ JDK 17.0.8
    ✓ Maven 3.9.5
    ✓ Node.js 18.18.0
    ✓ Playwright 1.42.0
    ✓ Git 2.42.0
    [skill-check] 检测子技能...
    ✓ brainstorming (obra/superpowers, v1.2.0, health OK)
    ✓ ui-ux-pro-max (nextlevelbuilder, v3.1.0, health OK)
    ✓ prd-development (pm-skills, abc1234, health OK)
    ✓ design-taste-frontend (Leonxlnx, v2.0.0, health OK)
    ✓ frontend-design (anthropics, v1.0.0, health OK)
    ✓ test-driven-development (obra/superpowers, v4.0.0, health OK)
    ✓ security-and-hardening (built-in, health OK)
    ✓ executing-plans (obra/superpowers, v2.1.0, health OK)
    ✓ doubt-driven-development (obra/superpowers, v3.0.0, health OK)
    
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
    ...
```

## Important Notes

- This is an orchestrator, not an executor. Each stage delegates to its sub-skill via subagent.
- State (`docs/workflow-state.yaml`) is mandatory and authoritative. No state = no resume.
- Gates are hard stops. Never proceed without explicit confirmation.
- If a sub-skill is unavailable, try alternatives first, then fallback (Rule 5).
- Versioning: artifacts use v1, v2, ... Never overwrite — always increment for iterations.
- Tech stack is selected in Stage 1.5 (built-in); backend_arch is also built-in (no external skill).
- Stage 2 uses `dispatching-parallel-agents` for Group A parallel execution with git-branch isolation.
- `api-and-interface-design` skill is explicitly banned from this workflow.
- Progress is visible in `docs/progress.html` and `docs/progress.json`.
- Project templates in `templates/` provide buildable skeletons for each tech stack.
