# Repo guide for Claude

This is the **Whitespace India** site. One Next.js 14 app, one Vercel project, multiple verticals as sub-routes.

## Layout

```
app/
  page.tsx              # Parent landing — 4-card design, links into /csr
  page.module.css       # Landing styles (ported from design bundle)
  layout.tsx            # Root layout: fonts (Newsreader/Public Sans/Space Grotesk), parent-brand metadata
  csr/                  # CSR vertical — the interactive opportunity index
    page.tsx
    layout.tsx          # CSR-specific metadata (title, OG)
  about/  methodology/  reports/  simulator/  api/
                        # CSR-related sub-routes (kept at top level for stable URLs)

components/             # Shared chrome used by the CSR app (Navbar, Footer, Hero, IndiaMap, …)
                        # The landing page does NOT use these — it's chrome-less by design.

public/landing/         # Hero WebPs for the landing page (high-quality, ~1.5 MB total)
public/                 # Other static assets (logo.svg, favicon.svg, og image, topojson, etc.)

middleware.ts           # Hostname-based routing.
                        # `whitespaceindia-csr.vercel.app` and `csr.whitespaceindia.com`
                        # rewrite to /csr/* so legacy CSR bookmarks keep working.
                        # Add new hosts to LEGACY_CSR_HOSTS as needed.

tailwind.config.ts      # Brand tokens — clay/ink/paper colors, font families, sharp-corner radius.
                        # Used by everything except the landing (which uses CSS variables in page.module.css).
```

## When working on a specific vertical

**Tell Claude which vertical at session start** ("I'm working on CSR" or "…on Education when it exists"). Then:

- **CSR work** → files under `app/csr/`, `app/about/`, `app/methodology/`, `app/reports/`, `app/api/`, plus `components/` (Navbar, Footer, Hero, IndiaMap, RankingList, WeightsPanel, BriefModal, SubscribeForm). Do NOT touch `app/page.tsx` or `app/page.module.css` — those are the parent landing.
- **Future verticals** (Health / Education / Energy) → create `app/<vertical>/page.tsx` + `app/<vertical>/layout.tsx`. Reuse brand tokens from `tailwind.config.ts`. Decide per-vertical whether to share `components/` chrome or build vertical-specific chrome under `components/<vertical>/`.
- **Brand changes (logo, colors, fonts)** → `tailwind.config.ts`, `app/layout.tsx`, `public/logo.svg`, `public/favicon.svg`. Both the landing and the CSR app pick these up automatically (the landing through CSS-variable references like `var(--font-newsreader)`).
- **Landing-only changes** → `app/page.tsx` and `app/page.module.css` only.

## When activating a "Coming soon" card on the landing

In `app/page.tsx`, the Health / Energy / Education cards are `<article>` elements with `cardDim`. To make one live:
1. Build the vertical at `app/<vertical>/page.tsx` (and `app/<vertical>/layout.tsx` for metadata).
2. In `app/page.tsx`, replace the dim `<article>` with a `<Link href="/<vertical>">` element using `cardActive` instead of `cardDim`, mirroring how the CSR card is wired.
3. Update the card's `meta` text from "Coming soon" to a real subtitle (e.g. "Outcomes index · 12 states").
4. Add the new route to `app/sitemap.ts`.

## Hostname routing

`middleware.ts` checks the request `Host` header:
- Hosts in `LEGACY_CSR_HOSTS` (currently `whitespaceindia-csr.vercel.app` + `csr.whitespaceindia.com`) → silently rewrite `/` and any non-`/csr/*` path to `/csr/*`. So someone hitting the old CSR URL never sees the landing.
- All other hosts → behave normally. `/` shows the landing; `/csr` shows the CSR app.

When adding a new custom domain (e.g. `whitespaceindia.com`), no middleware change is needed unless you want it to skip the landing. To add a CSR-dedicated subdomain (e.g. `csr.newdomain.com`), append it to `LEGACY_CSR_HOSTS`.

## Things to know

- **Sharp corners everywhere.** `borderRadius` is overridden in `tailwind.config.ts` so all `rounded-*` classes (except `rounded-full`) are 0. This is intentional — see the design system spec.
- **Three-font system.** `font-headline` (Newsreader serif), `font-body` (Public Sans), `font-label` (Space Grotesk). Don't introduce a fourth.
- **The CSR page is `"use client"`.** It uses framer-motion, react-simple-maps, recharts. Heavy. Keep it on `/csr` only — don't import from it into the landing.
- **No backend.** API routes (`/api/brief`, `/api/subscribe`) handle email collection and PDF generation server-side; otherwise the site is fully static-renderable.
