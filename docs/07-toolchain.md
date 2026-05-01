# 7. Infrastructure & Toolchain

## Design principle

The toolchain exists to serve the workflow, not the other way around. Every tool in this list earns its place by solving a specific problem in the agentic development cycle. When a tool stops earning its place, remove it.

---

## Core tools

### Claude Code (agent runtime)
The execution environment for agents. Provides:
- File system access (read, write, edit)
- Bash execution
- Web search and fetch
- GitHub CLI integration
- Skill system (`CLAUDE.md` instructions + registered skills)

[claude.ai/code](https://claude.ai/code) — CLI, desktop app, IDE extensions

**Configuration:** Each agent's machine has Claude Code installed and configured with their identity (API key scoped to their role).

---

### GitHub (source of truth)
Everything that matters lives in GitHub:
- Source code
- OpenAPI specs
- PRDs and ADRs in `/docs/`
- Issues as the work queue
- PRs as the review mechanism
- GitHub Actions for CI/CD

No external project management tool. No Jira, no Linear, no Notion for work tracking. GitHub Issues is sufficient and keeps everything in one place.

**Recommended GitHub Actions:**
- Spectral lint on spec changes
- Contract tests on implementation changes
- Type checking and unit tests on all PRs
- Deployment on merge to main (with manual approval gate for production)

---

### CCPM — Claude Code PM skill
[automazeio/ccpm](https://github.com/automazeio/ccpm)

The `/pm` skill that automates the product management loop:
- `/pm prd` — generate a PRD from a feature description
- `/pm issues` — break a PRD into GitHub Issues
- `/pm epic` — create an epic and link sub-issues
- `/pm worktrees` — set up parallel worktrees for independent issues

Install once in your Claude Code skills directory, available to all agents.

---

### Matt Pocock skills
Methodology inspired by [Matt Pocock's agent workflow](https://www.youtube.com/watch?v=-QFHIoCo-Ko):

- `/to-prd` — converts a feature description into a structured PRD
- `/to-issues` — converts a PRD into scoped GitHub Issues
- `/tdd` — test-driven development workflow (write failing test, implement, verify)
- `/improve-codebase-architecture` — architectural improvement suggestions with ADR output
- `/grill-with-docs` — asks probing questions about a design using relevant documentation

`CONTEXT.md` is a key pattern from this methodology: a short (100-line max) file in the repo root that gives agents everything they need to orient to the codebase without reading every file.

**CONTEXT.md template:**
```markdown
# [Project Name] — Context for agents

## What this is
[One paragraph]

## Tech stack
- Language: 
- Framework:
- Database:
- Deployment:

## Key directories
- `src/` — [description]
- `docs/` — [PRDs, ADRs, domain models]
- `openapi/` — [API specs]

## Running locally
[Exact commands]

## Running tests
[Exact commands]

## Key conventions
- [Naming convention]
- [File organization convention]
- [Error handling convention]

## Do not touch
- [Files or directories agents should not modify]
```

---

### Spectral (API spec linting)
[stoplight.io/open-source/spectral](https://stoplight.io/open-source/spectral)

Validates OpenAPI specs against configurable rules. Runs in CI on every spec change.

```bash
npm install -g @stoplight/spectral-cli
spectral lint openapi/openapi.yaml --ruleset .spectral.yaml
```

Start with `spectral:oas` and add [APIAddicts rules](https://github.com/apiaddicts/spectral-rules) as your API standards mature.

---

### Microcks (mock server + contract testing)
[microcks.io](https://microcks.io)

Generates live mock servers from OpenAPI specs. Runs contract tests against implementations.

```bash
# Quick start with Docker
docker run -p 8080:8080 quay.io/microcks/microcks-uber:latest
```

Import your OpenAPI spec and Microcks immediately serves mock responses based on spec examples. The frontend agent points to this URL from day one.

---

### MSW — Mock Service Worker (frontend testing)
[mswjs.io](https://mswjs.io)

Intercepts HTTP requests in tests and in Storybook. Generated from OpenAPI spec examples.

Used by the frontend agent for component tests — no running backend needed.

---

### APIAddicts ecosystem (optional, for API governance at scale)
[github.com/apiaddicts](https://github.com/apiaddicts)

Additional tooling for mature API programs:
- `sonar-openapi` — SonarQube plugin for API quality metrics
- `openapi2postman` — Postman collection generation
- `o2e` — generate code, docs, mocks from spec
- `visual-mapper` — visualize domain relationships from OpenAPI

Start with Spectral. Add these tools as your API portfolio grows.

---

### Orchestration layer (OpenClaw or equivalent)
An orchestration layer that can:
- Spawn agent sessions from triggers (GitHub webhook, schedule, message)
- Route messages between the human coordinator and agents
- Schedule recurring tasks (dependency updates, security scans, report generation)
- Provide a conversational interface to the coordinator

This is the "nervous system" that connects the human to the agent team. The specific tool matters less than having one — without it, the coordinator manages agents manually, which defeats the purpose.

---

## What's intentionally not in this list

| Tool | Why excluded |
|---|---|
| Jira / Linear / Notion for task tracking | GitHub Issues is sufficient; fragmentation is costly |
| ChatGPT / other AI for code generation | One AI runtime per agent avoids context fragmentation |
| Slack for agent communication | Text-based; not auditable as GitHub; agents can't participate meaningfully |
| Docker for local development | Worthwhile for production, optional for agent-driven development flows |
| OpenSpec | Valuable for human-centered API review; conflicts with autonomous operation goal |

Every excluded tool is a judgment call, not a dogma. If your context makes one of these the right choice, use it.

---

## Toolchain setup checklist

- [ ] GitHub org created, repo structure defined
- [ ] Claude Code installed on each agent machine
- [ ] CCPM skill installed
- [ ] Spectral installed and `.spectral.yaml` configured
- [ ] Microcks running (Docker or hosted)
- [ ] MSW set up in frontend repo
- [ ] CONTEXT.md written for each repo
- [ ] `CLAUDE.md` written for each agent role
- [ ] GitHub Actions: Spectral lint, tests, type check, deploy
- [ ] Secrets manager configured with agent credentials
- [ ] Agent GitHub accounts created with fine-grained tokens
- [ ] Agent email accounts created with external sender restrictions
