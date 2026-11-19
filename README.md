# Vice City Countdown: unofficial GTA VI fan site

This is a multi-page Next.js 16 (App Router) fan site for Grand Theft Auto VI. It shows a live launch countdown for each region, world clocks, character profiles, Leonida locations, platforms, news, and leaks with labels that say how far each claim can be trusted.

> This is an unofficial GTA VI fan website. It is not affiliated with, sponsored by, or endorsed by Rockstar Games or Take-Two Interactive.
> Designed & Developed by The Agentic State.

## Quick start

```bash
npm install
cp .env.example .env.local   # set NEXT_PUBLIC_SITE_URL / NEXT_PUBLIC_TEAM_WEBSITE_URL
npm run dev                  # http://localhost:3000
npm run build && npm start   # production
```

### Environment variables

| Variable | Purpose |
|---|---|
| `NEXT_PUBLIC_SITE_URL` | The canonical origin. It is used for metadata, the sitemap, robots.txt and JSON-LD. |
| `NEXT_PUBLIC_TEAM_WEBSITE_URL` | The official website of The Agentic State. **Leave it empty until the exact domain is confirmed.** While it is empty, the footer shows the credit without a link. |

## Testing

```bash
npx playwright install chromium        # once
npm run build
npm run test:e2e                       # functional tests (desktop + mobile)
BASE_URL=http://localhost:3001 node scripts/qa-crawl.mjs   # every route × 5 widths: console/hydration errors, broken images, dead links, overflow
node scripts/shot.mjs /regions/pakistan 375 shot.png       # single screenshot + console report
```

## How it's built

| Area | Implementation |
|---|---|
| Shared clock | `src/lib/clock.ts` is a single store built on `useSyncExternalStore`. It ticks once per second, aligned to the wall clock. Only the leaf components that show time re-render. The server snapshot is `null`, so the markup always hydrates without a mismatch. |
| Time zones | `src/lib/time.ts` uses the `Intl` API with IANA time zones. It caches formatters, and offsets and DST come from the runtime's time zone data. |
| Launch logic | `src/content/release.ts`. Rockstar's stated policy is "midnight local time … in all regions". `localMidnightUtc(zone)` returns the exact UTC instant of local midnight on Nov 19 for each zone. Regions without an official storefront (Pakistan) get **no** countdown. They show "Exact unlock time not confirmed" instead. |
| Motion | GSAP + ScrollTrigger handle the hero entrance, scrubbed parallax, batched reveals and timelines. Motion handles nav, the mobile menu and the lightbox. Only transforms and opacity are animated. Everything goes through `gsap.matchMedia` with `prefers-reduced-motion`. Parallax is disabled below 768px. |
| Images | `next/image` serves AVIF/WebP. Hero art is preloaded. The location illustrations are deterministic inline SVG (`LocationScene`), so they need no image downloads. YouTube embeds are click-to-load facades. |
| SEO | Each page has its own metadata and canonical URL. The site also provides `sitemap.xml`, `robots.txt`, and JSON-LD (WebSite, BreadcrumbList, ProfilePage, Article). |

## Editorial update workflow

All content lives in `src/content/`. These files are typed data, kept separate from the UI. Components never hard-code facts.

| File | What to edit |
|---|---|
| `release.ts` | Release date, unlock policy, pre-load info, disputed claims |
| `platforms.ts` | Platform `status` (`confirmed`, `announced`, `coming-soon`, `released`, `unannounced`). **When PC is announced:** change the `pc` entry's `status` and `statusNote`, and replace its `details`. The UI updates everywhere. |
| `prices.ts` | Price ledger. Every record has `amount`, `currency`, `kind`, `sourceUrl` and `verifiedOn`. |
| `regions.ts` | Add a market by appending an object with an IANA `timeZone`. The clocks, search, rollout table and sitemap all pick it up. |
| `characters.ts`, `locations.ts` | Profiles. Every claim carries a `status` and its `sources`. |
| `news.ts`, `leaks.ts`, `trailers.ts`, `media.ts` | Articles, rumours, videos and gallery items |

Rules:

1. **Never overwrite verified data silently.** Add the new record with a new `verifiedOn` date and source. Remove the superseded record in the same commit, so the git history keeps the provenance.
2. **Keep prices in their original local currency.** Never convert them. Use `kind` to tell apart `official-digital`, `reported-store` (single source), `authorized-retailer`, `independent-reseller`, `estimate` and `unverified`.
3. **Label every claim:** `official` · `credible` · `rumor` · `disputed`. If you're unsure, pick the lower level.
4. **Write summaries in your own words and link to the source.** Do not copy articles. Do not host leaked material.
5. **Images:** set `origin` to `official`, `fan` or `ai`. Add official Rockstar media only when the file is stored locally with permission and credited.
6. **New Rockstar screenshots:** put the files in `public/images/gallery/` and add `MediaItem` entries with `origin: "official"` and a `creditUrl`.

## Assets

`references-original/` holds the five supplied reference images. `npm run optimize-images` regenerates the cropped and compressed WebP assets in `public/images/` from those images. All of the character and city artwork is **AI-generated fan art**, and it is labelled that way wherever it appears. The mockups themselves (which contain made-up prices and times) are not published.

## Deployment (Vercel)

1. Import the repository into Vercel. The framework (Next.js) is detected automatically, so no build settings are needed.
2. Set `NEXT_PUBLIC_SITE_URL` to the production domain. Set `NEXT_PUBLIC_TEAM_WEBSITE_URL` once The Agentic State's domain is confirmed.
3. Add the custom domain under **Project → Settings → Domains**, then create the DNS record Vercel shows. That is an `A` record for an apex domain, or a `CNAME` to `cname.vercel-dns.com` for a subdomain.
4. Every page is prerendered statically. No database or paid service is needed.
