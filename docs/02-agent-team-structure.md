# 2. Agent Team Structure

## Core concept: agents as named team members

Each agent has:
- A **persistent identity** (name, email address, machine)
- A **defined role** (what they own and what they don't)
- A **trust level** (what they can do autonomously vs. what requires review)
- A **CLAUDE.md** (the agent's operating instructions, scoped to their domain)

This is not metaphorical. Agents have real email addresses, real GitHub accounts, and real access credentials — scoped precisely to what their role requires.

---

## Reference team structure

This is a starting point. Adapt the roles and names to your domain and team size.

### Coordinator (human or senior agent)
- Assigns issues to agents
- Reviews PRs that span multiple domains
- Resolves cross-agent conflicts
- Owns the product roadmap

### Backend Agent (e.g., "Axel")
- Implements API routes and business logic
- Writes and runs database migrations
- Owns the OpenAPI spec updates
- Does not merge to main without review

### Frontend Agent (e.g., "Vera")
- Implements UI components and pages
- Consumes the OpenAPI spec (uses mock server during development)
- Writes component and integration tests
- Does not deploy without review

### Spec/QA Agent (e.g., "Leo")
- Validates OpenAPI specs against style rules (Spectral)
- Generates mock servers (Microcks)
- Writes contract tests
- Flags spec inconsistencies before implementation begins

### Research Agent (e.g., "Kai")
- Web searches, competitive analysis, technology evaluation
- Produces structured research reports as GitHub Issues or `/docs` files
- Read-only access to production systems

---

## Identity setup

### Email addresses
Each agent gets a dedicated email address under your company domain. Use your email provider's admin console to:
- Create `axel@yourcompany.com`, `vera@yourcompany.com`, etc.
- **Restrict inbound email** to internal senders only — agents should not receive external email (phishing vector)
- Allow outbound email only for notifications (GitHub, CI, etc.)

### GitHub accounts
Each agent gets a dedicated GitHub account. Best practice:
- Username format: `yourcompany-axel`, `yourcompany-vera`, etc.
- Invite each agent account to the GitHub organization
- Use **fine-grained personal access tokens** scoped to specific repos and permissions
- Never use org-wide tokens for agents

### Machine assignment
If running agents on physical or virtual machines:
- Assign one primary machine per agent role
- Use your MDM solution (e.g., Jamf, Intune) to enroll and manage agent machines
- Apply baseline security policies: disk encryption, screen lock, no direct internet browsing

### Token and credential management
- Store agent tokens in a secrets manager (e.g., 1Password, Vault, GitHub Secrets)
- Rotate tokens on a schedule (quarterly minimum)
- Audit token usage logs monthly
- Never share tokens across agent identities

---

## CLAUDE.md — the agent's operating contract

Each agent's working directory contains a `CLAUDE.md` file that defines:

```markdown
# Role
You are [Name], the [Role] agent for [Company].

# Responsibilities
- [What this agent owns]
- [What this agent produces]

# Constraints
- Do not modify files outside [path scope]
- Do not push directly to main
- Do not access production credentials
- Escalate to the Coordinator for [specific situation types]

# Workflow
1. Read the assigned issue
2. [Step-by-step process for this role]
3. Open a PR and request review from [reviewer]

# Tools available
- [List of tools this agent can use]

# Out of scope
- [Explicit list of what this agent must NOT do]
```

The `CLAUDE.md` is the trust boundary. Keep it specific, keep it honest about limits, and update it as the agent's role evolves.

---

## Trust levels

| Level | Description | Example |
|---|---|---|
| **L1 — Read only** | Can read code, docs, issues. Cannot write. | Research agent browsing repos |
| **L2 — Draft** | Can open PRs, create issues, write to feature branches. Cannot merge. | Default for all agents |
| **L3 — Merge** | Can merge after passing CI. Cannot deploy. | Senior agents after trust established |
| **L4 — Deploy** | Can trigger deployments. Restricted to non-production. | Rare, explicit approval required |
| **L5 — Production** | Human only. No agent ever has unreviewed production access. | Always human |

Start all agents at L2. Promote based on demonstrated reliability, not assumption.

---

## How agents receive work

Agents do not monitor Slack or email for work assignments. Work comes through:

1. **GitHub Issues** — the primary work queue. An issue assigned to an agent (or their team label) is their signal to start.
2. **PR review requests** — agents assigned as reviewers act on those.
3. **Scheduled triggers** — cron-style tasks for recurring work (dependency updates, report generation, etc.).

An agent that has no assigned issues should be idle. They do not self-assign work without explicit authority to do so.
