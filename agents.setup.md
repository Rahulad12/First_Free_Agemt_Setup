# agents Agent Setup — Universal Bootstrap

> **How to use:** Copy this file to the root of any new project, then tell agents:
> "Read `agents.setup.md` and set up this project for agentic development."
>
> agents will explore the project, ask a few questions, then create the full `.agents/` setup.

---

## What this creates

```
.agents/
├── README.md                        ← master navigation guide
├── settings.json                    ← permissions, hooks, safety blocks
├── rules/
│   ├── architectural.md             ← 4-phase pre-flight checklist for every new feature
│   ├── code-style.md                ← naming, imports, TypeScript/JS, banned patterns, UI + color rules
│   ├── api-conventions.md           ← service layer, data fetching, mutation patterns
│   ├── module-structure.md          ← monorepo-aware folder structure rules (ALL packages)
│   ├── security.md                  ← auth, secrets, input validation
│   └── testing.md                   ← test structure and mocking rules
├── wiki/
│   ├── index.md                     ← table of contents
│   ├── WIKI.md                      ← complete project reference
│   └── log.md                       ← append-only change log
├── taskboard/
│   ├── TASKBOARD.md                 ← live GitLab task board mirror (auto-updated)
│   └── README.md                    ← format spec and update rules
├── skills/
│   ├── taskboard.skill              ← skill discovery manifest (REQUIRED alongside folder)
│   ├── prompt.skill                 ← skill discovery manifest (REQUIRED alongside folder)
│   ├── taskboard/
│   │   └── SKILL.md                 ← GitLab task board skill implementation
│   └── prompt/
│       └── SKILL.md                 ← prompt enhancement skill implementation
└── raw/
    └── README.md                    ← immutable source document store
agents.md                            ← root instructions (required by agents Code)
```

> **Skill file convention — CRITICAL:**
> Every skill requires **two** co-located artifacts:
> 1. `.agents/skills/<name>.skill` — YAML discovery manifest. The agent reads this to know the skill exists and how to invoke it.
> 2. `.agents/skills/<name>/SKILL.md` — Full implementation, steps, and examples.
>
> Both must be present and kept in sync. The `.skill` file is the entry point; the `SKILL.md` is the body.
> A skill folder without a `.skill` file is invisible to agents. A `.skill` file without a `SKILL.md` will error on invocation.

---

## Phase 1 — Explore the project

Before creating any file, gather context:

```bash
# Identify tech stack
cat package.json 2>/dev/null || cat pyproject.toml 2>/dev/null || cat go.mod 2>/dev/null || cat Cargo.toml 2>/dev/null || cat pom.xml 2>/dev/null

# Detect monorepo tooling
cat pnpm-workspace.yaml 2>/dev/null || cat lerna.json 2>/dev/null || cat nx.json 2>/dev/null || cat turbo.json 2>/dev/null || echo "no monorepo config found"

# List top-level packages / apps / services
ls packages/ 2>/dev/null && echo "--- packages ---"
ls apps/ 2>/dev/null && echo "--- apps ---"
ls services/ 2>/dev/null && echo "--- services ---"
ls backend/ 2>/dev/null && echo "--- backend ---"
ls frontend/ 2>/dev/null && echo "--- frontend ---"

# Recent commits
git log --oneline -10

# Existing agents.md
cat agents.md 2>/dev/null || echo "none"
```

Then ask the user **one message with all questions at once**:

> 1. **Project name** — what do we call this project?
> 2. **Repo type** — single package, monorepo (list all packages/apps/services), or polyrepo?
> 3. **GitLab URL** — e.g. `https://gitlab.example.com/group/project`
> 4. **GitLab project ID** — found in GitLab → Settings → General → Project ID
> 5. **Protected branch** — the branch no one pushes to directly (usually `master` or `main`)
> 6. **Integration branch** — where feature branches merge into (usually `develop` or `staging`)
> 7. **Any legacy rules** — files/folders/patterns that must never be changed?
> 8. **UI framework(s)** — React, Vue, Angular, plain HTML, none? (per package if monorepo)
> 9. **Component library** — Shadcn, Vuetify, Material UI, Tailwind-only, other, none?
> 10. **Backend language/framework** — Node/Express, Python/FastAPI, Go, Java/Spring, none?

Wait for all answers before Phase 2.

---

## Phase 2 — Detect monorepo layout and map all packages

If the project is a **monorepo**, map every package before writing any rules:

```bash
# For each discovered package/app/service:
for dir in packages/* apps/* services/* backend frontend 2>/dev/null; do
  [ -d "$dir" ] && echo "=== $dir ===" && cat "$dir/package.json" 2>/dev/null | grep '"name"\|"main"\|"scripts"' || true
done

# Identify src structure per package
find . -name "src" -maxdepth 4 -type d 2>/dev/null | grep -v node_modules
```

Build a mental map of:
- Package name → type (frontend app / backend service / shared library / tooling)
- Primary language and framework per package
- Shared packages and who imports them
- Whether packages have independent or root-level build/lint/test runners

This map drives `module-structure.md`, `agents.md`, and `settings.json`.

---

## Phase 3 — Create folder structure

```bash
mkdir -p .agents/rules .agents/wiki .agents/taskboard .agents/skills/taskboard .agents/skills/prompt .agents/raw
```

---

## Phase 4 — Write `agents.md`

Create `agents.md` at the project root. Fill in `[PLACEHOLDERS]` from Phase 1–2 answers.
For monorepos, include a **per-package section** under "Dev Commands" and "Key Files".

