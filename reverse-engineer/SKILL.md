---
name: reverse-engineer
description: Reverse-engineer an existing codebase into complete, dual-audience documentation. Use this skill whenever the user wants to understand, document, or map out an existing/legacy/unfamiliar application — including requests to "reverse engineer this app", "document every feature", "map the flow and logic", "create a CLAUDE.md for this repo", "explain how this codebase works", "onboard me to this project", or produce both business-analyst-friendly flows AND technical docs (database schema, API surface, process flows). Trigger even when the user only says "understand this app" or "documentize the features" without naming the artifacts explicitly. Invoked via /reverse-engineer.
---

# Reverse Engineer

Turn an existing application into two layers of documentation from one investigation:
1. A **CLAUDE.md** so future AI-assisted work on the repo is accurate.
2. A **documentation set** covering every feature — with business-analyst flows AND technical detail (process flows, database, API).

The core principle: **investigate before you write.** Never document from assumptions. Read the code, trace the data, verify against what actually runs.

## Workflow

Work through these phases in order. Track them in a TodoList so nothing is skipped.

### Phase 0 — Orient

Before anything else, understand what you're looking at.

- Identify the repo root. If code isn't uploaded, ask for the path or a repo link, or offer to connect a code host (see "Connecting to sources").
- Detect stack: languages, frameworks, package manifests (`package.json`, `requirements.txt`, `pom.xml`, `go.mod`, `composer.json`, etc.), build tooling, containerization (`Dockerfile`, `docker-compose.yml`), infra config.
- Map the top-level directory structure (2 levels deep) and note the architectural shape: monolith, microservices, monorepo, frontend/backend split.
- Locate entry points: `main`, server bootstrap, route registration, CLI commands, cron/workers.
- Read any existing README/docs — but treat them as claims to verify, not facts.

Produce a short **orientation summary** and confirm the app's purpose with the user before deep-diving. If the user already described the app, skip the confirmation and proceed.

### Phase 1 — Write CLAUDE.md (first deliverable)

Create `CLAUDE.md` at the repo root. This anchors all later work and makes the codebase legible to AI tools. See `references/claude-md-guide.md` for the full template and what makes a CLAUDE.md accurate rather than generic. Cover at minimum: project purpose, tech stack, directory map, how to run/build/test, key architectural decisions, data model overview, external integrations, and conventions. Prefer concrete inline contracts (type mappings, function/callback signatures, env vars) over vague prose — these are what actually improve AI coding accuracy.

Confirm CLAUDE.md with the user before moving on. It's the foundation.

### Phase 2 — Feature inventory

Enumerate every feature by tracing the code, not by guessing. Build the list from:
- Route/endpoint definitions (REST, GraphQL, RPC, page routes).
- UI navigation and screens (if a frontend exists).
- Background jobs, schedulers, event handlers, message consumers.
- CLI commands / admin tools.
- Domain modules/services.

Output a **feature inventory table**: feature name, entry point(s), owning module, one-line purpose. Get user sign-off on the list before writing per-feature docs — this catches missing or misnamed features early.

### Phase 3 — Per-feature documentation

For each feature, produce documentation that serves **both** audiences. Use the exact structure in `references/feature-doc-template.md`. Each feature doc contains:

- **Business view**: what it does, who uses it, the user-facing flow in plain language, business rules, and a BA-readable flow diagram (Mermaid `flowchart`).
- **Technical view**: the request/execution flow (Mermaid `sequenceDiagram`), the code path (files/functions touched, in order), API contract (method, path, params, request/response shape, auth, status codes), data touched (tables/collections, reads/writes), external calls, and error/edge-case handling.

Trace each flow through the actual code. Cite real file paths and function names. If something can't be determined from the code, mark it explicitly as **"unverified — needs confirmation"** rather than inventing it.

### Phase 4 — Cross-cutting technical docs

Beyond individual features, document the system-wide technical layers:

- **Database**: full schema (tables/collections, columns, types, keys, relationships), derived from migrations/ORM models/DDL — not from memory. Include an ER diagram (Mermaid `erDiagram`). Note where GIS/spatial or non-standard types are used.
- **API reference**: consolidated endpoint catalog with auth model, common headers, pagination/error conventions.
- **Architecture**: component/deployment diagram, data flow across services, external integrations (identity providers, message queues, third-party APIs), and the overall runtime topology.
- **Auth & security**: authN/authZ flow, session/token handling, roles/permissions.

### Phase 5 — Assemble & deliver

Organize output as a coherent doc set:

```
docs/
├── README.md                 (index + how to navigate this documentation)
├── architecture.md
├── database.md
├── api-reference.md
├── auth-and-security.md
└── features/
    ├── <feature-1>.md
    ├── <feature-2>.md
    └── ...
```

Plus `CLAUDE.md` at repo root. Use the docx/pptx skills only if the user asks for Word/slide deliverables; the default is Markdown with embedded Mermaid diagrams. Present the files when done.

## Connecting to sources

If the code is not on disk, check for a connected code host (GitHub, GitLab, Bitbucket) or issue tracker via the connector registry before falling back to asking the user to paste code. A repo connection lets you read the full tree instead of working from fragments. Only suggest a connector that matches the user's actual host.

## Investigation discipline (why this matters)

Reverse-engineering fails when the documenter pattern-matches on framework conventions instead of reading the specific implementation. The value of this skill is *accuracy about this app*, so:

- Read the real code path end to end before describing a flow.
- Verify database schema against migrations/models, never infer it from field names in the UI.
- When two sources disagree (README vs. code), the code wins; note the discrepancy.
- Distinguish "what the code does" from "what it appears intended to do."
- Flag anything unverifiable rather than filling gaps with plausible fiction — a labeled gap is useful; a confident wrong claim is harmful.

## Scaling

For a small app, do all phases in one pass. For a large codebase, do Phases 0–2 fully, then document features in prioritized batches (user picks priority order), delivering incrementally. If the investigation would require an impractical number of steps, say so and propose a batched plan rather than producing shallow coverage of everything.
