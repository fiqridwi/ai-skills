# CLAUDE.md Authoring Guide

A CLAUDE.md is the file an AI coding tool reads first. Its job is to make the codebase legible and to prevent the AI from making wrong assumptions. Accuracy and concreteness beat completeness — a short file full of verified specifics outperforms a long generic one.

## Template

```markdown
# <Project Name>

## Purpose
One paragraph: what this application does and for whom. Business context, not marketing.

## Tech Stack
- Language(s) + versions
- Framework(s) + versions
- Datastore(s): e.g. PostgreSQL 15 (+PostGIS), ClickHouse, Redis
- Key libraries that shape how code is written
- Infra: Docker, orchestration, CI

## Architecture (one screen)
Shape of the system (monolith / microservices / monorepo), major components, and how
they talk. Keep it to what an engineer needs to orient in 60 seconds. Link to docs/architecture.md
for detail.

## Directory Map
Annotated tree of the top 2 levels — what lives where and why.
```
src/
├── api/        # HTTP route handlers
├── services/   # business logic
├── models/     # ORM models / schema
└── ...
```

## Running & Building
- Install: `<cmd>`
- Run (dev): `<cmd>`
- Build: `<cmd>`
- Test: `<cmd>`
- Required services / how to start them (docker-compose etc.)

## Environment
Required env vars and what they configure (never include secret values). Note where
they're loaded and validated.

## Data Model (overview)
Core entities and relationships in prose; link to docs/database.md for the full schema.
Call out non-obvious things: soft deletes, multi-tenancy, spatial types, denormalization.

## External Integrations
Identity provider (e.g. Keycloak), third-party APIs, message brokers, object storage —
with the integration point in code.

## Conventions & Contracts
The high-value part. Be concrete:
- Error handling pattern
- Auth/authz enforcement points
- Naming and layering rules
- Type mappings between layers (e.g. DB type ↔ app type ↔ API type)
- Important function/callback signatures the AI must match
- Anti-patterns already present in the code that should NOT be copied

## Gotchas
Race conditions, startup ordering, known fragile areas, things that look wrong but are intentional.
```

## What makes it good

- **Verified, not assumed.** Every command should actually run; every path should exist.
- **Inline contracts.** Type mappings and signatures inline are what most improve AI coding accuracy — prefer them over prose.
- **Points outward.** Link to the detailed docs rather than duplicating them; keep CLAUDE.md scannable.
- **Names the anti-patterns.** If the codebase calls an admin login on every request instead of caching a token, say so — it stops the AI from replicating the mistake.
