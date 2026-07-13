---
name: "flow-weaver"
description: "Flow-weaver orchestrates a full delivery pipeline from a one-line idea to shipped code with smoke tests. Invoke when user says '启动 flow-weaver' / '启动流水线' / 'flow-weaver' or asks to turn a rough idea into a complete delivery. Orchestrates open-source sub-skills (brainstorming, ui-ux-pro-max, prd-development, design-taste-frontend, frontend-design, test-driven-development) across 10 stages with 5 human gates."
---

# Flow Weaver (Meta Skill)

## Description

Flow Weaver orchestrates a complete delivery pipeline: from a one-line idea to shipped code that passes automated smoke tests. It coordinates 10 stages, delegates each stage to a designated open-source sub-skill, persists state across stages, and pauses for human confirmation at 5 explicit gates.

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

```
[env-check] Environment + Skills ready?
  ↓
Stage 1: Requirement brainstorming (skill: brainstorming)
  ↓ GATE 1: Requirement sign-off
Stage 1.3: Project Type Selection
  ↓
Stage 1.5: Tech Stack Selection
  ↓
Stage 1.6: Cost & Risk Assessment
  ↓
Stage 2: Artifact generation (parallel groups)
  ① product_design (skill: ui-ux-pro-max) → GATE 2: Design sign-off
  ② prd (skill: prd-development)          → GATE 3: PRD sign-off
  ┌─ ③ prototype (skill: design-taste-frontend) ┐
  └─ ⑤ backend_arch (built-in) ─────────────────┘→ Group A (parallel)
                                               ↓
  ④ frontend_arch (skill: frontend-design)    → Group B (depends on prototype)
                                               ↓
  ⑥ test_case (skill: test-driven-development)→ Group C (depends on backend_arch)
                                               ↓ GATE 4: Artifacts sign-off
Stage 2.7: Security Review (skill: security-and-hardening)
  ↓
Stage 3: Code generation + smoke test (skill: executing-plans + subagent: TDD)
  Per-module generation → unit test per module → smoke test
  Failure handling: auto-diagnose → fix → retry (max 3) → dialog intervention
  ↓ GATE 5: Cross-review (skill: doubt-driven-development)
[Final gate] → Deliver
```

## Execution Rules

### Rule 0: Explicit trigger only
This skill must be explicitly triggered. Never auto-invoke based on vague phrasing. If unsure, ask the user to confirm whether they want to start the full workflow.

### Rule 1: State persistence is mandatory
Before starting any stage, read `docs/workflow-state.yaml`. After completing any stage, update it. This is the single source of truth for progress, versions, and confirmations. If the file doesn't exist, initialize it from the template in `rules/workflow-state-template.yaml`.

**Resume logic:** On invocation, first check `docs/workflow-state.yaml`. If a previous run exists, offer to resume from the last unconfirmed gate rather than starting over.

### Rule 2: Parallel group execution
Stage 2 executes in parallel groups based on dependencies:
- **Group A**: prototype + backend_arch (both depend only on PRD, can run concurrently via `dispatching-parallel-agents`)
- **Group B**: frontend_arch (depends on prototype)
- **Group C**: test_case (depends on backend_arch)

Within each group, tasks run sequentially. Group B starts only after prototype completes. Group C starts only after backend_arch completes. Use `dispatching-parallel-agents` to dispatch Group A's independent tasks without shared state. See `rules/workflow-state-template.yaml` for parallel_group definitions.

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
artifact: <requirement|product_design|prd|prototype|frontend_arch|backend_arch|test_case|security_review|cross_review|code>
version: v1
depends_on:
  - <path/to/upstream/artifact.md>
