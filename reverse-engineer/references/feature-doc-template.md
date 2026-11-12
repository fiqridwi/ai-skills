# Per-Feature Documentation Template

Use this exact structure for every feature doc so the set stays consistent and both audiences can navigate it. Fill only what the code supports; label anything unverifiable.

```markdown
# Feature: <Name>

**Module:** <owning module/service>
**Entry point(s):** <route / command / handler + file:line>
**Status:** <active | deprecated | partial>

---

## Business View

### What it does
Plain-language description a non-engineer can follow. One short paragraph.

### Who uses it
Roles/actors and their goal.

### User flow
Numbered plain-language steps of the end-to-end journey.
1. ...
2. ...

### Business rules
Constraints, validations, and decisions expressed in business terms
(e.g. "an order over $X requires manager approval").

### Business flow diagram
\`\`\`mermaid
flowchart TD
    A[User does X] --> B{Condition?}
    B -->|Yes| C[Outcome 1]
    B -->|No| D[Outcome 2]
\`\`\`

---

## Technical View

### Execution flow
\`\`\`mermaid
sequenceDiagram
    participant Client
    participant API
    participant Service
    participant DB
    Client->>API: request
    API->>Service: call
    Service->>DB: query
    DB-->>Service: rows
    Service-->>API: result
    API-->>Client: response
\`\`\`

### Code path
Ordered list of files/functions the request travels through, with what each does.
1. `api/x.py:handle_x()` — validates input, ...
2. `services/x.py:process()` — ...

### API contract (if applicable)
- Method & path: `POST /api/v1/...`
- Auth: <scheme / required role>
- Request: params / body shape
- Response: shape + example
- Status codes: 200 / 4xx / 5xx and when each occurs

### Data touched
| Table/Collection | Operation | Notes |
|---|---|---|
| ... | read/write | ... |

### External calls
Any third-party / cross-service calls and their purpose.

### Error & edge-case handling
How failures are handled; notable edge cases; anything fragile.

### Unverified / needs confirmation
Anything that couldn't be determined from the code.
```

## Diagram guidance

- Keep Mermaid diagrams readable — split a huge flow into two rather than one unreadable graph.
- The `flowchart` is for the BA audience (decisions, outcomes). The `sequenceDiagram` is for engineers (component interaction and ordering).
- Use real component names from the codebase in the sequence diagram, not generic placeholders.
