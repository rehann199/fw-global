# Technical Case Study

## Goal

Ship a fast, credible company website and, with the same codebase and hosting, give the team one private place to manage the product list, builds and analytics, without operating an application server.

## Shape of the solution

A React 18 single-page app built with Vite and TypeScript, hosted as static files on Netlify. Anything that needs a server-side decision runs as a Netlify Edge Function. Shared state (product registry, build status, layout, the ad-verification file) is stored as JSON and text objects in a Cloudflare R2 bucket, which is S3-compatible.

## Front end

- **Routing.** React Router 6. Public pages are in the main bundle; the five admin entry pages and the app-privacy page are lazy-loaded, as are about fifteen analytics pages inside the admin module.
- **Data fetching.** TanStack Query with a 60-second stale time for the registry.
- **UI kit.** shadcn/ui components on Radix primitives, styled with Tailwind and a small set of design tokens. Display type is Geist, body type is Inter.
- **Motion.** Shared easing and variant tokens in one module, with scroll-reveal helpers; hover scale is kept at or below 1.02.
- **Error handling.** A top-level error boundary wraps the app; scroll position resets on navigation.
- **Path aliases.** `@`, `@shared`, `@builds` and `@admin-dashboard` let the site, the build tracker and the analytics module live in separate folders but compile as one app.

## Edge layer

Seven edge functions cover: the admin gate and login endpoint, the product registry, product icons, the repository store, build and QA status, dashboard layout, and the ad-verification file. Public reads (registry, ad file) are open; every write requires the admin session.

Storage requests to R2 are signed with a small AWS Signature V4 implementation using Web Crypto, factored into one shared module.

## Hosting configuration

- SPA fallback: every unknown path serves `index.html` with status 200, so client routes work on refresh. The client renders its own not-found screen.
- Fingerprinted assets under `/assets/*` are cached for a year as immutable; HTML is always revalidated.
- Security headers on all responses: frame denial, no MIME sniffing, strict referrer policy and a restrictive permissions policy (camera, microphone and geolocation disabled).
- An old ad-file URL redirects permanently to the canonical `/app-ads.txt`.

## SEO

Titles and descriptions are set per page with react-helmet-async. The HTML shell carries canonical, Open Graph and Twitter metadata and Organization JSON-LD. A sitemap lists eight public URLs and a robots file allows crawling.

## Verified build output

A clean install and production build succeed. The build emits about 95 assets. The main entry chunk is roughly 641 kB (about 195 kB gzipped). The product editor chunk (about 389 kB) and a funnel chart chunk (about 423 kB) are only loaded in the admin area. The build tool warns that the main chunk exceeds 500 kB.

## Verified on the live site

All public routes return 200 and set a descriptive title. `/admin` redirects to `/admin/login` with a `next` parameter. At a 412 px viewport there is no horizontal overflow.
