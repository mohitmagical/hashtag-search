# Z Coach – AI Coding Workflow (GPT‑5‑Codex)

This playbook makes GPT‑5‑Codex (Programming GPT) your primary co‑dev for React + NestJS multi‑tenant Z Coach. Copy/paste the prompt blocks as-is. Keep this doc in your repo (`/docs/ai-workflow.md`).

---

## 0) Tech Standards (Baseline)

* **Language**: TypeScript strict mode everywhere.
* **FE**: React (Vite/Next), Zustand/Redux for state, React Query for data fetching, Tailwind + shadcn/ui, Recharts.
* **BE**: NestJS (modules, providers, controllers), TypeORM/Prisma (choose one), PostgreSQL (multi‑tenant via schema per tenant or tenant_id per row), Zod/DTO validation, Passport JWT (roles: superadmin, org_admin, trainer, learner).
* **Testing**: Vitest/Jest + React Testing Library (FE), Jest + Supertest (BE), e2e smoke.
* **Quality**: ESLint (airbnb + @typescript-eslint), Prettier, Husky + lint‑staged, commitlint conventional.
* **CI**: GitHub Actions: lint → typecheck → test → build → docker.
* **Security**: Helmet, rate-limit, CORS, OWASP headers, input validation.

---

## 1) Repository Layout

```
.
├─ apps/
│  ├─ web/                     # React app
│  │  ├─ src/
│  │  │  ├─ pages/ routes/ components/ hooks/ stores/
│  │  │  ├─ features/          # vertical slices
│  │  │  ├─ lib/               # http client, auth, utils
│  │  │  └─ tests/
│  │  └─ vite.config.ts
│  └─ api/                     # NestJS app
│     ├─ src/
│     │  ├─ modules/           # each domain module
│     │  ├─ common/            # interceptors, guards, pipes
│     │  ├─ prisma/ orm/
│     │  ├─ seeds/
│     │  └─ main.ts
│     └─ test/
├─ packages/
│  ├─ ui/                      # shared UI components
│  ├─ types/                   # shared TS types
│  └─ config/                  # eslint, tsconfig, tailwind, prettier
├─ docs/
│  └─ ai-workflow.md
├─ scripts/                    # dev/ops scripts
├─ .github/workflows/
└─ package.json / pnpm-workspace.yaml
```

---

## 2) Session Bootstrap (Paste Once per New Chat)

**System / Setup Prompt**

```
You are the lead engineer for Z Coach (React + NestJS multi-tenant SaaS). Follow these permanent rules:
- TypeScript strict, production-grade code, no pseudocode.
- Generate only changed files with paths, and unified diffs when editing.
- Include tests and brief reasoning. Cite assumptions.
- Follow repo layout provided; respect vertical-slice architecture.
- For security: validate inputs (Zod/DTO), sanitize outputs, enforce RBAC via Nest Guards.
- Performance: cache where reasonable, paginate, avoid N+1.
- Observability: add minimal logs, error boundaries, and health checks.
```

**Project Context Snippet Template** (keep ≤ 1–2k tokens):

```
Repo: Z Coach monorepo (pnpm). Key constraints: multi-tenant (tenant_id per row), roles (superadmin, org_admin, trainer, learner). DB: Postgres + Prisma.
Relevant files:
- apps/api/src/modules/users/*
- apps/web/src/features/auth/*
- packages/types/src/*.ts
```

---

## 3) Prompt Templates by Task

### A) Feature Implementation (Backend Module)

```
TASK: Implement feature <name>: <1-2 line description>.
ACCEPTANCE CRITERIA:
1) <criterion>
2) <criterion>
3) Auth: roles allowed = <roles>; tenant scoping required.
CONSTRAINTS: Postgres + Prisma; DTO validation; e2e test.
DELIVERABLES: New/edited file list + diffs; unit + e2e tests; seed data if needed.
STARTING POINT: <paste minimal code/context>
```

### B) Feature Implementation (Frontend Slice)

```
TASK: Build React feature <name> (page + components + API hooks).
UI: Tailwind + shadcn/ui; state via React Query; form via react-hook-form + zod.
ACCEPTANCE CRITERIA:
- Page route: /app/<route>
- Loading, error, empty states.
- Access controlled UI (role gates).
TESTS: RTL tests for components and hooks.
DELIVERABLES: file paths, code, tests, and minimal screenshot notes.
```

### C) Bugfix / Refactor

```
TASK: Diagnose and fix <bug/refactor area>.
CONTEXT: <stack traces / code>
EXPECTATIONS:
- Root cause analysis (2-4 bullets)
- Patch with diff
- Regression test
- Risk assessment + follow-ups
```

### D) API Contract & Client Hook

