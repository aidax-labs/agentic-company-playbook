# 4. API-First Cycle (Backend)

## The core rule

**No implementation code is written until the OpenAPI spec is reviewed and merged.**

The spec is the contract between backend and frontend, between the service and its consumers. Writing implementation first and deriving a spec later produces specs that describe implementation details rather than product intent — and makes the spec useless as a coordination tool.

---

## The cycle

```
Domain model → OpenAPI spec → Lint + validate → Human PR review
  → Mock server → Contract tests → Implementation → Integration tests → Merge
```

Each step has a clear owner and a clear exit criterion.

---

## Step 1 — Domain model

Before writing any spec, define the core entities and their relationships.

The `/to-domain-model` skill (or equivalent) takes a PRD and produces:
- A list of entities with their key attributes
- Relationship diagram (in Mermaid, embeddable in GitHub)
- A mapping from user stories to domain operations (CRUD + custom actions)

This becomes the `/docs/domain/feature-name.md` file, referenced by the spec.

**Example:**
```markdown
## Entities

### Organization
- id: UUID
- name: string
- slug: string (unique)
- created_at: timestamp

### Member
- id: UUID
- organization_id: FK → Organization
- user_id: FK → User
- role: enum(owner, admin, member)
```

---

## Step 2 — OpenAPI spec

The spec agent (Leo in the reference team) or the backend agent writes the OpenAPI 3.1 spec for the new endpoints.

**Spec file location:** `/openapi/paths/feature-name.yaml` (or equivalent split structure)

**Non-negotiable spec requirements:**
- Every endpoint has a `summary` and `description`
- Every response status code is documented
- Every schema has property-level descriptions
- Error responses follow a consistent schema (e.g., `{ error: string, code: string }`)
- All `$ref` components are in `/openapi/components/`

---

## Step 3 — Lint and validate with Spectral

[Spectral](https://stoplight.io/open-source/spectral) validates the spec against style rules before any human reviews it.

**Recommended ruleset:** Start with the built-in OAS ruleset, then layer [APIAddicts rules](https://github.com/apiaddicts/spectral-rules) for stricter governance.

```yaml
# .spectral.yaml
extends:
  - spectral:oas
  - https://raw.githubusercontent.com/apiaddicts/spectral-rules/main/ruleset.yaml
rules:
  operation-description: error
  operation-tags: error
  contact-properties: warn
```

CI blocks the PR if Spectral returns errors. Warnings are acceptable but tracked.

**Additional tools from the APIAddicts ecosystem:**
- [sonar-openapi](https://github.com/apiaddicts/sonar-openapi) — SonarQube plugin for API quality metrics
- [openapi2postman](https://github.com/apiaddicts/openapi2postman) — generate Postman collections from specs
- [o2e (openapi to everything)](https://github.com/apiaddicts/o2e) — generate SDK stubs, docs, mocks

---

## Step 4 — Human PR review of the spec

This is one of the two mandatory human gates in the cycle (the other is the implementation PR).

The reviewer checks:
- Does the spec match the PRD intent?
- Are resource names consistent with existing API conventions?
- Are the response schemas appropriate (no over-fetching, no under-fetching)?
- Is pagination handled consistently?
- Are error cases realistic and documented?

A spec PR should take ~10 minutes to review. If it's taking longer, the spec is too large — split the issue.

---

## Step 5 — Mock server with Microcks

Once the spec is merged, [Microcks](https://microcks.io) generates a live mock server from the spec examples.

```bash
# Start Microcks with your spec
docker run -p 8080:8080 quay.io/microcks/microcks-uber:latest \
  --spring.config.location=/path/to/microcks.yaml
```

The mock server:
- Is available immediately after spec merge — no backend implementation needed
- Lets the frontend agent start building against real API shapes
- Serves as the baseline for contract tests

Keep the mock server URL in the repo's `CONTEXT.md` or environment config so all agents know where to find it.

---

## Step 6 — Contract tests

Before the implementation PR merges, automated contract tests verify that the implementation matches the spec.

Options:
- [Microcks contract testing](https://microcks.io/documentation/guides/usage/run-tests/) — tests implementation against the mock
- [Dredd](https://dredd.org) — HTTP API testing from OpenAPI/Blueprint specs
- Custom test suite that imports the OpenAPI spec and validates response schemas

Contract tests run in CI on every PR that touches an implementation file. They fail if the implementation deviates from the spec — not the other way around. The spec is authoritative.

---

## Step 7 — Implementation

With the spec reviewed and the mock running, the backend agent implements the endpoints.

Implementation follows the spec exactly. If the implementation needs to deviate, the agent must:
1. Update the spec first (new PR)
2. Get spec PR reviewed and merged
3. Then update the implementation

Never implement something different from the spec and update the spec to match after the fact.

---

## Step 8 — Integration tests

The backend agent writes integration tests against a real database (not mocked). These tests:
- Cover the happy path for each endpoint
- Cover the primary error cases (validation errors, not found, unauthorized)
- Do NOT use mocked database connections — use a real test database

After integration tests pass and the implementation PR is reviewed, merge.

---

## Summary: who does what

| Step | Owner | Human required? |
|---|---|---|
| Domain model | Backend or Spec agent | Review recommended |
| OpenAPI spec | Spec agent (Leo) | **Yes — review required** |
| Spectral lint | CI (automated) | No |
| Mock server | Spec agent | No |
| Contract tests | Spec agent | No |
| Implementation | Backend agent | **Yes — review required** |
| Integration tests | Backend agent | No |