schema: docs/schemas/<artifact>-schema.yaml
status: <draft|confirmed|stale>
generated_by: <sub-skill-name>
generated_at: <ISO-8601 timestamp>
---
```

Before a stage starts, validate that its declared `depends_on` artifacts exist and have `status: confirmed`. If any dependency is `stale` or missing, halt and ask the user.

### Rule 5: Change propagation
When an upstream artifact is modified after its downstream has been generated:
1. Mark all downstream artifacts as `status: stale` in `docs/workflow-state.yaml`
2. Notify the user which artifacts are stale
3. Ask whether to re-generate the stale chain or keep the old versions
4. If re-generating, follow normal stage order and gate rules

### Rule 6: Five human gates
These are mandatory stop points. Use `AskUserQuestion` at each gate:

| Gate | Location | What user confirms | On reject |
|---|---|---|---|
| Gate 1 | After Stage 1 | Requirement brainstorming doc | Re-clarify → re-generate requirement |
| Gate 2 | After 2.① | Product design proposal | Revise → re-generate design |
| Gate 3 | After 2.② | PRD | Revise → re-generate PRD |
| Gate 4 | After 2.⑥ | UI prototype + frontend arch + backend arch + test cases (batched) | Revise → re-generate specific items |
| Gate 5 | After Stage 2.7 | Security review findings + mitigations | Address findings → re-review |
| Final gate | After Stage 3 smoke test + cross-review pass | Deliver code | Re-iterate on failing areas |

**Never proceed past a gate without explicit user confirmation.** If user is unavailable, halt and wait.

### Rule 7: Environment + Skill pre-check
Before Stage 1, run env-check (see `rules/env-check.md`) consisting of two parts:

**Part 1: Environment tools check**
- JDK 17+ (for Java backend)
- Maven or Gradle
- Node.js 18+ (for frontend + smoke test)
- Playwright (for smoke test)
- Git (for commit-based rollback)

**Part 2: Sub-skill availability check**
- Verify all configured sub-skills can be loaded (brainstorming, ui-ux-pro-max, prd-development, design-taste-frontend, frontend-design, test-driven-development, security-and-hardening, doubt-driven-development, dispatching-parallel-agents, executing-plans)
- For unavailable skills, check alternatives list
- Record results in `docs/workflow-state.yaml` under `skill_check`

If any tools are missing, list them and ask the user to install. If skills are unavailable but have fallback, warn the user and proceed. If skills are unavailable with no fallback, halt and ask user to install.

### Rule 8: Automated quality validation before each gate
Before presenting a gate for human review, automatically validate the artifact against its schema (`checklists/<gate>.md`):
- Requirement doc must pass brainstorming skill's internal quality check
- PRD must contain all required sections (per prd-development schema)
- Prototype HTML must be openable in browser
- Test cases must have explicit assertions (per test-driven-development conventions)
- Security review must have no critical findings

If auto-validation fails, fix and retry before surfacing to user. Users should only review artifacts that already pass mechanical checks.

### Rule 9: Failure recovery in Stage 3
Code generation is per-module (module = one backend domain module from backend_arch). For each module:
1. Generate code
2. Run unit tests for that module
3. If pass → `git commit` → next module
4. If fail → auto-diagnose → fix → retry (max 3 attempts per module)
5. If still fails after 3 attempts → pause, output diagnostic report, switch to dialog mode (describe problem + attempted fixes, ask user for direction)

**Granular rollback policy:**
- Module unit test failure: roll back only that module's code (to last commit), keep other modules
- Smoke test failure: roll back to pre-smoke-test code state, do NOT discard generated artifacts
- Never roll back across gate boundaries without explicit user permission

### Rule 10: Commit-based rollback
- After each artifact generation in Stage 2: `git commit -m "chore(artifact): add <artifact-type> v<n>"`
- After each successful module in Stage 3: `git commit -m "feat(<module>): generate code + pass unit test"`
- Before smoke test: tag `pre-smoke-test`
- Rollback targets the last commit of the appropriate scope (see Rule 9)

### Rule 11: Skill fallback with alternatives
If a designated sub-skill fails to load or produces output failing auto-validation:
1. Retry once with the primary sub-skill
2. If still failing, try each alternative skill in order (from `skills.<stage>.alternatives`)
3. If all alternatives fail, fall back to the configured `fallback` (typically `built-in-chat`)
4. Log the fallback in `docs/workflow-state.yaml` under `fallbacks:` with:
   - stage name
   - original skill id
   - reason for failure
   - fallback_to skill
5. Continue pipeline; flag the artifact for extra human review at its gate

**Skill configuration in `docs/workflow-state.yaml`:**
```yaml
skills:
  backend_arch:
    skill_id: built-in
    type: built-in
    fallback: built-in-chat
    alternatives: []
    available: true
  frontend_arch:
    skill_id: frontend-design
    type: community
    source: https://github.com/anthropics/skills
    fallback: built-in-chat
    alternatives: []
    available: true