```
TASK: Define API endpoint(s) for <domain> and FE hooks.
BACKEND: Nest controller + service + DTO + guard (RBAC), Prisma model changes + migration.
FRONTEND: /lib/api client + React Query hook(s) with typing.
TESTS: Supertest e2e for API; RTL for hook (mock server).
OUTPUT: file list + code + tests + migration SQL.
```

### E) DB Migration + Seed

```
TASK: Add table/columns for <entity> with tenant_id and indices.
REQUIREMENTS: non-breaking migration, default values, backfill strategy.
DELIVERABLES: Prisma schema diff + migration SQL; seed script; rollback notes.
```

### F) PR Review (AI as Reviewer)

```
ROLE: You are a strict PR reviewer.
INPUTS: Patch/diff + context.
CHECK:
- Correctness, security, tenancy, RBAC.
- Types, null-safety, error handling.
- Tests cover AC.
OUTPUT: Blocking comments + suggestions + approve/changes requested.
```

### G) Test Authoring (TDD burst)

```
TASK: Write tests for <module/feature> before implementation.
SCOPE: unit + e2e minimal happy path + 1-2 edge cases.
EXPECTED: test files with imports, realistic mocks/fixtures.
```

### H) UI Component (Design‑system)

```
TASK: Implement reusable <ComponentName> in packages/ui with accessibility.
PROPS: typed, sensible defaults.
STATES: loading/disabled/error.
DOCS: story/mdx snippet and usage example.
TESTS: RTL basic render + a11y expectations.
```

---

## 4) Guardrails & Conventions

* **Tenancy**: Always filter by `tenant_id` from JWT/req context; forbid cross‑tenant queries.
* **RBAC**: Decorators `@Roles('org_admin')` + guard checks.
* **Validation**: Zod/DTO at boundaries; return typed errors.
* **Error handling**: Global filter → JSON problem details.
* **Pagination**: Default `limit=20`, `cursor` or `offset`.
* **Idempotency**: For mutations via UI, use React Query mutation keys and optimistic UI cautiously.
* **Security**: Never trust client; whitelist fields; use parameterized queries; sanitize HTML.
* **Logs**: Minimal `info` for start/end of operations; `error` with correlation id.

---

## 5) Branching, Commits, PRs

* **Branches**: `main` (protected) → `feat/<scope>`, `fix/<scope>`, `chore/<scope>`.
* **Conventional Commits**: `feat(auth): add refresh token rotation`.
* **PR Template** (put in `.github/pull_request_template.md`):

```
## Summary
## Acceptance Criteria
## Screenshots / Notes
## Tests
- [ ] Unit
- [ ] e2e
## Risk
- [ ] Migration
- [ ] Tenancy/RBAC verified
```

---

## 6) One‑Shot Bootstrap Prompts

### Install Tooling

```
TASK: Generate configs and scripts for ESLint, Prettier, Husky, lint-staged, commitlint, TS strict, GitHub Actions.
OUTPUT: files + package.json changes + pnpm commands.
```

### Auth & Roles Foundation

```
TASK: Scaffold NestJS auth (JWT access/refresh), roles guard, and sample `me` endpoint; FE login/register pages + protected route wrapper.
AC: JWT rotation, RBAC guard, tenant_id claim.
TESTS: e2e auth flow + FE RTL for login form.
```

### Course/Module Domain Slice

```
TASK: Implement Courses (CRUD, list, detail) with tenant scoping; FE pages: list/detail/create.
AC: pagination, search, RBAC (org_admin manage; learner read).
TESTS: unit + e2e + RTL.
```

---

## 7) Local Dev & CI Commands (put in README)

```bash
# Monorepo setup
pnpm i
pnpm -r build

# Backend
pnpm --filter @zcoach/api dev

# Frontend
pnpm --filter @zcoach/web dev

# Tests
pnpm -r test

# Lint & typecheck
pnpm -r lint
pnpm -r typecheck
```

---

## 8) “Context Packing” Tips (for GPT)

* Provide only relevant file excerpts and a short repo map.
* Pin **acceptance criteria** and **constraints** in every prompt.
* Ask for **file list + diffs** to minimize copy errors.
* Keep a running CHANGELOG of AI-made edits.

---

## 9) Definition of Done (per task)

* Meets AC; passes lint/type/test; respects tenancy/RBAC.
* Includes tests and minimal docs.
* No secrets committed; envs handled via `.env.example`.

---

## 10) On‑Call Prompts (Copy/Paste)

* **“Write me a failing test first for <X>, then implement.”**
* **“Show unified diffs only, respect file paths, don’t reprint unchanged code.”**
* **“List risks and rollbacks for this migration.”**
* **“Act as PR reviewer: block or approve with exact comments.”**

---

**Use this as your single source of truth when pairing with GPT‑5‑Codex on Z Coach.**
