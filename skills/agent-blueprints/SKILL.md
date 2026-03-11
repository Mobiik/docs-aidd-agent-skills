---
name: agent-blueprints
description: >
  Reference guide for the AI development framework architecture. Load when:
  creating or modifying agent configurations, setting up project framework,
  bootstrapping agents/skills/rules, or discussing agent architecture design.
  Contains templates, conventions, and the complete specification for the
  5-agent + 12-skill + 3-rule framework structure.
---

# AI Development Framework — Reference Specification

This skill contains the blueprint for generating agent frameworks.
Consumed by any agent that needs to generate or update framework configurations.

## Architecture

```
FRAMEWORK = 5 Agents + 12 Skills + AGENTS.md (rules)

Agents  = AUTONOMY + OWN CONTEXT + TOOL RESTRICTIONS
Skills  = KNOWLEDGE + PROCEDURES + LAZY-LOADED
Rules   = ALWAYS-ON + SHORT + IN AGENTS.md
```

## Decision Matrix

```
Needs own context window?          → Agent
Makes autonomous decisions?        → Agent
Is domain knowledge/procedures?    → Skill
Must ALWAYS be in context?         → Rule (in AGENTS.md)
```

## OUTPUT: Always .claude/

```
project-root/
├── .claude/
│   ├── agents/
│   │   ├── architect.md
│   │   ├── coder.md
│   │   ├── tester.md
│   │   ├── reviewer.md
│   │   └── explorer.md
│   └── skills/
│       ├── architecture/SKILL.md
│       ├── frontend/SKILL.md
│       ├── backend-api/SKILL.md
│       ├── database/SKILL.md
│       ├── testing-patterns/SKILL.md
│       ├── security/SKILL.md
│       ├── git-workflow/SKILL.md
│       ├── docker-deploy/SKILL.md
│       ├── performance/SKILL.md
│       ├── error-handling/SKILL.md
│       ├── dependencies/SKILL.md
│       └── debugging/SKILL.md
└── AGENTS.md
```

---

## AGENT TEMPLATES

### Template: Architect

```markdown
---
name: architect
description: >
  Invoke when: user requests a new feature, structural refactor,
  system design, or asks "how should I build X". Also when task
  involves 3+ files, new {{ENDPOINT_TERM}}, or cross-module changes.
  Produces implementation plans, never writes code.
tools:
  - Read
  - Grep
  - Glob
---

You are a senior software architect for a {{FRAMEWORK}} project.
Your job is to PLAN, never to code.

## Process
1. Understand the requirement (ask if unclear)
2. Explore existing codebase patterns
3. Produce a plan: files, interfaces, order, risks
4. Wait for human approval

## Project Context
- Structure: {{PROJECT_STRUCTURE_SUMMARY}}
- Patterns: {{KEY_PATTERNS}}

## Rules
- NEVER write implementation code
- ALWAYS explore existing patterns first
- Keep plans under 50 lines
- Flag if task is simple enough to skip planning
```

**Placeholders:** `{{FRAMEWORK}}`, `{{ENDPOINT_TERM}}`, `{{PROJECT_STRUCTURE_SUMMARY}}`, `{{KEY_PATTERNS}}`

---

### Template: Coder

```markdown
---
name: coder
description: >
  Default implementation agent. Invoke for: writing new code,
  implementing features from plans, fixing bugs, refactoring.
  Works with {{LANGUAGE}} and {{FRAMEWORK}}.
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

You are a senior {{STACK_DESCRIPTION}} developer.

## Process
1. If plan exists from Architect, follow it
2. If no plan, assess complexity:
   - Complex (3+ files): suggest Architect first
   - Simple: proceed directly
3. Read related files to match patterns
4. Implement incrementally
5. Run {{LINT_COMMAND}} after changes
6. Run {{TEST_COMMAND}} if tests exist

## Project Stack
{{STACK_DETAILS}}

## Rules
- Match existing code style exactly
- Read 2-3 related files before writing
- Never install dependencies without stating why
```

**Placeholders:** `{{LANGUAGE}}`, `{{FRAMEWORK}}`, `{{STACK_DESCRIPTION}}`, `{{LINT_COMMAND}}`, `{{TEST_COMMAND}}`, `{{STACK_DETAILS}}`

---

### Template: Tester

```markdown
---
name: tester
description: >
  Invoke when: code was just written/modified, user asks for tests,
  TDD workflow, or needs test coverage. Uses {{TEST_FRAMEWORK}}.
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

You are a QA engineer using {{TEST_FRAMEWORK}}.

## Process
1. Read existing tests in {{TEST_DIR}} to understand patterns
2. Identify what needs testing
3. Write tests matching project style
4. Run: {{TEST_COMMAND}}
5. Fix failures, re-run until green

## Rules
- Match {{TEST_FRAMEWORK}} style exactly
- Test behavior, not implementation
- Include: happy path, edge cases, error cases
```

**Placeholders:** `{{TEST_FRAMEWORK}}`, `{{TEST_DIR}}`, `{{TEST_COMMAND}}`

---

### Template: Reviewer

Read-only agent. Key config:
- `tools: Read, Grep, Glob` (NEVER modifies files)
- Triggers: "review", before commits, after feature completion
- Checklist: Correctness → Security → Patterns → Simplicity → Types
- Severity: 🔴 Must fix / 🟡 Should fix / 🟢 Suggestion
- Max 10 items. Specific: file paths, line numbers, concrete fixes.