```markdown
# [PROJECT_NAME] — agents Instructions

**Stack:** [TECH_STACK — list per package for monorepos]
**Repo type:** [single | monorepo — list package names]

---

## Start Here

Before any feature work, read:
- `.agents/README.md` — master navigation for all rules, wiki, and skills
- `.agents/rules/architectural.md` — mandatory 4-phase checklist for every new feature
- `.agents/rules/module-structure.md` — strict folder rules for EVERY package in this repo

---

## Critical Rules — NEVER Break

1. Follow the exact folder and file structure for each package — see `.agents/rules/module-structure.md`
2. Never push to `[PROTECTED_BRANCH]` — always push to `[INTEGRATION_BRANCH]` or a feature branch
3. Never rename, reorganize, or restructure existing files — this is a [legacy/established] codebase
4. Never add files to a package that belongs to a different layer (e.g. no DB code in frontend packages)
5. Form validation must use a schema library — never trust raw form input
6. Never store secrets or tokens in source code or non-standard storage locations
7. All UI must use the project's component library — no inline styles or one-off components
8. Shared logic lives in shared/common packages only — never duplicate across app packages

---

## Dev Commands

### Root (monorepo runner)
```bash
[ROOT_DEV_COMMAND]     # start all services (e.g. pnpm dev, turbo dev)
[ROOT_BUILD_COMMAND]   # build everything
[ROOT_LINT_COMMAND]    # lint all packages
[ROOT_TEST_COMMAND]    # test all packages
```

### Per-package
```bash
# [PACKAGE_1_NAME] — [type: frontend/backend/lib]
cd [PACKAGE_1_PATH] && [DEV_CMD]
cd [PACKAGE_1_PATH] && [LINT_CMD]
cd [PACKAGE_1_PATH] && [TEST_CMD]

# [PACKAGE_2_NAME] — [type]
cd [PACKAGE_2_PATH] && [DEV_CMD]
# ... repeat for each package
```

---

## Packages / Services

| Package | Path | Type | Stack |
|---|---|---|---|
| [PACKAGE_NAME] | [path] | frontend \| backend \| lib | [stack] |

---

## Key Files

| File | Package | Purpose |
|---|---|---|
| [path] | [package] | [purpose] |

---

## Feature Workflow

1. Branch: `feature/<issue-id>-<short-slug>` off `[INTEGRATION_BRANCH]`
2. Follow the 4-phase checklist in `.agents/rules/architectural.md`
3. Apply the correct module structure for the package being edited — see `.agents/rules/module-structure.md`
4. Lint must pass (root + affected packages) with zero errors before commit
5. Open MR targeting `[INTEGRATION_BRANCH]` with `Closes #<id>` in description

---

## Rules Reference

| Rule file | Covers | Auto-loads for |
|---|---|---|
| `architectural.md` | 4-phase feature pre-flight checklist | source module files |
| `module-structure.md` | Folder trees per package, naming, barrel files, cross-package rules | module, route, service files |
| `api-conventions.md` | Data fetching, mutations, error handling | service/API files |
| `code-style.md` | TypeScript/JS, imports, UI, colors, banned patterns | all source files |
| `security.md` | Auth, secrets, input validation | auth and API files |
| `testing.md` | Test structure, mocking, coverage | test files |

---

## Skills

| Skill | Invoke | Purpose |
|---|---|---|
| taskboard | `/taskboard [sync\|new <title>\|move #<iid> <col>]` | GitLab board management |
| prompt | `/prompt <your draft prompt or text>` | Enhance and refine prompts |

---

## Wiki

Project knowledge base lives in `.agents/wiki/`. Start with `.agents/wiki/index.md`.