```

### Rule 12: Progress visualization
Generate/update `progress.html` and `progress.json` after each stage completion and gate confirmation. See `rules/progress-visualization.md` for detailed rules, progress calculation formula, and update timing.

### Rule 13: Cross-review before final gate
Before presenting the Final gate, run a cross-review using `doubt-driven-development`:
1. Invoke `doubt-driven-development` to perform an adversarial review of the generated code
2. Focus on: correctness, security, edge cases, and architectural consistency
3. Log findings in `docs/07-review/cross-review-v1.md`
4. If critical issues found, pause for dialog before delivering

### Rule 14: Timeout & Retry Mechanism
Each Stage has configurable timeout thresholds and retry strategies. See `rules/timeout-retry-config.yaml` for per-stage timeout configuration and `rules/retry-policy.md` for execution tracking rules.

### Rule 15: Execution Logging & Audit Trail
Maintain detailed execution logs for audit and debugging. See `rules/logging.md` for log format, mandatory log points, audit requirements, and log management rules.

## Stage Details

### Stage 1: Requirement Brainstorming
**Sub-skill:** `brainstorming` (obra/superpowers)
**Input:** User's one-line idea (free text)
**Process:**
1. Invoke `brainstorming` skill to explore user intent, requirements, and design before implementation
2. Follow brainstorming's structured divergent + convergent thinking flow
3. Generate draft `docs/00-requirement/需求确认书.md`
4. Targeted follow-up questions based on uncertain items
5. Finalize `docs/00-requirement/需求确认书.md` with `status: confirmed`
**Gate 1:** Present finalized requirement doc for user sign-off.

### Stage 1.3: Project Type Selection
**Sub-skill:** Built-in (workflow orchestrator)
**Input:** `docs/00-requirement/需求确认书.md`
**Process:**
1. Analyze the requirements to recommend a project type
2. Use `AskUserQuestion` to let user select project type:
   - **Microservice**: 微服务架构，适合大型企业级应用
   - **Monolith**: 单体应用，适合中小型项目或快速原型
   - **Frontend-only**: 纯前端项目，无后端逻辑
   - **API-only**: 纯 API 服务，无前端界面
3. Update `docs/workflow-state.yaml` under `project_type:`
4. Adjust workflow stages based on project type selection

**Project Type Options:**

| 选项 | 描述 | 适用场景 | 包含阶段 |
|------|------|----------|----------|
| Microservice | 微服务架构 | 大型项目、团队协作、长期维护 | 完整流程（Stage 1~3） |
| Monolith | 单体应用 | 中小型项目、快速迭代 | 完整流程，简化架构设计 |
| Frontend-only | 纯前端项目 | 静态网站、单页应用、前端组件库 | 跳过 Stage 2.⑤、Group C、Stage 3 后端部分 |
| API-only | 纯 API 服务 | BFF 层、数据服务、第三方集成 | 跳过 Stage 2.③、Group B、Stage 3 前端部分 |

**Project Type → Stage Mapping:**

| 项目类型 | Stage 2.① | Stage 2.② | Group A (prototype) | Group A (backend_arch) | Group B | Group C | Stage 3 前端 | Stage 3 后端 |
|----------|-----------|-----------|---------------------|------------------------|---------|---------|-------------|-------------|
| Microservice | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Monolith | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Frontend-only | ✓ | ✓ | ✓ | ✗ | ✓ | ✗ | ✓ | ✗ |
| API-only | ✓ | ✓ | ✗ | ✓ | ✗ | ✓ | ✗ | ✓ |

### Stage 1.5: Tech Stack Selection
**Sub-skill:** Built-in (workflow orchestrator)
**Input:** `docs/00-requirement/需求确认书.md`, `docs/workflow-state.yaml` (project_type)
**Process:**
1. **智能技术栈推荐**: 分析需求文档中的关键字和特征，自动推荐最合适的技术栈:
   - **高并发/高性能需求** → 推荐 Go
   - **AI/ML 集成需求** → 推荐 Python
   - **快速迭代/全栈统一** → 推荐 Node.js
   - **企业级稳定性/大型团队** → 推荐 Java
   - **纯前端项目** → 跳过后端选择
   - **纯 API 项目** → 跳过前端选择
2. **推荐逻辑**:
   - 分析需求中的核心场景描述
   - 识别技术相关关键词(如"并发"、"机器学习"、"实时"、"大数据")
   - 根据项目类型(microservice/monolith/frontend-only/api-only)过滤选项
   - 综合评分后给出推荐排序
3. Present tech stack options with recommended order based on requirements
4. Use `AskUserQuestion` to let user select:
   - **Backend**: Java 17 + Spring Boot 3.x / Go 1.21 + Gin / Node.js + NestJS / Python 3.11 + FastAPI
   - **Frontend**: React + Next.js / Vue 3 + Nuxt
   - **Database**: PostgreSQL / MySQL / SQLite / MongoDB
5. Update `docs/workflow-state.yaml` under `tech_stack:`
6. Dynamically select the appropriate sub-skill for `frontend_arch` and `backend_arch` stages
7. Validate that required environment tools for selected tech stack are available

**智能推荐规则:**

| 需求特征 | 推荐后端 | 推荐理由 |
|----------|----------|----------|
| 高并发、高性能、低延迟 | Go | Go 的 goroutine 模型适合高并发场景 |
| AI、机器学习、数据科学 | Python | 丰富的 ML 库生态(pandas、scikit-learn、TensorFlow) |
| 实时通信、WebSocket | Node.js | 事件驱动模型适合实时应用 |
| 企业级、大型团队、长期维护 | Java | 成熟的生态和工具链，适合大型项目 |
| 快速原型、小型项目 | Node.js/Python | 开发效率高，学习曲线平缓 |
| 微服务架构 | Java/Go | 成熟的微服务框架和生态 |

**Tech Stack Options:**

| 选项 | 后端 | 前端 | 数据库 | 适用场景 |
|------|------|------|--------|----------|
| A - 企业级 | Java 17 + Spring Boot 3.x | React + Next.js | PostgreSQL | 大型项目、团队协作、长期维护 |
| B - 轻量级 | Go 1.21 + Gin | Vue 3 + Nuxt | SQLite/PostgreSQL | 快速迭代、高性能服务 |
| C - 全栈 JS | Node.js + NestJS | React + Next.js | MongoDB | 前后端统一技术栈 |
| D - Python | Python 3.11 + FastAPI | React + Next.js | PostgreSQL | AI/ML 集成、快速原型 |

### Stage 1.6: Cost & Risk Assessment
**Sub-skill:** Built-in (workflow orchestrator)
**Input:** `docs/00-requirement/需求确认书.md`, `docs/workflow-state.yaml` (project_type, tech_stack)
**Process:**
1. 分析需求复杂度和技术栈选择，进行成本估算
2. 识别潜在风险点和缓解措施
3. 生成评估报告 `docs/00-requirement/assessment-v1.md`
4. 呈现评估结果供用户确认

**成本评估维度:**

| 维度 | 评估指标 |
|------|----------|
| 开发成本 | 预估工时、人员配置、技术难度 |
| 维护成本 | 技术栈成熟度、社区支持、学习曲线 |
| 基础设施成本 | 服务器、数据库、云服务费用 |
| 时间成本 | 开发周期、迭代速度、部署时间 |

**风险评估维度:**

| 风险类型 | 评估指标 | 缓解措施 |
|----------|----------|----------|
| 技术风险 | 新技术栈学习、架构复杂度 | 技术调研、POC验证 |
| 业务风险 | 需求变更、范围蔓延 | 需求冻结、变更控制 |
| 进度风险 | 技术难题、依赖阻塞 | 预留缓冲时间、并行开发 |
| 质量风险 | 代码质量、测试覆盖 | 代码审查、自动化测试 |

### Stage 2: Artifact Generation (parallel groups)

#### 2.① Product Design
**Sub-skill:** `ui-ux-pro-max` (nextlevelbuilder, ⭐24.7k)
**Input:** `docs/00-requirement/需求确认书.md`
**Output:** `docs/01-product/design-v1.md` (with Mermaid diagrams: user journey, information architecture)
**Gate 2:** Present design for sign-off.

#### 2.② PRD
**Sub-skill:** `prd-development` (pm-skills)
**Input:** `docs/01-product/design-v1.md`
**Output:** `docs/02-prd/prd-v1.md` (problem context, target users, solution approach, success criteria)
**Gate 3:** Present PRD for sign-off.

#### Group A (parallel execution)

##### 2.③ UI Prototype
**Sub-skill:** `design-taste-frontend` (Leonxlnx/taste-skill)
**Input:** `docs/02-prd/prd-v1.md`
**Output:** `docs/03-prototype/index.html` (interactive, multi-page if needed)

##### 2.⑤ Backend Architecture
**Sub-skill:** Built-in (workflow orchestrator)
**Input:** `docs/02-prd/prd-v1.md`
**Process:** Generate backend architecture directly based on PRD + selected tech stack, without delegating to an external skill
**Output:** `docs/05-backend/architecture-v1.md` + `docs/05-backend/openapi.yaml` (module decomposition, OpenAPI spec, data storage, deployment topology)

#### Group B (depends on prototype completion)

##### 2.④ Frontend Architecture
**Sub-skill:** `frontend-design` (anthropics/skills)
**Input:** `docs/02-prd/prd-v1.md`, `docs/03-prototype/`
**Output:** `docs/04-frontend/architecture-v1.md` (directory structure, component tree, state management, routing)

#### Group C (depends on backend_arch completion)

##### 2.⑥ Test Cases
**Sub-skill:** `test-driven-development` (obra/superpowers)
**Input:** `docs/02-prd/prd-v1.md`, `docs/05-backend/openapi.yaml`
**Output:** `docs/06-test/cases-v1.md` (functional cases) + `docs/06-test/smoke/` (executable smoke cases)

**Gate 4:** Batch-present 2.③ 2.④ 2.⑤ 2.⑥ for user sign-off.

#### 2.⑦ Security Review
**Sub-skill:** `security-and-hardening`
**Input:** `docs/05-backend/architecture-v1.md`, `docs/04-frontend/architecture-v1.md`
**Output:** `docs/07-review/review-v1.md` (vulnerability assessment, threat model, hardening recommendations)
**Gate 5:** Present review findings + mitigation plan for user sign-off.

### Stage 3: Code Generation + Smoke Test
**Sub-skill:** `executing-plans` (obra/superpowers) + `test-driven-development` for unit tests
**Process:**
1. Invoke `executing-plans` to execute the implementation plan from PRD + architecture docs
2. For each module (in dependency order):
   - Use TDD: write test first → generate code → verify test passes
   - On pass: `git commit` → next module
   - On fail: auto-diagnose → fix → retry (max 3) → dialog if stuck
3. Generate frontend code from `docs/04-frontend/architecture-v1.md` and `docs/03-prototype/`
4. `git tag pre-smoke-test`
5. Run smoke test cases from `docs/06-test/smoke/`
6. Run cross-review via `doubt-driven-development` on generated code
7. On 100% pass + no critical findings: present final gate
8. On failure: per Rule 9 (granular rollback, max 3 retries, then dialog)

**Final gate:** Present passing smoke test results + cross-review findings + code summary. On user approval → deliver.

## Output Directory Structure

```
docs/
├── workflow-state.yaml          # State file (single source of truth)
├── progress.html                # Visual progress dashboard
├── progress.json                # Machine-readable progress API
├── 00-requirement/
│   └── 需求确认书.md
├── 01-product/
│   └── design-v1.md
├── 02-prd/
│   └── prd-v1.md
├── 03-prototype/
│   └── index.html
├── 04-frontend/
│   └── architecture-v1.md
├── 05-backend/
│   ├── architecture-v1.md
│   └── openapi.yaml
├── 06-test/
│   ├── cases-v1.md
│   └── smoke/
├── 07-review/
│   └── cross-review-v1.md
├── schemas/                     # Artifact contracts (from this skill)
├── rules/                       # Orchestration rules (from this skill)
└── checklists/                  # Gate validation checklists (from this skill)
```

## Sub-skill Configuration

Stored in `docs/workflow-state.yaml` under `skills:`. Each skill has metadata for discovery, fallback, and alternatives.

```yaml
skills:
  requirement:
    skill_id: brainstorming
    type: community
    source: https://github.com/obra/superpowers/tree/main/skills/brainstorming
    fallback: built-in-chat
    alternatives: []
    available: true
  product_design:
    skill_id: ui-ux-pro-max
    type: community
    source: https://github.com/nextlevelbuilder/skills
    fallback: built-in-chat
    alternatives: []
    available: true
  prd:
    skill_id: prd-development
    type: community
    source: https://github.com/dingxingkai/pm-skills/tree/main/skills/prd-development
    fallback: built-in-chat
    alternatives: []
    available: true
  prototype:
    skill_id: design-taste-frontend
    type: community
    source: https://github.com/Leonxlnx/taste-skill
    fallback: built-in-chat
    alternatives: []
    available: true
  frontend_arch:
    skill_id: frontend-design
    type: community
    source: https://github.com/anthropics/skills/tree/main/frontend-design
    fallback: built-in-chat
    alternatives: []
    available: true
  backend_arch:
    skill_id: built-in
    type: built-in
    fallback: built-in-chat
    alternatives: []
    available: true
  test_case:
    skill_id: test-driven-development
    type: community
    source: https://github.com/obra/superpowers/tree/main/skills/test-driven-development
    fallback: built-in-chat
    alternatives: []
    available: true
  security:
    skill_id: security-and-hardening
    type: community
    source: (built-in opencode skill)
    fallback: built-in-chat
    alternatives: []
    available: true
  code_generation:
    skill_id: executing-plans
    type: community
    source: https://github.com/obra/superpowers/tree/main/skills/executing-plans
    fallback: built-in-chat
    alternatives: []
    available: true
  cross_review:
    skill_id: doubt-driven-development
    type: community
    source: https://github.com/obra/superpowers/tree/main/skills/doubt-driven-development
    fallback: built-in-chat
    alternatives: []
    available: true
  parallel_dispatch:
    skill_id: dispatching-parallel-agents
    type: community
    source: https://github.com/obra/superpowers/tree/main/skills/dispatching-parallel-agents
    fallback: sequential
    alternatives: []
    available: true
