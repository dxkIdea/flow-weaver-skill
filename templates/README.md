# Project Templates

This directory contains starter scaffolds for each supported technology stack.
During Stage 3 code generation, the orchestrator instantiates the appropriate
template and populates it with generated code.

## Directory Structure

```
templates/
├── backend-spring-boot/     # Java 17 + Spring Boot 3.x + Maven
├── backend-go-gin/          # Go 1.21+ + Gin
├── backend-nestjs/          # Node.js 18+ + NestJS + TypeScript
├── backend-fastapi/         # Python 3.11+ + FastAPI
├── frontend-react-next/     # React + Next.js + TypeScript + Tailwind
├── frontend-vue-nuxt/       # Vue 3 + Nuxt 3 + TypeScript + Tailwind
└── openapi-default.yaml     # Empty OpenAPI 3.x starter spec
```

## Usage

The flow-weaver orchestrator copies the relevant template to the project
root during Stage 3 initialization. Sub-agents then populate the scaffold
with generated code modules.

## Template Contract

Each template must provide:

1. A valid buildable project (compiles/runs with empty routes)
2. Standard layered directory structure matching the architecture schema
3. A basic `health` or `ping` endpoint for smoke test verification
4. `.gitignore` appropriate to the stack
5. A `README.md` with setup instructions
