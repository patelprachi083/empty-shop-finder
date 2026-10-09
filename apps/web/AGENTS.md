# Web Sub-directory Rules

## Next.js 15 — SEO Frontend

### File & Component Rules
- All page components in `app/` are **Server Components** by default.
- Add `"use client"` **only** when the component uses hooks, event handlers, or browser APIs.
- Never `"use client"` on a page file — wrap client logic in a child component.
- All images must use `next/image` — never `<img>` tags.

### SEO Requirements (Non-negotiable)
- Every `page.tsx` must export `generateMetadata()`.
- Listing detail pages must include JSON-LD `<script>` block.
- City and category landing pages must use `generateStaticParams()` for SSG.
- Run `pnpm build` and check for any missing metadata warnings before committing.

### Data Fetching
- Server Components: fetch data directly with `fetch()` or API client (no useEffect).
- Client Components: always use **TanStack Query** — never `useState` + `useEffect` for server data.
- Use `cache: 'force-cache'` for static data, `cache: 'no-store'` for user-specific data.

### Styling
- Use **Tailwind CSS utility classes** only — no inline `style={{}}` props.
- Follow **shadcn/ui** patterns for new UI components.
- Dark mode is not required in MVP — do not add `dark:` variants yet.
