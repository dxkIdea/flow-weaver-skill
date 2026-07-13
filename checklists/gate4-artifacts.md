# Gate 4 Checklist: Artifacts Batch Sign-off

## Purpose
Gate 4 在 Stage 2 所有并行组完成后、Stage 3 开始前执行。批量审阅 4 个产出物(UI Prototype、Frontend Architecture、Backend Architecture、Test Cases)。

## Parallel Group Completion Check
在开始自动验证前,确认所有并行组已完成:
- [ ] Group A (prototype + backend_arch) 状态为 completed
- [ ] Group B (frontend_arch) 状态为 completed
- [ ] Group C (test_case) 状态为 completed

## Schema Routing (技术栈特定)

根据 `tech_stack` 选择对应的 schema 文件:

| 技术栈 | 后端 Schema | 前端 Schema |
|--------|-------------|-------------|
| Java Spring Boot | `backend-arch-schema.yaml` | `frontend-arch-schema.yaml` |
| Go Gin | `backend-arch-schema-go.yaml` | `frontend-arch-schema.yaml` |
| Node.js NestJS | `backend-arch-schema-node.yaml` | `frontend-arch-schema.yaml` |
| Python FastAPI | `backend-arch-schema-python.yaml` | `frontend-arch-schema.yaml` |
| Vue Nuxt | 对应后端 schema | `frontend-arch-schema-vue.yaml` |

## Auto-Validation (per artifact)

### UI Prototype (对照 prototype-schema.yaml)
- [ ] `docs/03-prototype/index.html` 存在且为有效 HTML
- [ ] 页面清单 ≥1
- [ ] 导航结构完整,引用的页面文件都存在
- [ ] 链接/按钮可交互(有 href 或 onclick)
- [ ] 覆盖 requirement 的所有核心场景
- [ ] 主要流程无 lorem-ipsum 占位

### Frontend Architecture (根据 tech_stack.frontend 选择 schema)
**React + Next.js (对照 frontend-arch-schema.yaml):**
- [ ] `docs/04-frontend/architecture-v1.md` 存在
- [ ] tech_stack 含 framework=React + Next.js/language=TypeScript/styling/state_management/http_client/testing
- [ ] directory_structure 为有效 tree
- [ ] component_tree ≥1 组件,每个含 responsibility
- [ ] routing 覆盖 prototype 的所有页面
- [ ] api_integration 覆盖 PRD 的所有 api_contracts

**Vue 3 + Nuxt (对照 frontend-arch-schema-vue.yaml):**
- [ ] `docs/04-frontend/architecture-v1.md` 存在
- [ ] tech_stack 含 framework=Vue 3 + Nuxt/language=TypeScript/styling/state_management/http_client/testing
- [ ] directory_structure 为有效 tree
- [ ] component_tree ≥1 组件,每个含 responsibility
- [ ] routing 覆盖 prototype 的所有页面
- [ ] api_integration 覆盖 PRD 的所有 api_contracts

### Backend Architecture (根据 tech_stack.backend 选择 schema)
**Java Spring Boot (对照 backend-arch-schema.yaml):**
- [ ] `docs/05-backend/architecture-v1.md` 存在
- [ ] `docs/05-backend/openapi.yaml` 存在且可解析为 OpenAPI 3.x
- [ ] tech_stack 含 Java 17+/Spring Boot 3.x/build_tool/orm/database
- [ ] module_decomposition ≥1 模块,每模块含 name/responsibility/depends_on/entities/api_endpoints/priority
- [ ] 每个模块的 api_endpoints 在 openapi.yaml 中存在
- [ ] 每个模块的 entities 在 PRD data_models 中存在
- [ ] module depends_on 无循环依赖
- [ ] 含分层架构(controller/service/mapper/entity/dto)
- [ ] 含安全设计(authentication/authorization)
- [ ] 含异常处理
- [ ] 含部署拓扑(Mermaid)

