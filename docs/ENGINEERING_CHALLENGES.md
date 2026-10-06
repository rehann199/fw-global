# Engineering Challenges

## 1. Storage without a server

- **Problem:** the admin tools need shared, durable state, but the site is static and no application server is run.
- **Approach:** edge functions talk to an S3-compatible bucket. Because the edge runtime has no AWS SDK, requests are signed with a compact SigV4 implementation on Web Crypto (HMAC chain, SHA-256 payload hash), kept in one module.
- **Trade-off:** whole-document writes are simple and fit the small data volumes, but two admins editing at once could overwrite each other.
- **Outcome:** one small client reused by every storage-backed endpoint.

## 2. Deciding what is "live"

- **Problem:** a product with a missing icon or policy must not appear on a public page, yet should remain editable.
- **Approach:** a single completeness predicate (all required fields present, at least one benefit and store link, non-empty policy) plus an enabled flag and a per-site visibility field. The admin list's "visible" switch uses the same predicate so it cannot disagree with the public page.
- **Outcome:** records move from "coming soon" to live purely by being completed.

## 3. A small auth gate that avoids common mistakes

- **Problem:** password-only admin access with no user database.
- **Approach:** exact `/admin` and `/admin/*` matching, an HMAC session token compared in constant time, an `HttpOnly`, `SameSite=Strict` (and `Secure` over HTTPS) cookie with a seven-day lifetime, a login endpoint, redirect targets restricted to admin paths, and `no-store, private` on authenticated responses. Legacy dashboard URLs are redirected permanently to the current analytics path.
- **Trade-off:** one shared password and a token derived from it; rotating the password invalidates every session, and there are no per-user audit trails.
- **Outcome:** a gate that fails closed (503 when unconfigured).

## 4. Large admin surface, small public bundle

- **Problem:** the analytics module has many pages and chart-heavy dependencies that public visitors never need.
- **Approach:** admin entry points and each per-app analytics page are lazy-loaded; the build separates large chunks such as the product editor and a funnel chart.
- **Remaining issue:** the shared entry chunk is still about 641 kB (195 kB gzipped) and the build warns about it. Further manual chunking of shared libraries would be the next step.

## 5. Heavy CSV parsing off the main thread

- **Problem:** analytics exports can be large; parsing on the UI thread would freeze the dashboard.
- **Approach:** a web worker parses CSV with PapaParse and normalises rows, trimming stray carriage returns that otherwise corrupt date fields. Parsed datasets are cached in IndexedDB.
- **Outcome:** parsing is off the main thread and repeat visits skip the work.

## 6. One registry shared by two sites

- **Problem:** two company sites read the same stored registry.
- **Approach:** each record has a visibility scope, and each site filters on it; the second site's own ad-verification file uses a separate storage key even though the bucket is shared.
- **Trade-off:** shared storage couples the two sites' data model; changes to the record shape must stay backward compatible.

## 7. Scope notes

- Routing uses a single-page fallback: unknown URLs are served the app, which renders its own not-found screen.
- Per-app privacy policy HTML is authored in the admin area and rendered as HTML, so authoring access is limited to trusted admins.
- The sitemap covers the public marketing pages.
