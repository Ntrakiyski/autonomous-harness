# PROJECT.md — Project Constitution

> **The highest-level document. Every milestone, phase, PRD, and ticket
> descends from this file.** Write it once. Change it rarely. Read it first.

## What This Project Is

[FILL: One paragraph. What does this product do? Who is it for? Why does
it exist? Keep it under 5 sentences. A new team member should understand
the product from this paragraph alone.]

## North Star

[FILL: One sentence. The single most important outcome this project exists
to achieve. Not a feature. Not a metric. The reason the project matters.]

Example: "Make health data from wearables actionable for patients and their
doctors without requiring medical training."

## Users

[FILL: Who uses this? Be specific.]

- **Primary user:** [FILL: e.g. Patient managing a chronic condition]
- **Secondary user:** [FILL: e.g. Doctor reviewing patient history]
- **Admin user:** [FILL: e.g. Clinic administrator]

## Core Capabilities

What the product MUST do at minimum. These are not features — they are
capabilities the product cannot ship without.

- [FILL]
- [FILL]
- [FILL]

## Explicit Non-Goals

What this product explicitly does NOT do. Prevents scope creep across
milestones.

- [FILL]
- [FILL]

## Constraints

Technical, business, or regulatory constraints that apply to ALL
milestones.

- [FILL: e.g. Must be HIPAA compliant]
- [FILL: e.g. Must work offline]
- [FILL: e.g. Must support Bulgarian language]
- [FILL: e.g. Tech stack: Next.js, TypeScript, PostgreSQL, Tailwind]

## Milestones

High-level roadmap. Each milestone is a self-contained deliverable that
adds value on its own. Details live in each milestone's PRD.

### Milestone 1: [FILL: Name]
- **Goal:** [FILL: One sentence]
- **Status:** [planned / in-progress / shipped]
- **PRD:** `milestones/m1/prd.md`

### Milestone 2: [FILL: Name]
- **Goal:** [FILL: One sentence]
- **Status:** [planned]
- **PRD:** `milestones/m2/prd.md`

## Design Principles

Principles that guide decisions across ALL milestones. These are the
project's "constitution" — they override convenience.

- [FILL]
- [FILL]
- [FILL]

## Key Decisions (ADRs)

Architecture Decision Records that affect the entire project. Milestone-
specific ADRs live in ARCHITECTURE.md.

| ID | Decision | Rationale | Date |
|---|---|---|---|
| ADR-001 | [FILL] | [FILL] | [FILL] |
| ADR-002 | [FILL] | [FILL] | [FILL] |

## Glossary

Domain-specific terms that mean something specific in this project.
Shared across all agents and all milestones.

| Term | Definition |
|---|---|
| [FILL] | [FILL] |
| [FILL] | [FILL] |