**Go Gin (对照 backend-arch-schema-go.yaml):**
- [ ] `docs/05-backend/architecture-v1.md` 存在
- [ ] `docs/05-backend/openapi.yaml` 存在且可解析为 OpenAPI 3.x
- [ ] tech_stack 含 Go 1.21+/Gin/build_tool/orm/database
- [ ] module_decomposition ≥1 模块,每模块含 name/responsibility/depends_on/entities/api_endpoints/priority
- [ ] 每个模块的 api_endpoints 在 openapi.yaml 中存在
- [ ] 每个模块的 entities 在 PRD data_models 中存在
- [ ] module depends_on 无循环依赖
- [ ] 含分层架构(handler/service/repository/entity/dto)
- [ ] 含安全设计(authentication/authorization)
- [ ] 含异常处理
- [ ] 含部署拓扑(Mermaid)

**Node.js NestJS (对照 backend-arch-schema-node.yaml):**
- [ ] `docs/05-backend/architecture-v1.md` 存在
- [ ] `docs/05-backend/openapi.yaml` 存在且可解析为 OpenAPI 3.x
- [ ] tech_stack 含 Node.js 18+/NestJS/build_tool/orm/database
- [ ] module_decomposition ≥1 模块,每模块含 name/responsibility/depends_on/entities/api_endpoints/priority
- [ ] 每个模块的 api_endpoints 在 openapi.yaml 中存在
- [ ] 每个模块的 entities 在 PRD data_models 中存在
- [ ] module depends_on 无循环依赖
- [ ] 含分层架构(controller/service/repository/entity/dto)
- [ ] 含安全设计(authentication/authorization)
- [ ] 含异常处理
- [ ] 含部署拓扑(Mermaid)

**Python FastAPI (对照 backend-arch-schema-python.yaml):**
- [ ] `docs/05-backend/architecture-v1.md` 存在
- [ ] `docs/05-backend/openapi.yaml` 存在且可解析为 OpenAPI 3.x
- [ ] tech_stack 含 Python 3.11+/FastAPI/build_tool/orm/database
- [ ] module_decomposition ≥1 模块,每模块含 name/responsibility/depends_on/entities/api_endpoints/priority
- [ ] 每个模块的 api_endpoints 在 openapi.yaml 中存在
- [ ] 每个模块的 entities 在 PRD data_models 中存在
- [ ] module depends_on 无循环依赖
- [ ] 含分层架构(router/service/repository/model/schema)
- [ ] 含安全设计(authentication/authorization)
- [ ] 含异常处理
- [ ] 含部署拓扑(Mermaid)

### Test Cases (对照 test-case-schema.yaml)
- [ ] `docs/06-test/cases-v1.md` 存在
- [ ] `docs/06-test/smoke/` 目录存在且有用例文件
- [ ] functional_cases ≥1,每个含 id/module/title/preconditions/steps/expected_result/priority/type
- [ ] smoke_cases ≥1,每个含 id/title/endpoint/method/request_payload/expected_status/executable=true
- [ ] 所有 case id 唯一
- [ ] 每个 P0 功能至少 1 个 functional_case
- [ ] 每个 core_scenario 至少 1 个 smoke_case
- [ ] smoke_case 的 endpoint 在 openapi.yaml 中存在
- [ ] coverage_matrix 覆盖所有 P0 需求

## Cross-Artifact Consistency
- [ ] 前端 api_integration 的接口与后端 openapi.yaml 一致
- [ ] 测试用例的 module 与后端 module_decomposition 一致
- [ ] 前端 routing 的页面与 prototype 页面一致
- [ ] backend_arch 的 tech_stack 与 workflow-state.yaml 中的 tech_stack.backend 一致
- [ ] frontend_arch 的 tech_stack 与 workflow-state.yaml 中的 tech_stack.frontend 一致

## Human Review Prompts

批量审阅,建议用户重点看:
- 后端模块划分是否合理
- OpenAPI 接口设计
- 冒烟测试用例是否覆盖核心路径
- 技术栈选择是否与实际架构一致

- 选项 A: 全部确认,进入代码生成
- 选项 B: 部分修改(指明哪几个产出物)
- 选项 C: 全部重做

## On Reject
- 选 B: 针对指定产出物修订,版本递增,触发 change-propagation(下游标记 stale)
- 选 C: 清空 Group A、B、C 的所有产出物,按并行组顺序重做
