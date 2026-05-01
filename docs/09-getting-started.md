# 9. Getting Started — Day-One Checklist

## Before you start

This checklist assumes:
- You have a GitHub organization (or are creating one)
- You have at least one human coordinator (probably you)
- You have Claude Code access
- You have at least one machine available for agents (can be your primary machine initially)

Work through this in order. Each section builds on the previous one.

---

## Phase 1 — Foundation (Day 1)

### GitHub setup
- [ ] Create GitHub organization
- [ ] Create the main product repo (e.g., `yourcompany/app`)
- [ ] Create an `agentic-company-playbook` (or equivalent) repo — your company's adaptation of this guide
- [ ] Enable branch protection on `main`: require PR, require CI, no direct push
- [ ] Enable GitHub Actions

### Repository structure
Set up your main repo with this structure from the start:
```
/
├── CONTEXT.md              ← Agent orientation file
├── CLAUDE.md               ← Coordinator-level agent instructions
├── docs/
│   ├── adr/                ← Architecture Decision Records
│   ├── prds/               ← Product Requirements Documents
│   └── domain/             ← Domain model diagrams
├── openapi/
│   ├── openapi.yaml        ← Root spec file
│   └── paths/              ← Spec split by resource
├── src/                    ← Source code
└── .github/
    └── workflows/          ← CI/CD
```

### Write CONTEXT.md
Write it before you write any code. Include:
- What the product is (one paragraph)
- Tech stack
- Key directories
- How to run locally
- How to run tests
- Key conventions

### Write your first ADR
Document the decisions you're making right now. Good first ADRs:
- `0001-tech-stack.md` — language, framework, database
- `0002-api-first-development.md` — why you're committing to this workflow
- `0003-agent-team-model.md` — your specific agent roles and trust levels

---

## Phase 2 — Agent identities (Day 1–2)

### Email accounts
- [ ] Create agent email accounts (`agentname@yourcompany.com`)
- [ ] Configure inbound email restriction (internal senders only)
- [ ] Create a distribution group for agent accounts for easy bulk policy

### GitHub accounts
- [ ] Create one GitHub account per agent role
- [ ] Invite each account to your GitHub org with appropriate base permissions
- [ ] Create fine-grained personal access tokens for each agent (scoped to specific repos)
- [ ] Store tokens in a secrets manager

### Machine setup
- [ ] Install Claude Code on each agent machine
- [ ] Configure each machine with the agent's Claude Code API key
- [ ] Enroll in MDM if you have one (recommended for company-owned devices)
- [ ] Configure disk encryption on all agent machines

---

## Phase 3 — Toolchain (Day 2–3)

### Install skills
- [ ] Install CCPM (`/pm` skill) — [automazeio/ccpm](https://github.com/automazeio/ccpm)
- [ ] Set up your team's custom skills in `.claude/commands/`

### API toolchain
- [ ] Install Spectral: `npm install -g @stoplight/spectral-cli`
- [ ] Write `.spectral.yaml` with `spectral:oas` + APIAddicts rules
- [ ] Set up Microcks (Docker): `docker run -p 8080:8080 quay.io/microcks/microcks-uber:latest`
- [ ] Add Spectral lint to GitHub Actions (runs on every spec change)

### Write CLAUDE.md files
Write one for each agent role. At minimum:
- [ ] Coordinator CLAUDE.md
- [ ] Backend agent CLAUDE.md
- [ ] Frontend agent CLAUDE.md

Template in [Agent Team Structure](02-agent-team-structure.md).

---

## Phase 4 — First feature (Day 3–5)

Run the full cycle once, manually, so you understand every step before automating it.

1. **Write a small feature idea** — something that takes ~2 hours to implement
2. **Generate the PRD** using `/to-prd` or `/pm prd`
3. **Review and merge the PRD**
4. **Generate issues** using `/to-issues` or `/pm issues`
5. **Generate the OpenAPI spec** for the first issue
6. **Run Spectral** — fix any lint errors
7. **Review and merge the spec PR**
8. **Start the Microcks mock server** with the spec
9. **Implement the backend** (you or the backend agent)
10. **Implement the frontend** (you or the frontend agent, against the mock)
11. **Review and merge both PRs**
12. **Document what you learned** — update CONTEXT.md, ADRs, or this playbook

The first cycle will take longer than 30 minutes. The second will be faster. By the tenth, it should be routine.

---

## Phase 5 — Automation (Week 2+)

Once the manual cycle feels natural, automate the repeatable parts:

- [ ] GitHub webhook → trigger agent on new issue assignment
- [ ] Spectral lint in CI (already done in Phase 3)
- [ ] Contract tests in CI on implementation PRs
- [ ] Scheduled dependency updates (weekly)
- [ ] Scheduled security scan (weekly)

---

## Common mistakes on day one

**Giving agents too much access upfront.**
Start restrictive. It's easy to expand access; it's hard to recover from an agent deleting things it shouldn't.

**Skipping CONTEXT.md.**
Agents will spend the first part of every task exploring the repo to orient themselves. CONTEXT.md cuts this down to seconds. Write it first.

**Writing issues that are too large.**
"Build the auth system" is not an issue. "Implement POST /v1/auth/register with email+password validation and returning a JWT" is an issue. Scope down until you're uncomfortable with how small it is — that's usually the right size.

**Merging without review because "it's just an agent's PR."**
The review is not for the agent's benefit. It's for yours. You are accountable for what goes into main, regardless of who wrote it.

**Not writing ADRs.**
Six months from now, you will not remember why you made the decisions you made today. Write the ADRs. Future you (and future agents) will thank you.

---

## Getting help

- Read the docs in this repo before asking an agent — the answer is probably here
- If something is wrong with the workflow, update the docs before fixing the symptom
- Open an issue in this repo if you find a gap in the playbook
- Share what you learn — this guide improves when practitioners contribute back
