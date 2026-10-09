---
name: nextjs-seo
description: >-
  Use this skill when working on the Next.js 15 web frontend (apps/web/).
  Covers SEO patterns (generateMetadata, JSON-LD, ISR, SSG), Tailwind CSS,
  shadcn/ui components, TanStack Query, Razorpay checkout, and Mapbox integration.
---

# Next.js 15 SEO Frontend Skill

## Project Location
All web frontend code lives in `apps/web/`.

## Key Commands
```bash
# Start dev server
cd apps/web && pnpm dev

# Build (checks for SSG/SSR errors)
cd apps/web && pnpm build

# Type check
cd apps/web && pnpm typecheck

# Lint
cd apps/web && pnpm lint

# Run E2E tests
cd apps/web && pnpm test:e2e
```

## SEO Checklist for Every New Page

### 1. Metadata (Required on every page)
```typescript
import type { Metadata } from "next";

export async function generateMetadata({ params }): Promise<Metadata> {
  return {
    title: "Page Title | Empty Shop Finder",
    description: "150-160 character description with primary keyword",
    openGraph: {
      title: "Page Title",
      description: "OG description",
      images: [{ url: "/og-image.jpg", width: 1200, height: 630 }],
    },
    alternates: { canonical: `/path/to/page` },
    robots: { index: true, follow: true },
  };
}
```

### 2. JSON-LD Structured Data (Required for listing pages)
```typescript
// Use LocalBusiness schema for shop listings
const schema = {
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  name: listing.title,
  address: {
    "@type": "PostalAddress",
    addressLocality: listing.city,
    addressRegion: listing.state,
    postalCode: listing.pincode,
    addressCountry: "IN",
  },
  geo: {
    "@type": "GeoCoordinates",
    latitude: listing.lat,
    longitude: listing.lng,
  },
  priceRange: `₹${listing.rent_daily_paise / 100}/day`,
};
return <script type="application/ld+json">{JSON.stringify(schema)}</script>;
```

### 3. Rendering Strategy
- **Homepage, city pages, category pages** → SSG (`generateStaticParams`)
- **Listing detail pages** → ISR (`export const revalidate = 300`)
- **Search results, dashboard, bookings** → CSR (Client Components with TanStack Query)
- **Never** render SEO-critical content client-side only

### 4. Images (Always use next/image)
```typescript
import Image from "next/image";
// Always provide width, height, and alt to prevent CLS
<Image src={url} alt={listing.title} width={800} height={600} priority={isAboveFold} />
```

## Adding a New shadcn/ui Component
```bash
cd apps/web && pnpm dlx shadcn@latest add <component-name>
```

## TanStack Query Pattern (Client Components)
```typescript
"use client";
import { useQuery } from "@tanstack/react-query";
import { api } from "@/lib/api";

const { data, isLoading } = useQuery({
  queryKey: ["listings", filters],
  queryFn: () => api.listings.search(filters),
  staleTime: 30_000, // 30 seconds
});
```

## Environment Variables Required
```
NEXT_PUBLIC_API_URL=https://api.emptyshopfinder.in
NEXT_PUBLIC_RAZORPAY_KEY_ID=
NEXT_PUBLIC_MAPBOX_TOKEN=
NEXTAUTH_URL=
NEXTAUTH_SECRET=
```
