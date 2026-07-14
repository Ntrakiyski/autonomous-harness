# DATABASE.md — Database Models

> **Project-specific.** Fill in for each project. Defines all database
> models, relationships, migrations, and invariants.

## Database Overview

- **Engine:** [FILL: PostgreSQL, SQLite, MySQL, MongoDB, etc.]
- **ORM:** [FILL: Prisma, Drizzle, SQLAlchemy, Mongoose, etc.]
- **Host:** [FILL: local, Supabase, PlanetScale, RDS, etc.]
- **Connection string:** [FILL: env var name, e.g. DATABASE_URL]

## Models

[FILL: List every model/table with columns, types, constraints, and
relationships. Use the exact names from the schema.]

### [FILL: Model Name, e.g. User]

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK, default: uuid() | Unique identifier |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | User email |
| `name` | VARCHAR(255) | NOT NULL | Display name |
| `created_at` | TIMESTAMP | NOT NULL, default: now() | Creation time |
| `updated_at` | TIMESTAMP | NOT NULL, default: now() | Last update |

**Relationships:**
- Has many: [FILL: e.g. Orders]
- Belongs to: [FILL: e.g. Organization]

### [FILL: Repeat for each model]

## Relationships Diagram

[FILL: Describe or diagram the relationships between models.
e.g. User 1→N Order N→1 Product]

## Invariants

[FILL: Business rules that the database must enforce. These are
non-negotiable constraints.]

- [FILL: e.g. Email must be unique across all users]
- [FILL: e.g. Order total must equal sum of line items]
- [FILL: e.g. Deleted users cascade to their orders (soft delete)]

## Migrations

- **Migration tool:** [FILL: Prisma Migrate, Alembic, Knex, etc.]
- **Migration command:** [FILL: `npx prisma migrate dev`, etc.]
- **How to create a migration:** [FILL: step-by-step]
- **How to apply migrations:** [FILL]
- **Rollback:** [FILL: procedure]

## Seed Data

- **Seed command:** [FILL]
- **What seeds exist:** [FILL: dev, test, demo accounts]

## Query Patterns

[FILL: Common query patterns that agents should follow.]

- Pagination: [FILL: cursor-based or offset?]
- Filtering: [FILL: how to build dynamic filters]
- Ordering: [FILL: default sort, allowed sort fields]
- Eager loading: [FILL: when to include relations]
