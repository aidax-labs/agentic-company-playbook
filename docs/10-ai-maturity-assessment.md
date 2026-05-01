# 10. AI Maturity Assessment

Before designing an agentic architecture for any company, assess where they actually are. Most organizations overestimate their level. This framework cuts through the theater.

The assessment is built around four questions that expose the real state of AI adoption — not the perceived one:

1. **What can AI see?** Is the company's work legible to a machine, or does it live in people's heads, undocumented meetings, and SaaS tools AI cannot read?
2. **What can AI do?** Can it act on systems of record (open PRs, update CRMs, reconcile invoices) or only summarize what humans already wrote down?
3. **Who can extend the system?** Are non-engineers shipping production workflows, or is everything held together by a few power users whose knowledge walks out the door when they leave?
4. **How has the organization changed?** Or is it running a 2023 org chart with better autocomplete?

---

## The six levels

### L0 — AI as theater

| Question | Reality |
|---|---|
| What can AI see? | Nothing structured. Knowledge lives in people's heads. |
| What can AI do? | Nothing of consequence. Maybe summarize a pasted transcript. |
| Who can extend it? | No one. AI is a personal tool, ungoverned, unintegrated. |
| How has the org changed? | It hasn't. Same chart, same handoffs, same dependence on managers. |

**Hard test:** Can AI complete any recurring business process end-to-end?

**Common false positive:** A CEO who gives an excellent speech about AI transformation while still running the company through the same executive staff meetings and headcount plans. Announcements ≠ adoption.

---

### L1 — Personal productivity

| Question | Reality |
|---|---|
| What can AI see? | Only what each individual feeds it. No org-level visibility. |
| What can AI do? | Help individuals draft, summarize, brainstorm, code. No action on systems of record. |
| Who can extend it? | Each user reinvents independently. Power users are heroes. |
| How has the org changed? | It hasn't. Maybe a "Head of AI" hire with a budget. |

**Hard test:** If your best AI user left tomorrow, would their workflow remain in the company?

**Common false positive:** "80% of employees use AI weekly!" — probably true, and meaningless.

---

### L2 — Team workflows

| Question | Reality |
|---|---|
| What can AI see? | Shared context within teams (shared prompts, function-specific integrations). AI sees within team boundaries. |
| What can AI do? | Functional workflows — sales prospecting, support triage, code review. Bounded actions within a team's domain. |
| Who can extend it? | Within a team, non-engineers can tap shared workflows. Across teams, each function rebuilds the same thing privately. |
| How has the org changed? | Functional efficiency within roles. Hiring slows but org shape unchanged. |

**Hard test:** Does any workflow cross team boundaries, or is every function building its own private AI stack?

**Common false positive:** "We have AI workflows in every department." But they don't connect — the company is a collection of AI-enhanced silos, not an AI-native organization.

---

### L3 — Organizational infrastructure

| Question | Reality |
|---|---|
| What can AI see? | The whole organization is queryable. Cross-functional context accessible via CLI, MCP, or well-defined APIs. |
| What can AI do? | Agents act across systems — update CRMs, open PRs, route tickets, draft customer communications, reconcile invoices. |
| Who can extend it? | Non-engineers author skills, not just consume them. Skills move horizontally across functions. |
| How has the org changed? | The org chart looks materially different from a 2023 equivalent. An explicit structural choice has been made about how AI changes who does what. |

**Hard test:** Can an agent answer — across systems — what shipped last sprint, who asked for it, what broke after launch, what customers said, and what the company should do next, without convening a cross-functional meeting?

**Common false positive:** A landfill of meeting transcripts and dashboards with no synthesis. Capture is not legibility.

**This is the critical jump.** Most companies stall between L2 and L3. The blocker is almost never technical capability — it's trust. Can leadership trust an agent to act on systems of record without a human reviewing every step? This is where compliance infrastructure becomes a competitive advantage.

---

### L4 — Compounding operating system

