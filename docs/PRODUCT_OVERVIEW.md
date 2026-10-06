# Product Overview

FW Global is the public website and internal operations console for FW Global LLC, a company that positions itself as an AI applications company. This case study covers the website at https://fwglobal.co and the private tooling that sits behind it.

## Public site

| Route | Purpose |
|---|---|
| `/` | Home: positioning statement, three commitments, a "zero to one" argument, a scrolling strip of products, closing call to action |
| `/about` | Company story, philosophy and operational commitments |
| `/products` | Catalogue of apps with platform filters (All, Android TV, Android, iOS, Apple TV), benefits, store badges and privacy links |
| `/kalendra` | Dedicated page for the featured AI scheduling product, with capability and audience sections and a link out to the product's own site |
| `/careers` | Hiring philosophy, operating principles and what the environment offers, with contact links |
| `/contact` | Direct email, social link and a contact form |
| `/privacy`, `/terms` | Company-level legal pages (numbered sections) with an index of per-app policies |
| `/privacy/:appId` | Privacy policy for one app, addressed by bundle id or a custom slug |
| any other path | Branded "Page not found" screen |

## Product catalogue

At the time of capture the Products page lists seven products: Kalendra (web scheduling product) and six mobile apps. Filter chips show a count per platform. Cards with store links show Google Play and App Store badges; where a store listing is not yet linked the badge carries a "coming soon" label.

The catalogue is data-driven. A built-in list renders immediately; the live registry is fetched from `/api/apps` and merged in, so products can be added or edited without redeploying the site.

## Admin console (private)

Access requires a password-based session. It is described here at a high level only.

| Area | What it does |
|---|---|
| Edit website | Add, edit, hide or remove products; icons; platforms; store links; benefits; rich-text privacy policies; the ad-verification file |
| Builds | Track builds per app, QA queue and sign-off, release history, upload entries |
| Repository | Per-app reference data: privacy links, icons, identifiers, documentation and store links |
| Analytics | Overview and per-app dashboards for usage, monetisation, geography and stability |

## Audience

Prospective users and partners reading about the company, store reviewers checking that each app has a reachable privacy policy, ad networks crawling the app-ads file, and the internal team running the console.