```

**Skill types:**
- `built-in`: Core orchestrator skills (always available)
- `community`: Open-source skills from external repos
- `built-in-chat`: Fallback using direct LLM conversation

**Parallel dispatch skill:**
Group A (prototype + backend_arch) must be dispatched concurrently using `dispatching-parallel-agents` (obra/superpowers). This skill enables parallel subagent execution without shared state conflicts.

**Tech Stack Configuration:**

```yaml
tech_stack:
  backend: java-spring-boot              # java-spring-boot / go-gin / node-nestjs / python-fastapi
  frontend: react-next                   # react-next / vue-nuxt
  database: postgresql                   # postgresql / mysql / sqlite / mongodb
```

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
    ✓ brainstorming (obra/superpowers)
    ✓ ui-ux-pro-max (nextlevelbuilder)
    ✓ prd-development (pm-skills)
    ✓ design-taste-frontend (Leonxlnx)
    ✓ frontend-design (anthropics)
    ✓ test-driven-development (obra/superpowers)
    ✓ security-and-hardening (opencode built-in)
    ✓ executing-plans (obra/superpowers)
    ✓ doubt-driven-development (obra/superpowers)
    
    [Stage 1] 启动 brainstorming, 基于 "团队周报自动汇总工具" 探索需求...
    [发散 → 收敛 → 需求确认书]
    [Gate 1] 请确认需求确认书...
    
    [Stage 1.5] 技术选型...
    请选择技术栈:
    A - 企业级: Java 17 + Spring Boot 3.x + React + Next.js + PostgreSQL
    B - 轻量级: Go 1.21 + Gin + Vue 3 + Nuxt + SQLite
    C - 全栈 JS: Node.js + NestJS + React + Next.js + MongoDB
    D - Python: Python 3.11 + FastAPI + React + Next.js + PostgreSQL
    
    User: A
    
    [Stage 2] 启动产出物生成...
    [Group A] dispatching-parallel-agents → 并行启动 prototype 和 backend_arch...
    ...
```

## Important Notes

- This is an orchestrator, not an executor. Each stage delegates to its sub-skill via subagent.
- State (`docs/workflow-state.yaml`) is mandatory and authoritative. No state = no resume.
- Gates are hard stops. Never proceed without explicit confirmation.
- If a sub-skill is unavailable, try alternatives first, then fallback (Rule 11).
- Versioning: artifacts use v1, v2, ... Never overwrite — always increment for iterations.
- Tech stack is selected in Stage 1.5 (built-in); backend_arch is also built-in (no external skill).
- Stage 2 uses `dispatching-parallel-agents` for Group A parallel execution.
- `api-and-interface-design` skill is explicitly banned from this workflow.
- Progress is visible in `docs/progress.html` and `docs/progress.json`.
