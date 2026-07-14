# ARCHITECTURE.md — System Architecture

> **Project-specific.** Fill in for each project. Defines how the system
> is built — module boundaries, data flow, API contracts, seams.

## Architecture Overview

[FILL: One paragraph describing the high-level architecture. What pattern?
Monolith, microservices, serverless? What are the major tiers?]

## Module Map

[FILL: List every module with its interface, seam, and responsibility.
Use the vocabulary from AUTONOMOUS.md: module, interface, seam, adapter.]

| Module | Interface | Seam | Responsibility |
|---|---|---|---|
| [FILL: e.g. Auth] | `authenticate(token) → User` | HTTP middleware | User identity, sessions, permissions |
| [FILL] | [FILL] | [FILL] | [FILL] |

## Data Flow

[FILL: Describe how data moves through the system for key user actions.
Use diagrams or step-by-step descriptions.]

### Example: User Login
1. Client → `POST /api/auth/login` with credentials
2. Auth module validates → returns JWT
3. Client stores JWT, sends on subsequent requests
4. Auth middleware verifies JWT on protected routes

[FILL: Repeat for each critical flow]

## API Contracts

[FILL: List all API endpoints with method, path, request body, response body.]

| Method | Path | Request | Response | Auth |
|---|---|---|---|---|
| POST | `/api/auth/login` | `{email, password}` | `{token, user}` | None |
| GET | `/api/users/:id` | — | `{user}` | JWT |
| [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |

## Module Boundaries

[FILL: Define the seams between modules. What can cross a boundary?
What must not? How do modules communicate?]

### Boundary: Web → API
- **Protocol:** HTTP/HTTPS
- **Format:** JSON
- **Auth:** JWT in Authorization header
- **Rate limit:** [FILL]

### Boundary: API → Database
- **ORM/Driver:** [FILL: Prisma, Drizzle, SQLAlchemy, etc.]
- **Connection pooling:** [FILL]

[FILL: Add more boundaries as needed]

## Technical Decisions (ADRs)

[FILL: Record irreversible or surprising decisions. References to full
ADRs in `docs/adr/` if they exist.]

| ADR | Decision | Rationale |
|---|---|---|
| 001 | [FILL: e.g. Use PostgreSQL over MongoDB] | [FILL] |
| 002 | [FILL] | [FILL] |

## Technical Limitations

[FILL: What can't the system do? What are the known constraints?]

- [FILL: e.g. No real-time updates — polling only]
- [FILL: e.g. Max file upload size: 10MB]
- [FILL]