---

### Template: Explorer

Read-only, fast, cheap. Key config:
- `tools: Read, Grep, Glob`
- Triggers: understanding codebase, finding definitions, tracing data flow
- Three levels: quick (find item), medium (understand module), thorough (map architecture)
- Include `{{PROJECT_MAP}}` with actual folder summary
- NEVER suggests changes — only reports what exists

---

## SKILL TEMPLATES

### Structure

```
.claude/skills/
└── skill-name/
    ├── SKILL.md          # Main file (< 500 lines)
    └── references/       # Optional detail files (loaded on demand)
```

### The 12 Core Skills

#### 1. architecture

```yaml
name: architecture-patterns
description: >
  Load when: designing features, modules, refactoring structure,
  file organization, {{FRAMEWORK}} app structure.
```
Content: folder convention, approved patterns, module boundaries,
decision checklist, anti-patterns.

#### 2. frontend

```yaml
name: frontend-conventions
description: >
  Load when: components, styling, client state, {{UI_FRAMEWORK}},
  forms, routing. Keywords: component, {{CSS_APPROACH}}, hook, UI.
```
Content: component structure, styling rules, state management,
forms, accessibility. **Skip if pure backend project.**

#### 3. backend-api

```yaml
name: backend-api
description: >
  Load when: {{ENDPOINT_TERM}}, HTTP handlers, middleware, auth,
  validation. Keywords: endpoint, route, API, {{BACKEND_FRAMEWORK}}.
```
Content: endpoint naming, response format, auth, validation, middleware.

#### 4. database

```yaml
name: database
description: >
  Load when: migrations, queries, schema, {{ORM}} models.
  Keywords: migration, schema, query, {{DB_TYPE}}, {{ORM}}.
```
Content: ORM conventions, migrations, naming, queries, indexing.

#### 5. testing-patterns

```yaml
name: testing-patterns
description: >
  Load when: tests, coverage, TDD. Keywords: test, {{TEST_FRAMEWORK}},
  mock, fixture, unit, integration, e2e.
```
Content: framework, file location, naming, mocks, test levels.

#### 6. security

```yaml
name: security
description: >
  Load when: auth, sanitization, CORS, headers, secrets, permissions.
```
Content: OWASP adapted to stack, sanitization, secrets, auth checklist.

#### 7. git-workflow

```yaml
name: git-workflow
description: >
  Load when: branching, commits, PRs, merge, release.
```
Content: branch naming, commit format, PR template, merge rules.

#### 8. docker-deploy

```yaml
name: docker-deploy
description: >
  Load when: containers, CI/CD, environment config, deploy.
```
Content: Dockerfile patterns, compose, env management.
**Skip if no containerization.**

#### 9. performance

```yaml
name: performance
description: >
  Load when: optimization, caching, lazy loading, bundle size, N+1.
```
Content: frontend + backend perf checklists, caching, profiling.

#### 10. error-handling

```yaml
name: error-handling
description: >
  Load when: try/catch, error boundaries, logging, error codes.
```
Content: error pattern, logging conventions, error types, retry.

#### 11. dependencies

```yaml
name: dependencies
description: >
  Load when: installing packages, evaluating libraries, updating deps.
```
Content: evaluation criteria, approved list, approval process.

#### 12. debugging

```yaml
name: debugging
description: >
  Load when: investigating bugs, runtime errors, stack traces.
```
Content: debug process, tools, common error patterns for this stack.

---

## AGENTS.md TEMPLATE (Rules)

Single file at project root, under 80 lines total. Three sections.
AGENTS.md is the universal standard (Linux Foundation) — read by VS Code,
Cursor, Claude Code, Codex, Gemini, Windsurf, and 20+ other tools.

```markdown
# Project: {{PROJECT_NAME}}

## Stack
- Language: {{LANGUAGE}} {{VERSION}}
- Frontend: {{FRONTEND}}
- Backend: {{BACKEND}}
- Database: {{DB}}
- Testing: {{TEST}}
- Linting: {{LINTER}}

## Code Style
- Formatting: {{FORMATTER}}
- Naming: {{NAMING}}
- Imports: {{IMPORTS}}

## Critical Rules
- NEVER install dependencies without asking
- ALWAYS read existing code before writing new
- NEVER delete tests
- {{PROJECT_SPECIFIC_RULES}}

## Agent Behavior
- Research before coding: read 2-3 related files minimum
- Plan before implementing if task touches 3+ files
- Verify after implementing: run {{VERIFY_COMMAND}}
- Ask before assuming if requirements are ambiguous
- Be concise. Show diffs, not full files.

## Guardrails
- Do not touch: {{PROTECTED_PATHS}}
- Before commit: {{PRE_COMMIT_CHECKS}}
```

---

## STACK REFERENCE

For detailed per-language configuration (lint commands, test commands,
naming, frameworks), see: `references/stacks.md`

---

## MINIMAL SETUP (< 10 files)

For tiny projects, generate only:
- 2 agents: coder + explorer
- 3 skills: testing + git-workflow + primary stack skill
- 1 AGENTS.md
- Total: ~6 files

---

## UPDATE PROTOCOL

1. Read all existing .claude/ files
2. Diff against current project state
3. Only modify what changed
4. Never overwrite user customizations without asking
5. Show summary before applying
