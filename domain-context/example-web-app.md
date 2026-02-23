---
name: "API & Backend"
triggers:
  paths:
    - "src/api/"
    - "src/routes/"
    - "src/middleware/"
    - "src/server/"
    - "src/lib/db/"
  keywords:
    - "endpoint"
    - "route"
    - "middleware"
    - "auth"
    - "database"
    - "migration"
    - "query"
---

## Overview

The API layer handles all HTTP requests through a route-based architecture. Routes are organized by resource (users, invoices, payments), with middleware for authentication, rate limiting, and input validation. Database access goes through an ORM with typed queries.

## Key Patterns

- **Request lifecycle:** Request → Auth middleware → Validation (Zod) → Handler → Response. Every handler receives a validated, typed input object — no raw `req.body` access.
- **Authentication:** JWT-based with refresh tokens. Auth middleware extracts the user from the token and attaches it to the request context. All routes except `/auth/*` require authentication.
- **Authorization:** Resource-level checks in handlers, not middleware. Each handler verifies the authenticated user has access to the specific resource being requested (e.g., `invoice.orgId === user.orgId`).
- **Database conventions:** All queries go through the ORM. Raw SQL only in migrations. Transactions wrap multi-table writes. Soft deletes via `deleted_at` column on most tables.
- **Error handling:** Handlers throw typed errors (`NotFoundError`, `ForbiddenError`, `ValidationError`). A global error handler catches these and maps them to HTTP status codes. Untyped errors return 500 with a generic message.

## File Locations

| Path | Purpose |
|------|---------|
| `src/api/` or `src/routes/` | Route handlers grouped by resource |
| `src/middleware/` | Auth, rate limiting, validation, error handling |
| `src/lib/db/` | Database client, query helpers, transaction wrapper |
| `src/lib/db/migrations/` | Schema migrations (sequential, timestamped) |
| `src/types/` | Shared request/response types, error types |

## Notes

- Rate limiting is per-user, not per-IP. Stored in Redis with sliding window.
- File uploads go to S3 via presigned URLs — the API never handles raw file bytes.
- Webhook endpoints skip auth middleware but validate signatures (e.g., Stripe webhook secret).
- All timestamps stored as UTC. Timezone conversion happens only in the frontend.
