# Packages Sub-directory Rules

## Shared Packages (packages/)

### packages/validators/ (Zod Schemas)
- Every Zod schema must have a corresponding **TypeScript type export** using `z.infer<>`.
- Schema names use `PascalCase` (e.g., `CreateListingSchema`, `BookingResponseSchema`).
- Validation schemas used in both frontend and backend must live here — **never duplicate** in `apps/`.
- Run `pnpm typecheck` from monorepo root after any schema change to catch breakage across apps.

### packages/ui/ (Shared Components)
- Components here must be **framework-agnostic** (no Next.js-specific imports).
- Every exported component must have a **Storybook story**.
- Props interfaces must be exported alongside the component.
- No business logic in UI components — props and callbacks only.

### packages/config/ (Shared Config)
- ESLint, TypeScript, and Tailwind configs are extended by apps — never duplicated.
- To update a shared config, bump the version in `package.json` so consuming apps can update.