| Question | Reality |
|---|---|
| What can AI see? | Not just what happens, but the relationships between what happens. The system maintains its own context. Agents update agents. |
| What can AI do? | Policy-driven decision authority within scoped domains. Custom internal harnesses purpose-built for the company's most common work. |
| Who can extend it? | Non-engineers ship production internal tools. An AE ships a sales tool in under an hour without filing a ticket. |
| How has the org changed? | Hierarchy collapses toward "channel managers" of agent workflows. Compensation tied to AI proficiency. Customer signal-to-ship measured in hours. |

**Hard test:** Show a workflow that got better because the system learned from prior runs, not because one heroic person manually improved it. Plus: three production tools shipped by non-engineers in the last quarter.

**Common false positive:** Agent sprawl. A hundred brittle automations don't equal a compounding operating system. L4 requires managed compounding — lifecycle, observability, evaluation — not chaotic proliferation.

---

### L5 — Virtually self-driving organization

The core operating loops can sense reality, diagnose issues, initiate work, execute within delegated authority, update shared memory, and improve future behavior — with humans governing strategy, taste, risk, and values rather than running the loops themselves.

The six L5 markers:
1. The system notices something important without being asked
2. The system synthesizes across multiple sources of context
3. The system decides whether action is warranted
4. The system acts within delegated authority
5. The system escalates when uncertainty or consequence exceeds its authority
6. The system updates shared memory so future behavior improves

**Hard test:** What important thing did the company notice, decide, act on, and learn from recently without a human initiating the process? Not a threshold alert, not a configured automation — something the system synthesized that humans hadn't framed as a question yet.

*Note: No organization has fully achieved L5 as of 2026. It is a direction, not a destination.*

---

## How to use this assessment

### Step 1 — Score each dimension independently

A company rarely answers all four questions at the same level. Score each independently (L0–L5):

| Dimension | Score | Evidence |
|---|---|---|
| What AI can see | | |
| What AI can do | | |
| Who can extend it | | |
| How the org has changed | | |

The asymmetry tells you where the next intervention should focus. Sometimes AI sees a lot but can't do much. Sometimes AI can do a lot but only engineers can extend it.

### Step 2 — Identify the binding constraint

The lowest score is almost always the binding constraint. Address that first — raising any other dimension won't move the overall level.

### Step 3 — Design the path to the next level

Each level transition has a primary blocker:

| Transition | Primary blocker | What to build |
|---|---|---|
| L0 → L1 | Awareness and tooling | Individual AI access, basic prompting culture |
| L1 → L2 | Shared context | Team knowledge bases, shared prompts, function-specific integrations |
| L2 → L3 | **Trust and compliance** | Audit trails, access controls, system-of-record integrations, agent identity model |
| L3 → L4 | Observability and lifecycle | Agent monitoring, skill governance, compaction discipline |
| L4 → L5 | Generative sensing | Self-initiating loops, delegated authority frameworks |

**The L2→L3 transition is where aidax operates.** The companies that stall here have the technical capability but lack the trust infrastructure: clear agent authority boundaries, audit trails for every action, compliance posture that satisfies legal and regulators, and a trust center that communicates this to customers.

### Step 4 — Define the trust layer

Before any L3 work begins, answer:
- What systems of record will agents be allowed to write to?
- What is the escalation path when an agent exceeds its authority?
- Who is accountable for agent actions?
- How is this communicated to customers, regulators, and the board?

These are not afterthoughts. They are prerequisites for operating at L3 and above.

---

## Assessment template

Use this in the first conversation with a prospective client or new team:

```markdown
## AI Maturity Assessment — [Company Name]
Date: YYYY-MM-DD

### Current state scores
- AI visibility: L__  Evidence: ___
- AI capability: L__  Evidence: ___
- Extensibility:  L__  Evidence: ___
- Org change:    L__  Evidence: ___

### Binding constraint
[The lowest-scoring dimension and why]

### Target state (12 months)
[Realistic next level across all four dimensions]

### Primary intervention
[The one thing that unlocks the transition]

### Trust and compliance gaps
[What needs to be in place before agents can act on systems of record]
```
