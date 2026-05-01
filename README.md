# Agentic Company Playbook

A practical, opinionated guide for building an AI-native company with autonomous agents from day one.

This playbook is the result of hands-on experience building with agents, synthesizing emerging industry research, and iterating on what actually works at a small-company scale. It is intentionally generic — fork it, adapt it, and make it your own.

---

## Who this is for

- Founders starting a new company who want agents as core team members from the start
- Engineering teams transitioning to an agentic workflow
- Anyone who wants a concrete, actionable blueprint rather than theory

## What this covers

| Section | What you'll learn |
|---|---|
| [1. Philosophy](docs/01-philosophy.md) | Why agentic-first changes everything |
| [2. Agent Team Structure](docs/02-agent-team-structure.md) | Roles, identities, and trust boundaries |
| [3. Development Workflow](docs/03-development-workflow.md) | CCPM, issue-driven development, vertical slices |
| [4. API-First Cycle](docs/04-api-first-cycle.md) | OpenAPI → validation → mocks → implementation |
| [5. Frontend Workflow](docs/05-frontend-workflow.md) | How the frontend cycle differs from backend |
| [6. Security Model](docs/06-security-model.md) | Least-privilege, identity isolation, email security |
| [7. Infrastructure & Toolchain](docs/07-toolchain.md) | The exact tools and why |
| [8. Human Touchpoints](docs/08-human-touchpoints.md) | Where humans must stay in the loop |
| [9. Getting Started](docs/09-getting-started.md) | Day-one checklist for a new company |

---

## Core principles

**1. GitHub is the single source of truth.**
All work — ideas, decisions, specs, code, and reviews — lives in GitHub. No tool, Slack thread, or Notion doc supersedes what's in the repo.

**2. Agents are team members, not tools.**
Each agent has a persistent identity, a defined role, and a bounded scope of authority. They are assigned issues, open PRs, and participate in reviews just like humans.

**3. Autonomy is earned, not assumed.**
Agents operate autonomously within well-defined boundaries. The boundaries expand as trust is established. Human review gates protect shared systems.

**4. API contract before implementation.**
For any backend feature, the OpenAPI spec is written, reviewed, and validated before a single line of implementation code is written. The spec is the agreement.

**5. Vertical slices, not horizontal layers.**
Every issue delivers a complete, working slice of functionality — from API spec to passing test. Never open a "backend only" ticket followed by a "frontend only" ticket for the same feature.

---

## Prior art and inspiration

This playbook synthesizes and builds on:

- [Microsoft: An AI-led SDLC](https://devblogs.microsoft.com/engineering-at-microsoft/) — end-to-end agentic software development lifecycle
- [OpenAI Codex: Building an AI-Native Engineering Team](https://openai.com/codex) — agent-driven engineering culture
- [GitHub Awesome Agentic Workflow](https://github.com/topics/agentic-workflow) — community patterns
- [Matt Pocock's skills methodology](https://www.youtube.com/watch?v=-QFHIoCo-Ko) — PRD-to-issues workflow
- [automazeio/ccpm](https://github.com/automazeio/ccpm) — Claude Code PM skill
- [APIAddicts toolchain](https://github.com/apiaddicts) — API quality and governance
- Bain & Company "State of the Art of Agentic AI Transformation" (2025)
- IDC "Agentic AI: A Playbook for Corporate Innovation" (2025)

---

## License

MIT — fork freely, adapt for your context, contribute back what you learn.
