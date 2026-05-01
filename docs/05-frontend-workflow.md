# 5. Frontend Workflow

## How it differs from backend

The backend cycle is driven by the API spec. The frontend cycle is driven by consuming that spec. These are parallel concerns — the frontend agent does not wait for the backend implementation to start building.

```
Backend:  Spec (merged) → Mock server running → Implementation → Merge
Frontend:               ↘ UI components → Integration w/ mock → Swap to real API → Merge
```

The Microcks mock server is the seam between the two pipelines.

---

## The frontend cycle

### Step 1 — Read the spec and mock

Before writing a single component, the frontend agent:
1. Reads the merged OpenAPI spec for the feature
2. Verifies the mock server is running and returns expected shapes
3. Maps the user stories from the PRD to the UI flows they need to build

This is the equivalent of the domain model step for backend.

### Step 2 — Component development against the mock

The frontend agent builds UI components using the mock server as the API. No real backend required.

For local development, the mock server URL is set in the environment:
```
NEXT_PUBLIC_API_URL=http://localhost:8080/api
```

In CI and Storybook, use [MSW (Mock Service Worker)](https://mswjs.io/) to intercept requests:
```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.get('/api/v1/organizations/:orgId', ({ params }) => {
    return HttpResponse.json({ id: params.orgId, name: 'Acme Corp' })
  }),
]
```

MSW handlers are generated from the OpenAPI spec examples — the Spec agent (Leo) can produce these alongside the mock server setup.

### Step 3 — Component tests

Each component has tests that:
- Render correctly with expected data
- Handle loading states
- Handle error states (404, 401, 500)
- Handle empty states

Tests run against MSW, not the real API or Microcks. This makes them fast and deterministic.

### Step 4 — Integration with the real backend

Once the backend implementation is merged, the frontend agent:
1. Points the environment to the real API (staging or local dev server)
2. Runs the full flow end-to-end
3. Fixes any discrepancies between mock behavior and real behavior (these should be rare if the spec is tight)

If real behavior differs from the spec, the backend agent must fix the implementation — not the frontend agent's expectations.

### Step 5 — PR and review

The frontend PR references both:
- The feature issue it closes
- The backend PR it depends on (if merging order matters)

The reviewer checks:
- Does the UI match the design spec (if one exists)?
- Are all acceptance criteria from the issue met?
- Are error and loading states handled?
- Is there no direct API URL or secret hardcoded?

---

## Design tokens and component library

An agentic frontend works best when there is a clear component library. Agents should:
- Use existing components from the library rather than creating new ones
- Add to the library only when no existing component fits
- Follow the naming and variant conventions established for the project

Document the component library location in `CONTEXT.md` so agents always know where to look.

---

## CONTEXT.md

The frontend repo (or frontend directory) should have a `CONTEXT.md` that documents:
- Where the component library lives
- Design system tokens (colors, spacing, typography)
- Routing conventions
- State management patterns
- How to run the local dev server
- Where to find the mock server and OpenAPI spec

This file is always read by agents at the start of a frontend task. Keep it current.

---

## Summary: who does what

| Step | Owner | Human required? |
|---|---|---|
| Spec + mock available | Spec agent (upstream) | Spec review (already done) |
| MSW handler generation | Spec agent | No |
| Component development | Frontend agent | No |
| Component tests | Frontend agent | No |
| Integration with real API | Frontend agent | No |
| UI/UX review | Human | **Yes** |
| PR merge | Human or senior agent | **Yes** |
