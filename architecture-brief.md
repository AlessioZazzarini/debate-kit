<!-- CUSTOMIZE THIS FILE -->
<!-- This is the ONE file you must edit before using /debate. -->
<!-- Fill in each section with your project's details. -->
<!-- The reviewer reads this to understand your codebase — -->
<!-- the more specific you are, the better the review. -->
<!-- Delete these HTML comments when you're done. -->

# Project Architecture

**[Your Project Name]** — [One-sentence description of what it does].

## Tech Stack

<!-- List your languages, frameworks, databases, and key libraries. -->
<!-- Example: TypeScript, Next.js 14, React, PostgreSQL, Redis, Stripe API -->

- **Language:** [e.g., TypeScript]
- **Framework:** [e.g., Next.js 14]
- **Database:** [e.g., PostgreSQL via Supabase]
- **Key libraries:** [e.g., Tailwind CSS, Zod, React Hook Form]
- **External APIs:** [e.g., Stripe, SendGrid, OpenAI]
- **Background jobs:** [e.g., Inngest, BullMQ, none]

## Directory Structure

<!-- Map your key directories. This helps the reviewer navigate your codebase. -->

```
src/
├── app/             → [What lives here]
├── components/      → [What lives here]
├── lib/             → [What lives here]
├── types/           → [What lives here]
└── ...
```

## Key Patterns

<!-- Describe 3-5 architectural patterns the reviewer should know about. -->
<!-- Be specific — "we use X for Y" is better than "standard patterns". -->

- **[Pattern 1]:** [How it works in your project]
- **[Pattern 2]:** [How it works in your project]
- **[Pattern 3]:** [How it works in your project]

## Notes

<!-- Anything else the reviewer should know: deployment model, -->
<!-- auth strategy, multi-tenancy, known tech debt, etc. -->

- [Note 1]
- [Note 2]

<!--
============================================================
FILLED EXAMPLE (for reference — delete this when done)
============================================================

# Project Architecture

**Acme SaaS** — A B2B invoicing platform with automated payment reminders.

## Tech Stack

- **Language:** TypeScript
- **Framework:** Next.js 14 (App Router)
- **Database:** PostgreSQL via Prisma ORM
- **Key libraries:** Tailwind CSS, Zod, tRPC, NextAuth.js
- **External APIs:** Stripe (payments), Resend (email), S3 (file storage)
- **Background jobs:** Inngest (invoice reminders, webhook processing)

## Directory Structure

```
src/
├── app/             → Next.js App Router (pages + API routes)
├── components/      → Shared UI components (shadcn/ui based)
├── lib/             → Core business logic and utilities
│   ├── billing/     → Stripe integration and invoice logic
│   ├── email/       → Email templates and sending
│   └── inngest/     → Background job definitions
├── server/          → tRPC routers and procedures
└── types/           → Shared TypeScript types
```

## Key Patterns

- **tRPC for API layer:** All client-server communication goes through tRPC procedures. No raw fetch calls.
- **Inngest for async work:** Invoice reminders, webhook processing, and PDF generation run as Inngest functions.
- **Row-level security:** Prisma middleware enforces tenant isolation on all queries.
- **Zod everywhere:** Input validation at API boundaries, form validation, and env var parsing all use Zod schemas.

## Notes

- Single-tenant per request (org ID from session, never from client input).
- Stripe webhooks are the source of truth for payment status — we never poll.
- All dates stored as UTC. Frontend converts to user timezone for display.

============================================================
-->
