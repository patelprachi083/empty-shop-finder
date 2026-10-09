# Empty Shop Finder — Workspace Rules

## Project Overview
This is a two-sided commercial property rental marketplace for India.
- **Frontend**: Next.js 15 (App Router) — SEO-first web app
- **Mobile**: Flutter 3 (Android + iOS)
- **Backend**: Python FastAPI — core API (`apps/api/`)
- **AI Service**: Python FastAPI — AI/ML microservice (`apps/ai-service/`)
- **Database**: Supabase PostgreSQL 16 + PostGIS + pgvector
- **Cache**: Redis (Upstash)
- **Storage**: Cloudflare R2
- **Search**: Meilisearch
- **Payments**: Razorpay (UPI-first)
- **KYC**: Signzy
- **Infra**: GCP Cloud Run (asia-south1 — Mumbai)

---

## General Rules

### Language & Type Safety
- All TypeScript (Next.js, packages) must use **strict mode** (`"strict": true`).
- All Python code must use **type hints** on every function signature.
- Never use `any` in TypeScript. Use `unknown` and narrow the type.
- All Python models use **Pydantic v2** for validation.

### Code Style
- Python: follow **PEP 8**. Use `ruff` for linting and formatting.
- TypeScript: use **ESLint** + **Prettier**. No semicolons, single quotes.
- Dart (Flutter): follow official **Dart style guide** with `flutter analyze`.

### Naming Conventions
- Python files/modules: `snake_case`
- TypeScript files: `PascalCase` for components, `camelCase` for utilities
- Database columns: `snake_case`
- API endpoints: `kebab-case` nouns, plural (`/listings`, `/bookings`)
- Environment variables: `SCREAMING_SNAKE_CASE`

### Security
- Never hardcode secrets, API keys, or tokens in source code.
- Always use environment variables via `pydantic-settings` (Python) or `process.env` (Next.js).
- All API routes require authentication unless explicitly decorated with a `public` marker.
- Validate ALL inputs at the API boundary using Zod (TS) or Pydantic (Python).
- Use parameterised queries only — never raw SQL string interpolation.

### India-Specific Rules
- All monetary values are stored as **paise** (integer) in the database.
- Display amounts as **₹** (INR) in UI, converted from paise (`paise / 100`).
- GST (18%) must be calculated and shown separately in all invoices.
- Phone numbers must be stored with country code: `+91XXXXXXXXXX`.
- Default locale is `en-IN`; support `hi` (Hindi) as secondary.
- UPI must be the **default and most prominent** payment option.

---

## Architecture Rules

### Layered Architecture (Python API)
- `routers/` — HTTP layer only. No business logic.
- `services/` — All business logic lives here.
- `models/` — SQLAlchemy ORM models.
- `schemas/` — Pydantic request/response DTOs.
- Services must **never** import from `routers/`.
- Domain logic must **never** import from infrastructure (Razorpay, S3, Redis) directly — use service abstractions.

### Booking State Machine
- Booking status transitions must **always** be validated server-side via the state machine in `services/booking_service.py`.
- Frontend booking state is cosmetic only — never trust client-sent status values.
- Valid transitions: `PENDING_APPROVAL → APPROVED | REJECTED`, `APPROVED → PAYMENT_PENDING | CANCELLED`, `PAYMENT_PENDING → CONFIRMED | CANCELLED`, `CONFIRMED → ACTIVE | CANCELLED`, `ACTIVE → COMPLETED`.

### API Design
- All endpoints follow `/api/v1/{resource}` convention.
- Use cursor-based pagination (not offset) for all list endpoints.
- Standard response envelope: `{ "success": bool, "data": ..., "meta": { "cursor": ... } }`.
- Never expose internal UUIDs raw — always use the UUID as-is (no sequential IDs in public URLs).
- Idempotency keys required on all payment POST endpoints.

### Database
- All migrations managed by **Alembic** (`apps/api/alembic/`). Never edit the DB schema manually.
- All geo queries use **PostGIS `ST_DWithin`** with GiST index — never lat/lng math in application code.
- Vector similarity search uses **pgvector `<=>` operator** (cosine distance).
- Connection pooling via Supabase's built-in pooler (PgBouncer mode).

### SEO (Next.js)
- Every page must export `generateMetadata()` with `title`, `description`, `openGraph`, and `alternates.canonical`.
- Listing detail pages use **ISR** with `revalidate = 300` (5 minutes).
- City and category pages use **SSG** (`generateStaticParams`).
- All listing pages must include **JSON-LD** structured data (`LocalBusiness` schema).
- Never use client-side-only rendering for SEO-critical content.

---

## Monorepo Rules
- Package manager: **pnpm** (Node.js packages), **uv** (Python packages).
- Build orchestration: **Turborepo**.
- Shared Zod schemas live in `packages/validators/` — import from there, don't duplicate.
- Shared UI components live in `packages/ui/` — don't copy-paste components between apps.
- Apps must never import directly from other apps — only from `packages/`.

---

## Testing Rules
- Python: minimum **80% coverage** on `services/` and `models/` layers using `pytest`.
- TypeScript: unit test all utility functions in `packages/` with **Vitest**.
- E2E: critical flows (OTP login → list shop → search → book → pay) covered by **Playwright**.
- Never skip tests with `pytest.mark.skip` or `it.skip` without a linked issue comment.

---

## Git & PR Rules
- Branch naming: `feat/`, `fix/`, `chore/`, `refactor/` prefixes.
- Commit messages follow **Conventional Commits**: `feat(listings): add geo search endpoint`.
- PRs must pass all CI checks before merging.
- No direct pushes to `main`.