**Operating rules:**
- Never modify anything in `.agents/raw/`
- Always update `.agents/wiki/index.md` and `.agents/wiki/log.md` after any wiki change
- Page names: lowercase with hyphens (`feature-name.md`)
- All agents-generated knowledge goes inside `.agents/wiki/` — never at repo root
```

---

## Phase 5 — Write `.agents/settings.json`

For monorepos, the lint/build hooks reference the root runner. Adjust if packages use independent scripts.

```json
{
  "$schema": "https://json.schemastore.org/agents-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint *)",
      "Bash(npm run build *)",
      "Bash(npm run test *)",
      "Bash(npm run typecheck *)",
      "Bash(pnpm lint *)",
      "Bash(pnpm build *)",
      "Bash(pnpm test *)",
      "Bash(pnpm typecheck *)",
      "Bash(turbo lint *)",
      "Bash(turbo build *)",
      "Bash(turbo test *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(git switch *)",
      "Bash(git fetch *)",
      "Bash(git pull *)",
      "Bash(git stash *)",
      "Bash(git restore *)",
      "Bash(git merge *)",
      "Bash(git rebase *)",
      "Bash(git push origin [INTEGRATION_BRANCH])",
      "Bash(git push origin feature/*)",
      "Bash(curl -s *)",
      "Read(*)",
      "Grep(*)",
      "Glob(*)"
    ],
    "deny": [
      "Bash(git push * [PROTECTED_BRANCH])",
      "Bash(git push * main)",
      "Bash(git push * master)",
      "Bash(git push --force *)",
      "Bash(git push -f *)",
      "Bash(rm -rf *)",
      "Bash(curl * | bash)",
      "Bash(wget * | sh)",
      "Read(.env)",
      "Read(.env.*)",
      "Read(*secret*)",
      "Read(*credential*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[ROOT_LINT_COMMAND] 2>&1 | tail -5 || true"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "git status --short 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**Important:** Replace `[PROTECTED_BRANCH]`, `[INTEGRATION_BRANCH]`, and `[ROOT_LINT_COMMAND]` with real values. Remove the `pnpm`/`turbo` allow entries that don't apply to this project.

---

## Phase 6 — Write `.agents/README.md`

```markdown
# .agents/ — Master Navigation

Everything agents needs to work on [PROJECT_NAME] lives here, plus `agents.md` at the repo root.

---

## Structure

```
.agents/
├── README.md                        ← this file
├── settings.json                    ← permissions, hooks, safety blocks
├── rules/                           ← coding rules (auto-load via path frontmatter)
│   ├── architectural.md             ← 4-phase pre-flight checklist for new features
│   ├── code-style.md                ← naming, imports, UI, colors, banned patterns
│   ├── api-conventions.md           ← data fetching and mutation patterns
│   ├── module-structure.md          ← strict folder tree rules for ALL packages
│   ├── security.md                  ← auth, secrets, validation
│   └── testing.md                   ← test structure and mocking
├── wiki/                            ← project knowledge base
│   ├── index.md                     ← table of contents — read first
│   ├── WIKI.md                      ← complete project reference
│   └── log.md                       ← append-only change log
├── taskboard/                       ← live mirror of GitLab task board
│   ├── TASKBOARD.md                 ← active milestone + issues (auto-updated)
│   └── README.md                    ← format spec
├── skills/
│   ├── taskboard.skill              ← discovery manifest (entry point for agents)
│   ├── prompt.skill                 ← discovery manifest (entry point for agents)
│   ├── taskboard/SKILL.md           ← taskboard implementation
│   └── prompt/SKILL.md             ← prompt enhancement implementation
└── raw/                             ← immutable source documents for wiki ingestion
    └── README.md
```

> **Skill file convention:**
> Each skill has two required files:
> - `<name>.skill` — YAML manifest agents reads to discover the skill
> - `<name>/SKILL.md` — full implementation the agent executes
> Both must stay in sync. Never add a skill folder without its `.skill` manifest.

---

## Quick reference

| I need to… | Go to |
|---|---|
| Understand the project | `.agents/wiki/WIKI.md` |
| Start a new feature | `.agents/rules/architectural.md` |
| Check folder structure rules | `.agents/rules/module-structure.md` |
| Check data fetching patterns | `.agents/rules/api-conventions.md` |
| Check naming / style rules | `.agents/rules/code-style.md` |
| View current task board | `.agents/taskboard/TASKBOARD.md` |
| Sync task board from GitLab | `/taskboard sync` |
| Improve a prompt or issue description | `/prompt <draft>` |

---

## Rules — how they load

| Rule | Applies to |
|---|---|
| `architectural.md` | module and route files across all packages |
| `code-style.md` | all source files |
| `api-conventions.md` | service / API / hook files |
| `module-structure.md` | module, route, service files in ALL packages |
| `security.md` | auth, config, API files |
| `testing.md` | test and spec files |

---

## Wiki — how it works

- **Add knowledge:** drop a source file in `.agents/raw/`, ask agents to ingest it
- **Ask questions:** agents reads `.agents/wiki/index.md` first, then relevant pages
- **After changes:** always append to `.agents/wiki/log.md` and update `.agents/wiki/index.md`
```

---

## Phase 7 — Write all 6 rules files

### `.agents/rules/architectural.md`

```markdown
---
paths:
  - "src/**"
  - "apps/**"
  - "packages/**"
  - "services/**"
  - "backend/**"
  - "frontend/**"
---
# Architectural Pre-Flight Checklist

Complete every phase in order before writing a single line of code for any new feature.

---

## Phase 1 — Understand
- [ ] Identify which **package(s)** are affected — frontend, backend, shared lib, or multiple
- [ ] Confirm the type of change: new module · new page · update existing · service-only · cross-package
- [ ] Find the most similar existing code in the same package and read it
- [ ] Identify page/component type: table · form · dashboard · utility
- [ ] List all API endpoints needed (method, path, request shape, response shape)
- [ ] Confirm: new module, update existing, or cross-package shared logic?
- [ ] For cross-package features: define the API contract (types, shared lib exports) before touching either side

> **Exit gate:** state the file list with package paths before creating anything.

---

## Phase 2 — Scaffold
- [ ] Create folder tree matching the established pattern for the target package — see `module-structure.md`
- [ ] Write type definitions first (shared types go in the shared/common package)
- [ ] Write service/API functions (raw, no UI)
- [ ] Write route/navigation registration
- [ ] Create a minimal stub component so the build passes

> **Exit gate:** build passes with stub content.

---

## Phase 3 — Build
- [ ] Write data-fetching hooks with loading and error states
- [ ] Write mutation hooks with success toast + cache invalidation
- [ ] Write column/table definitions if data table
- [ ] Write the main page/component — filters via URL params, never client-side filter response data
- [ ] Write form with schema validation (never trust raw input)
- [ ] Extract any stateful element into its own component to isolate re-renders
- [ ] Avoid `useEffect` — derive from state or use event handlers; only use `useEffect` for genuine external system sync with a written justification comment
- [ ] Cross-package: backend endpoint complete and tested before frontend integration begins

> **Exit gate:** page renders, data loads, all interactions work end-to-end.

---

## Phase 4 — Verify
### Code quality
- [ ] Lint — zero errors (root runner or per-package, whichever applies)
- [ ] Build — passes for all affected packages

### Data correctness
- [ ] All filters send params to the API — no client-side `.filter()` on response data
- [ ] Loading and error states handled

### Architecture compliance
- [ ] No relative imports across package boundaries — use workspace aliases
- [ ] No `console.log` in committed code
- [ ] No hardcoded color values or inline styles — use design tokens
- [ ] No `// @ts-ignore` — fix the type
- [ ] Folder tree matches the established pattern for each affected package
- [ ] Shared logic added to shared packages, not duplicated

### Branch safety
- [ ] Branch is `[INTEGRATION_BRANCH]` or `feature/<id>-slug` — **never push to `[PROTECTED_BRANCH]`**

> **Exit gate:** all boxes checked. Commit and open MR targeting `[INTEGRATION_BRANCH]`.
```

---

### `.agents/rules/code-style.md`

```markdown
---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.py"
  - "**/*.go"
---
# Code Style

## Naming
- Component files: PascalCase or kebab-case (match the convention of the package you're editing)
- Utility files: kebab-case
- Custom hooks: camelCase with `use` prefix
- Constants: UPPER_SNAKE_CASE
- Types/Interfaces: PascalCase
- Backend handlers/controllers: camelCase or snake_case (match existing package convention)

## Imports
- Use workspace aliases or package names across packages — never `../../` across package boundaries
- Within a package: use the package's configured path alias (`@/`, `~/`, `#/`, etc.)
- Group: external libs → workspace packages → package-local

## TypeScript / JavaScript
- No `any` — use `unknown` + type guards
- Explicit return types on exported functions
- Derive form types from schema (`z.infer<typeof schema>` or equivalent)

## Python (if applicable)
- Type hints on all public function signatures
- No bare `except` — always catch a specific exception type
- Use dataclasses or Pydantic models for structured data — no raw `dict` passed between layers

## Go (if applicable)
- All exported functions have GoDoc comments
- Errors wrapped with context: `fmt.Errorf("doing X: %w", err)`
- No `panic` in library code

## React / Vue
- Functional components only (React)
- Custom hooks for all stateful logic extracted from components
- No inline event handlers longer than one line — extract to named handlers

## UI Components
- Always use the project's component library — never build custom buttons/inputs/modals if the library has them
- Available components: [COMPONENT_LIBRARY_PATH]

## Styling
- All styling via the CSS utility class system (Tailwind / UnoCSS / etc.)
- No `style={{}}` inline props
- No hardcoded color values — always use design tokens / CSS variables
- Semantic tokens: `bg-primary`, `text-muted-foreground`, `bg-destructive`, etc.

## Banned patterns
- No direct HTTP client calls in components — use the service layer
- No `localStorage` access outside the designated utility
- No hardcoded URL strings — use config constants
- No hardcoded colors or inline styles
- No `console.log` in committed code
- No `// @ts-ignore`
- No business logic in route handlers — delegate to service/use-case layer
```

---

### `.agents/rules/api-conventions.md`

```markdown
---
paths:
  - "**/services/**"
  - "**/api/**"
  - "**/hooks/**"
  - "**/queries/**"
  - "**/routes/**"
  - "**/controllers/**"
  - "**/handlers/**"
---
# API Conventions

## Service layer (frontend)
- Raw async functions only — no UI logic, no hooks
- Always use the project's HTTP wrapper — never the raw HTTP client directly
- Exception: multipart/form-data uploads may use the client directly with correct Content-Type

## Service layer (backend)
- Controllers/handlers call services — never put business logic in the route handler
- Services call repositories/data-access — never write raw queries in services
- Repository layer is the only place that touches the database or ORM

## Data fetching (React Query / SWR / TanStack / etc.)
- Always use `keepPreviousData` / `placeholderData` on paginated list queries
- `staleTime` and `refetchOnMount` set globally — do not override without a specific reason
- Import service functions from the services index — never call raw functions from components

## Mutations
- `onSuccess`: show success toast + invalidate relevant query keys (`exact: false`)
- `onError`: use `showToastErrors(error)` — never `toast.error(err.message)` directly
- Use `isPending` (not `isLoading`) for mutation loading state

## Query keys
- Use a key factory function — never hand-roll key arrays
- `MAIN_KEY` string: kebab-case matching the resource name
- Key factory always exported as `default`

## API response shape
- Success: `{ data: T, meta?: PaginationMeta }`
- Error: `{ error: { code: string, message: string, details?: unknown } }`
- Never return raw database models — always map to a DTO/response type

## Error handling
- 401 / 403 handled globally by the HTTP interceptor — do not duplicate in services
- Never access `error.response` directly in components — use the formatted error object
- Backend: all unhandled errors caught by a global error handler middleware
```

---

### `.agents/rules/module-structure.md`

```markdown
---
paths:
  - "src/**"
  - "apps/**"
  - "packages/**"
  - "services/**"
  - "backend/**"
  - "frontend/**"
---
# Module Structure — Monorepo Edition

This rule covers ALL packages in the repository.
Each package has its own section. Adding files to the wrong package, or creating a
new top-level folder without approval, is a critical violation.

---

## Repository Layout

```
[PROJECT_ROOT]/
├── [FRONTEND_PATH]/            ← frontend application(s)
├── [BACKEND_PATH]/             ← backend service(s)
├── [SHARED_PATH]/              ← shared types, utilities, constants
├── [TOOLING_PATH]/             ← build tools, scripts, configs (if present)
├── package.json                ← root workspace manifest
└── [MONOREPO_CONFIG_FILE]      ← pnpm-workspace.yaml / turbo.json / nx.json
```

> **[FILL IN DURING SETUP]** Replace every placeholder with real paths from Phase 2 exploration.

---

## SECTION A — Frontend Package(s)

> Path: `[FRONTEND_PATH]` (e.g. `apps/web`, `frontend`)

### Required folder tree for a new feature module

```
[FRONTEND_PATH]/src/
└── modules/
    └── <feature-name>/             ← kebab-case, matches the domain noun
        ├── index.ts                ← barrel — only export the public API
        ├── types.ts                ← all TypeScript types/interfaces for this module
        ├── services/
        │   └── <feature>.service.ts   ← raw async API calls, no UI
        ├── hooks/
        │   ├── use<Feature>Query.ts
        │   └── use<Feature>Mutation.ts
        ├── components/
        │   ├── <Feature>Page.tsx
        │   ├── <Feature>Form.tsx   ← (if feature has a form)
        │   └── <Feature>Table.tsx  ← (if feature has a list)
        └── __tests__/
            └── <Feature>.test.tsx
```

### Rules
- Every module must have an `index.ts` barrel — other packages import from the barrel only
- Service functions import only from the HTTP wrapper, not from other modules
- Hooks import from `services/` via the barrel — never a direct relative path across modules
- `types.ts` may import from the shared package — never from another feature module
- Route registration: **[FILL IN — e.g. `src/router/index.ts` or `src/App.tsx`]**
- No new top-level folders under `src/` without explicit approval

---

## SECTION B — Backend Package(s)

> Path: `[BACKEND_PATH]` (e.g. `apps/api`, `backend`, `services/api`)

### Required folder tree for a new feature module

```
[BACKEND_PATH]/src/
└── modules/
    └── <feature-name>/
        ├── index.ts / __init__.py    ← package entry / barrel
        ├── <feature>.types.ts        ← request/response DTOs
        ├── <feature>.schema.ts       ← validation schema (Zod / Pydantic / Joi)
        ├── <feature>.controller.ts   ← route handler — calls service only
        ├── <feature>.service.ts      ← business logic — calls repository only
        ├── <feature>.repository.ts   ← data access — only place ORM/DB is used
        ├── <feature>.routes.ts       ← route definitions and middleware wiring
        └── tests/
            ├── <feature>.service.test.ts
            └── <feature>.controller.test.ts
```

### Rules
- Controllers NEVER contain business logic — one line, calls a service method
- Services NEVER call the ORM/database directly — go through the repository
- Repositories NEVER import from controllers or services — data flows one way
- All request bodies validated by a schema before reaching the controller handler
- DTOs are the only types crossing the HTTP boundary — no raw ORM models in responses
- Route registration: **[FILL IN — e.g. `src/app.ts` or `src/router.ts`]**
- No new top-level folders under `src/modules/` for infrastructure — use existing `src/infrastructure/`, `src/config/`

---

## SECTION C — Shared / Common Package(s)

> Path: `[SHARED_PATH]` (e.g. `packages/shared`, `libs/common`)

### What belongs here
- Types consumed by both frontend and backend (API contract types)
- Pure utility functions with no framework dependency
- Constants shared across packages
- Validation schemas that must stay in sync across the boundary

### What NEVER goes here
- UI components
- Database models or ORM entities
- Framework-specific code (React hooks, Express middleware, etc.)
- Environment-specific config

### Required folder tree

```
[SHARED_PATH]/src/
├── types/        ← shared TypeScript interfaces and types
├── utils/        ← pure utility functions
├── constants/    ← shared constants
├── schemas/      ← shared validation schemas (if applicable)
└── index.ts      ← barrel — the only way other packages import from this package
```

### Rules
- No circular imports — shared package imports from nothing in this repo
- Breaking change to a shared type requires updating ALL consuming packages in the same MR
- Always export via `index.ts` — never import a deep path from another package

---

## SECTION D — Cross-Package Import Rules

```
frontend  →  shared         ✅ allowed
backend   →  shared         ✅ allowed
frontend  →  backend        ❌ NEVER
backend   →  frontend       ❌ NEVER
shared    →  frontend       ❌ NEVER
shared    →  backend        ❌ NEVER
```

- Use workspace package names for cross-package imports: `import { X } from '@[scope]/shared'`
- Never use relative paths that cross a package boundary
- If you need a type from the other side of the boundary, it belongs in shared

---

## Prohibited actions (all packages)
- Never rename existing files or folders without an MR discussion
- Never add a new top-level folder to any package without documenting the reason in the wiki
- Never move a file between packages without updating all imports and the wiki
- Never add framework-specific code to the shared package
```

---

### `.agents/rules/security.md`

```markdown
---
paths:
  - "**/auth/**"
  - "**/middleware/**"
  - "**/config/**"
  - "**/guards/**"
---
# Security Rules

## Authentication
- The auth context / hook is the ONLY way to read the current user in frontend components
- Protected routes must be wrapped by the auth guard
- Backend: every protected endpoint passes through the auth middleware — never skip it
- JWT / session validation happens in middleware — never in route handlers or controllers

## Secrets
- No secrets in source code — all URLs and keys come from environment variables or runtime config
- Never log auth tokens, passwords, or user PII
- `.env` files are gitignored — never commit them

## Input validation
- All form data validated with a schema before it reaches an API call (frontend)
- All request bodies validated with a schema before reaching the controller (backend)
- Never pass raw query strings or URL params to API calls or DB queries without validation

## API security
- Role / permission checks done by the auth middleware — do not duplicate in controllers
- Never expose admin endpoints without an explicit role check
- All user-supplied IDs scoped to the authenticated user's accessible resources to prevent IDOR
```

---

### `.agents/rules/testing.md`

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
  - "**/*.spec.tsx"
  - "**/tests/**"
  - "**/__tests__/**"
---
# Testing

## Structure
- Co-locate tests with source files: `feature.service.test.ts` beside `feature.service.ts`
- One describe block per component, function, or class
- Test names: "should [behaviour] when [condition]"

## Frontend — what to test
- Service functions: mock HTTP client, assert correct endpoint and payload
- Hooks: use `renderHook` + `act`
- Components: user interaction and visible output — not implementation details

## Backend — what to test
- Service layer: unit test with mocked repositories
- Repository layer: integration test against a real (test) database
- Controllers: integration test using a test HTTP client — assert status codes and response shapes
- Schemas: valid + invalid inputs

## Mocking
- Mock the HTTP client at module level for frontend service tests
- Mock repositories (not services) for backend service unit tests
- Never mock the data-fetching library internals — use a real provider
- DB integration tests use a dedicated test DB, seeded fresh per test suite

## Coverage expectations
- All service functions: happy-path + one error-path test
- All schemas: valid input + at least two invalid input cases
- Frontend components: user-facing behaviour, not prop-drilling
- Backend controllers: 200, 400, 401/403 for every endpoint
```

---

## Phase 8 — Write wiki files

### `.agents/wiki/index.md`

```markdown
# [PROJECT_NAME] — Wiki Index

**Summary**: Table of contents for the [PROJECT_NAME] knowledge base.
**Last updated**: [TODAY'S DATE]

---

## Pages

| Page | Description |
|---|---|
| [[WIKI]] | Complete project reference — tech stack, packages, structure, conventions, deployment |
| [[log]] | Append-only record of all wiki operations |

## Sections inside [[WIKI]]

| Section | What it covers |
|---|---|
| Tech Stack | Libraries, versions, tooling per package |
| Repository Layout | Monorepo structure with all packages annotated |
| Frontend Package(s) | Module structure, routing, component library, state management |
| Backend Package(s) | Module structure, routing, auth, database layer |
| Shared Package(s) | Shared types, utilities, cross-package contract |
| API Layer | HTTP client, request wrappers, response types, error handling |
| Authentication | Auth flows, token storage, role access (frontend + backend) |
| Build & Deployment | Root scripts, per-package scripts, Docker, CI/CD |
| Module Conventions | Quick reference — key parts of module-structure.md |

---

## Related pages
- [[log]]
- [[WIKI]]
```

### `.agents/wiki/WIKI.md`

```markdown
# [PROJECT_NAME] — Project Wiki

**Summary**: Complete project reference.
**Sources**: Codebase analysis
**Last updated**: [TODAY'S DATE]

---

**Project Name:** [PROJECT_NAME]
**Repo:** [GITLAB_URL]
**Repo type:** [single | monorepo]
**Stack:** [TECH_STACK per package]

---

## Tech Stack

[Fill in — library name + version for every key dependency, grouped by package]

---

## Repository Layout

[Annotated monorepo tree — all packages with types and tech stacks]

---

## Frontend Package(s)

[Per package: purpose, routes, component library, state management patterns]

---

## Backend Package(s)

[Per package: purpose, routes, database, auth mechanism, key middleware]

---

## Shared Package(s)

[What lives here, who consumes it, export surface]

---

## API Layer

[HTTP client setup, request wrappers, interceptors, response types — per package if different]

---

## Authentication

[Auth flows, token/session storage, role-based access — frontend + backend]

---

## Module / Feature Conventions

[Key patterns from module-structure.md — quick reference]

---

## Build & Deployment

[Root scripts, per-package scripts, Docker, CI/CD pipeline]

---

## Related pages
- [[index]]
- [[log]]
```

### `.agents/wiki/log.md`

```markdown
# Wiki Log

**Summary**: Append-only record of all wiki operations. Never edit past entries — only append.
**Last updated**: [TODAY'S DATE]

---

## [TODAY'S DATE]

- **Init**: Set up agents agent configuration via `agents.setup.md`
- **Created**: `agents.md` — root instructions for agents Code
- **Created**: `.agents/settings.json` — permissions, hooks, safety blocks
- **Created**: `.agents/README.md` — master navigation guide
- **Created**: `.agents/rules/` — 6 rule files (architectural, code-style, api-conventions, module-structure, security, testing)
- **Created**: `.agents/wiki/` — WIKI.md, index.md, log.md
- **Created**: `.agents/taskboard/` — TASKBOARD.md, README.md
- **Created**: `.agents/skills/taskboard.skill` + `.agents/skills/taskboard/SKILL.md`
- **Created**: `.agents/skills/prompt.skill` + `.agents/skills/prompt/SKILL.md`
- **Created**: `.agents/raw/README.md`
```

---

## Phase 9 — Write taskboard files

### `.agents/taskboard/TASKBOARD.md`

```markdown
# [PROJECT_NAME] — Task Board
**GitLab board:** [GITLAB_URL]/-/boards
**Last synced:** —

---

> No active milestone yet. Run `/taskboard sync` to pull current state from GitLab.
```

### `.agents/taskboard/README.md`

```markdown
# .agents/taskboard/

Local mirror of the GitLab task board. agents keeps `TASKBOARD.md` current automatically.

## Columns

| Column | Meaning |
|---|---|
| **To Do** | Open, not started (label: `TO DO`) |
| **In Progress** | Being worked on (label: `IN PROGRESS`) |
| **Done** | Closed issues |

## When agents updates it

| Event | Trigger |
|---|---|
| New issues created | Feature planning |
| Issue completed | Complete-issue skill |
| Full sync | `/taskboard sync` |

## Start of session

Run `/taskboard sync` to pull fresh state before starting work.
```

---

## Phase 10 — Write taskboard skill (BOTH files required)

### `.agents/skills/taskboard.skill`

```yaml
name: taskboard
version: "1.0"
description: View, sync, and manage the GitLab task board — creates issues, moves columns, keeps .agents/taskboard/TASKBOARD.md current
entry: .agents/skills/taskboard/SKILL.md
argument-hint: "[sync | new <title> | move #<iid> <todo|in-progress|done> | (blank = show board)]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
```

### `.agents/skills/taskboard/SKILL.md`

```markdown
---
name: taskboard
description: View, sync, and manage the GitLab task board
allowed-tools: [Bash, Read, Write, Edit]
argument-hint: "[sync | new <title> | move #<iid> <todo|in-progress|done> | (blank = show board)]"
---

# Task Board

GitLab: [GITLAB_URL]
Local board file: `.agents/taskboard/TASKBOARD.md`

Command: **$ARGUMENTS**

---

## Route the command

- blank / `show` → **Show**
- `sync` → **Sync**
- starts with `new ` → **New Issue**
- starts with `move ` → **Move**

---

## Show

Read and print `.agents/taskboard/TASKBOARD.md`. If missing, run Sync.

---

## Sync

### 1. Get active milestone

```bash
GITLAB="[GITLAB_URL]"
PROJECT_ID="${GITLAB_PROJECT_ID}"
TOKEN="${GITLAB_TOKEN}"

curl -s "$GITLAB/api/v4/projects/$PROJECT_ID/milestones?state=active&per_page=10" \
  -H "PRIVATE-TOKEN: $TOKEN" | node -e "
const d=[];
process.stdin.on('data',c=>d.push(c));
process.stdin.on('end',()=>{
  const ms=JSON.parse(d.join(''));
  if(!ms.length){console.log('NONE');return;}
  const m=ms.sort((a,b)=>b.iid-a.iid)[0];
  console.log(m.id+'|||'+m.iid+'|||'+m.title+'|||'+m.web_url);
});"
```

If `NONE`, write `> No active milestone.` to TASKBOARD.md and stop.

### 2. Fetch issues

```bash
MILESTONE_ID=<id from step 1>

curl -s "$GITLAB/api/v4/projects/$PROJECT_ID/issues?milestone_id=$MILESTONE_ID&per_page=50&state=all" \
  -H "PRIVATE-TOKEN: $TOKEN" | node -e "
const d=[];
process.stdin.on('data',c=>d.push(c));
process.stdin.on('end',()=>{
  const issues=JSON.parse(d.join(''));
  issues.sort((a,b)=>a.iid-b.iid).forEach(i=>{
    console.log([i.iid,i.title,i.state,i.labels.join(','),i.web_url].join('|||'));
  });
});"
```

### 3. Categorize

- `state=closed` → Done
- `state=opened` + label `IN PROGRESS` → In Progress
- `state=opened` → To Do

### 4. Write TASKBOARD.md

```markdown
# [PROJECT_NAME] — Task Board
**GitLab board:** [GITLAB_URL]/-/boards
**Last synced:** YYYY-MM-DD

---

## Active Milestone: M<IID> — <Title>
**GitLab:** <url>

### To Do
| # | Title |
|---|-------|
| [#iid](url) | title |

### In Progress
| # | Title |
|---|-------|
| [#iid](url) | title |

### Done
| # | Title |
|---|-------|
| [#iid](url) | title |
```

---

## New Issue

Parse title from `new <title>`. Get active milestone id. Create issue:

```bash
cat > /tmp/new_issue.json << 'ENDJSON'
{"title":"TITLE","labels":"TO DO","milestone_id":MILESTONE_ID}
ENDJSON

curl -s -X POST "$GITLAB/api/v4/projects/$PROJECT_ID/issues" \
  -H "PRIVATE-TOKEN: $TOKEN" -H "Content-Type: application/json" \
  --data-binary @/tmp/new_issue.json | node -e "
const d=[];process.stdin.on('data',c=>d.push(c));
process.stdin.on('end',()=>{const r=JSON.parse(d.join(''));console.log('#'+r.iid+': '+r.title+'\n'+r.web_url);});"
```

Append new row to To Do. Update Last synced.

---

## Move Issue

Parse `move #<iid> <todo|in-progress|done>`.

| Target | GitLab action |
|---|---|
| `todo` | `add_labels: "TO DO"`, `remove_labels: "IN PROGRESS"` |
| `in-progress` | `add_labels: "IN PROGRESS"`, `remove_labels: "TO DO"` |
| `done` | `state_event: "close"` |

```bash
# todo / in-progress:
curl -s -X PUT "$GITLAB/api/v4/projects/$PROJECT_ID/issues/$IID" \
  -H "PRIVATE-TOKEN: $TOKEN" -H "Content-Type: application/json" \
  -d '{"add_labels":"IN PROGRESS","remove_labels":"TO DO"}'

# done:
curl -s -X PUT "$GITLAB/api/v4/projects/$PROJECT_ID/issues/$IID" \
  -H "PRIVATE-TOKEN: $TOKEN" -H "Content-Type: application/json" \
  -d '{"state_event":"close"}'
```

Then run full Sync to rebuild TASKBOARD.md.
```

---

## Phase 11 — Write prompt skill (BOTH files required)

### `.agents/skills/prompt.skill`

```yaml
name: prompt
version: "1.0"
description: Enhance and refine any draft prompt, issue description, MR body, commit message, or agent instruction for clarity, specificity, and effectiveness
entry: .agents/skills/prompt/SKILL.md
argument-hint: "<draft prompt or text to enhance>"
allowed-tools:
  - Read
  - Write
  - Edit
```

### `.agents/skills/prompt/SKILL.md`

```markdown
---
name: prompt
description: Enhance and refine any draft prompt or instruction for clarity, specificity, and effectiveness
allowed-tools: [Read, Write, Edit]
argument-hint: "<draft prompt or text to enhance>"
---

# Prompt Enhancement Skill

Input: **$ARGUMENTS**

Take the user's draft and return an enhanced version that is clearer, more specific,
and more likely to produce the desired output — whether the reader is an LLM, a developer,
or a project stakeholder.

---

## Step 1 — Classify the input type

| Type | Examples |
|---|---|
| **AI task prompt** | Prompt sent to an LLM to generate code, text, analysis |
| **Agent instruction** | Step in a SKILL.md, rule file, or agents.md |
| **Issue / ticket** | GitLab issue body with acceptance criteria |
| **MR description** | Merge request summary and context |
| **Commit message** | Git commit subject + body |
| **API / function spec** | Description of what a function or endpoint should do |
| **User story** | "As a [role], I want [goal] so that [reason]" |

---

## Step 2 — Diagnose weaknesses

Evaluate the draft against these dimensions:

| Dimension | Ask |
|---|---|
| **Specificity** | Is the desired output format stated? Are constraints explicit? |
| **Context** | Does the reader have enough background to act without guessing? |
| **Action clarity** | Is there exactly one clear thing to do, or is it ambiguous? |
| **Scope** | Is the boundary of the task defined? What is out of scope? |
| **Success criteria** | How does the reader know when they're done? |
| **Tone / register** | Does the tone match the intended reader? |
| **Brevity** | Are there filler words or over-hedging that dilute the signal? |

---

## Step 3 — Apply enhancement patterns by type

### AI task prompts
- Add a **role frame**: "You are a [role] working on [context]..."
- State **output format** explicitly: "Respond with a JSON object containing...", "Return only the updated function, no explanation"
- Add **constraints**: "Do not modify the function signature", "Use only the existing service layer"
- Add a **concrete example** if the output shape is non-obvious
- Replace vague verbs: "improve" → "refactor to remove the nested ternary and replace with an early-return guard clause"
- One prompt = one task — split compound requests

### Agent instructions (SKILL.md / rules / agents.md)
- Use imperative mood: "Read the file" not "You should read the file"
- Number sequential steps — agents follow numbered lists reliably
- Add an **exit gate** after multi-step sequences: "Stop and confirm X before proceeding"
- Name files and paths explicitly — no pronouns ("it", "the file")
- Separate decision logic from action steps

### Issue / ticket descriptions
- Lead with the **user-visible problem**, not the technical cause
- Include **acceptance criteria** as a checklist: `- [ ] Given X, when Y, then Z`
- Add an **"out of scope"** section when the issue borders a larger topic
- Link to related issues or MRs

### MR descriptions
- One-line **summary** (what changed and why)
- **How to test** section: exact steps a reviewer should follow
- **Screenshots / before-after** note if UI changed
- List **breaking changes** or **migration steps** required

### Commit messages
- Subject: imperative mood, ≤72 chars, no period: "Add pagination to user list endpoint"
- Body: explain **why**, not **what** (the diff shows what)
- Reference: `Closes #123` or `Refs #123`

---

## Step 4 — Produce output

Return:

1. **Enhanced version** — the rewritten prompt or text, ready to use as-is
2. **Changes summary** — 2–5 bullet points explaining what was changed and why
3. **Variants (optional)** — if there are two reasonable interpretations of intent, provide a second version with a note explaining the different assumption

---

## Example

### Input
```
make the user list faster
```

### Enhanced
```
Investigate and resolve the performance bottleneck in the user list endpoint (`GET /api/users`).

Context: The endpoint currently takes >2s on datasets of 1,000+ users. Profiling suggests
the N+1 query on the `roles` relationship is the primary cause.

Acceptance criteria:
- [ ] Response time for 1,000 users is <300ms in staging
- [ ] No N+1 queries — confirmed via query log
- [ ] Existing pagination behaviour is unchanged
- [ ] All user-related tests pass

Out of scope: UI changes, frontend caching, unrelated endpoint optimisations.
```

### Changes
- Added specific endpoint path and current symptom (>2s, 1,000+ users)
- Added likely cause from profiling to focus the investigation
- Replaced vague "faster" with a measurable success criterion (<300ms)
- Added acceptance criteria checklist and explicit out-of-scope
```

---

## Phase 12 — Write `.agents/raw/README.md`

```markdown
# .agents/raw/ — Immutable Source Documents

Drop files here when you want agents to ingest them into the wiki.

**Rules:**
- Never modify files once added
- Never delete files from this folder
- To add knowledge: drop the file here, then ask agents to ingest it

**Ingest workflow:**
1. Read the full source document
2. Discuss key takeaways
3. Create or update wiki pages in `.agents/wiki/`
4. Update `.agents/wiki/index.md`
5. Append to `.agents/wiki/log.md`
```

---

## Phase 13 — Verify everything is in place

```bash
# Check all directories exist
ls .agents/
ls .agents/rules/
ls .agents/wiki/
ls .agents/taskboard/
ls .agents/skills/
ls .agents/skills/taskboard/
ls .agents/skills/prompt/

# Check BOTH skill files exist for each skill
ls .agents/skills/*.skill
ls .agents/skills/taskboard/SKILL.md
ls .agents/skills/prompt/SKILL.md

# Check agents.md is at root
ls agents.md

# Check settings.json is valid JSON
node -e "JSON.parse(require('fs').readFileSync('.agents/settings.json','utf8')); console.log('settings.json OK')"

# Show git status
git status --short
```

Report what was created and flag anything missing.

---

## Phase 14 — Fill in module-structure.md (mandatory)

After exploring the project in Phases 1–2, return to `.agents/rules/module-structure.md` and replace every `[FILL IN DURING SETUP]` placeholder with the **actual** structure from the real codebase.

For each package:
1. Find 2–3 existing feature modules
2. Map their exact folder tree
3. Identify naming patterns (file names, export names, route paths)
4. Document the layer responsibility split (controller → service → repository, component → hook → service)
5. List anything that must never be changed (legacy files, frozen contracts)
6. Write the cross-package import rules based on the actual workspace alias config in `package.json` / `tsconfig.json`

If the project is a **single-package repo**, simplify to Section A only, delete Sections B–D, and document the single package's structure in detail.

---

## Done

Tell the user:

```
✅ agents agent setup complete for [PROJECT_NAME].

Created:
  agents.md
  .agents/settings.json
  .agents/README.md
  .agents/rules/                          (6 files)
  .agents/wiki/                           (3 files)
  .agents/taskboard/                      (2 files)
  .agents/skills/taskboard.skill          ← discovery manifest
  .agents/skills/taskboard/SKILL.md       ← implementation
  .agents/skills/prompt.skill             ← discovery manifest
  .agents/skills/prompt/SKILL.md          ← implementation
  .agents/raw/README.md

Next steps:
  1. Set environment variables: GITLAB_TOKEN and GITLAB_PROJECT_ID
  2. Run /taskboard sync to pull current board state from GitLab
  3. Review .agents/rules/module-structure.md — complete Sections A, B, C, D with real paths
  4. Drop existing docs into .agents/raw/ and ask agents to ingest them into the wiki
  5. Try /prompt <your next issue description> to test the prompt skill
```
