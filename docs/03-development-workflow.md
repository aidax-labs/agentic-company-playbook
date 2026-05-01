# 3. Development Workflow

## The source-of-truth stack

```
Idea (human) 
  → PRD in /docs/prds/ (agent-generated, human-reviewed)
  → GitHub Epic Issue (agent-generated from PRD)
  → GitHub Sub-issues (agent-generated, one per vertical slice)
  → Feature branches + PRs (agent-implemented)
  → Merged to main (human or senior-agent approved)
```

Every item in this chain is traceable. You can always answer: "Why does this code exist?" by following the chain from commit → PR → issue → epic → PRD.

---

## Step 1 — From idea to PRD

The human (or coordinator) describes a feature or product need in a message to their AI assistant. The assistant uses the `/to-prd` skill to produce a structured Product Requirements Document.

**PRD structure:**
```markdown
# PRD: [Feature Name]

## Problem
[What problem this solves and for whom]

## Goals
[Measurable outcomes]

## Non-goals
[Explicit out of scope]

## User stories
[As a [user], I want [action] so that [outcome]]

## Domain model changes
[New entities, modified relationships — if any]

## API changes (high level)
[New or modified endpoints — links to OpenAPI spec if already written]

## Open questions
[Things that need a decision before implementation starts]
```

PRDs live in `/docs/prds/YYYY-MM-feature-name.md` in the main repo. They are versioned with the code.

**The human reviews the PRD** before any further work begins. This is the cheapest moment to catch misunderstandings.

---

## Step 2 — From PRD to issues

Once the PRD is approved (merged), the agent uses the `/to-issues` skill to break it into GitHub Issues.

Each issue is a **vertical slice**: it delivers a complete, testable piece of functionality from API to UI (or from schema to API for backend-only work). Never create "backend ticket" + "frontend ticket" for the same feature. One ticket, one slice.

**Issue template:**
```markdown
## Context
[One-paragraph summary of what this implements and why]

## Acceptance criteria
- [ ] [Specific, testable condition]
- [ ] [Specific, testable condition]

## Technical notes
[Links to PRD, ADR, relevant code, or spec section]

## Out of scope
[What this ticket does NOT include]
```

Issues are created in the repo, labeled by domain (`backend`, `frontend`, `spec`, `infra`), and assigned to the appropriate agent.

---

## Step 3 — Vertical slice implementation

An agent picks up an assigned issue and follows this sequence:

1. **Read** the issue, the PRD, and any linked ADRs
2. **Check** if there are upstream dependencies (spec must exist for backend work; spec + mock must exist for frontend work)
3. **Create** a feature branch: `feat/issue-123-short-description`
4. **Implement** following the acceptance criteria
5. **Write tests** that verify each acceptance criterion
6. **Open a PR** referencing the issue (`Closes #123`)
7. **Request review** from the appropriate human or senior agent

The agent does not merge their own PR (unless explicitly granted L3 trust).

---

## Step 4 — PR review

The PR reviewer checks:
- Does the implementation match the acceptance criteria?
- Are tests meaningful (not just happy path)?
- Does the change introduce any security issues?
- Is the diff scope-contained (no unrelated changes)?

If the answer to all is yes, merge. If not, request changes.

---

## Docs as code

All significant technical decisions are recorded as **Architecture Decision Records (ADRs)** in `/docs/adr/`.

```
/docs/adr/
  0001-use-postgres-as-primary-database.md
  0002-api-first-development-cycle.md
  0003-agent-identity-model.md
```

**ADR format:**
```markdown
# ADR-NNNN: [Decision title]

**Status:** Accepted | Superseded by ADR-XXXX | Deprecated

## Context
[What situation forced this decision]

## Decision
[What was decided]

## Consequences
[What becomes easier, what becomes harder]
```

ADRs are immutable once merged. If a decision changes, write a new ADR that supersedes the old one. Never edit a merged ADR.

---

## CCPM integration

[automazeio/ccpm](https://github.com/automazeio/ccpm) provides the `/pm` Claude Code skill that automates the PRD→Epic→Issues pipeline described above.

Install it as a Claude Code skill and use it to:
- Generate structured PRDs from a feature description
- Break PRDs into GitHub Issues with proper templates
- Create parallel worktrees for independent issues
- Track progress across a feature epic

CCPM treats GitHub Issues as the canonical work queue — no separate project management tool needed.

---

## Parallel work with worktrees

Git worktrees let multiple agents work on the same repo simultaneously without branch conflicts. When an epic has multiple independent issues:

1. Each issue gets its own worktree: `git worktree add ../repo-issue-123 feat/issue-123`
2. Agents work in parallel
3. PRs are opened independently and merged in dependency order
4. Worktrees are deleted after merging: `git worktree remove ../repo-issue-123`

This is the primary mechanism for parallelizing agent work. Do not have multiple agents work in the same worktree.
