# Architecture

## Overview

```mermaid
flowchart TB
    subgraph Browser
        SPA[React SPA<br/>Router + TanStack Query]
        PUB[Public pages]
        ADM[Admin pages<br/>lazy-loaded]
        WK[CSV web worker]
        IDB[(IndexedDB cache)]
        SPA --> PUB
        SPA --> ADM
        ADM --> WK
        ADM --> IDB
    end

    subgraph Netlify
        STATIC[Static hosting<br/>SPA fallback, headers, caching]
        subgraph Edge Functions
            GATE[protect-admin]
            APPS[apps]
            ICON[app-icon]
            REPO[repository]
            BLD[builds-status]
            LAY[widget-layout]
            ADS[app-ads-txt]
        end
    end

    R2[(Cloudflare R2)]
    MAIL[Hosted email service]

    SPA --> STATIC
    PUB -->|GET /api/apps| APPS
    PUB -->|GET /app-ads.txt| ADS
    PUB -->|contact form| MAIL
    ADM -->|writes, session cookie| APPS
    ADM --> ICON
    ADM --> REPO
    ADM --> BLD
    ADM --> LAY
    GATE -.guards /admin.-> STATIC
    APPS --> R2
    ICON --> R2
    REPO --> R2
    BLD --> R2
    LAY --> R2
    ADS --> R2
```

## Layers

**Presentation.** Pages and section components under one source tree. Public pages share a layout (navbar, footer). Admin pages share an admin layout with sidebar navigation. The analytics and build-tracker modules live in their own folders and are mounted by path alias.

**Data access.** `useApps()` wraps the registry fetch in TanStack Query. Admin screens use plain `fetch` helpers that send the session cookie. Analytics data is loaded as CSV or JSON, parsed in a web worker and cached in IndexedDB.

**Edge.** Each edge function owns one resource and one URL. Reads that must work for visitors are unauthenticated; writes check the admin session cookie. A shared module holds session helpers; another holds the R2 signing client.

**Storage.** One R2 bucket with key prefixes per concern: product registry, build state documents, ad-verification text and uploaded icons.

## Request flows

### Visitor opens Products

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Static host
    participant E as apps edge function
    participant R as R2
    B->>S: GET /products (index.html)
    S-->>B: SPA shell
    B->>B: Render built-in fallback products
    B->>E: GET /api/apps
    E->>R: Read registry document
    R-->>E: JSON
    E-->>B: apps
    B->>B: Filter enabled + complete + visible, merge, re-render
```

### Admin sign-in

```mermaid
sequenceDiagram
    participant B as Browser
    participant G as protect-admin
    B->>G: GET /admin/...
    G-->>B: 302 to /admin/login?next=...
    B->>G: POST /admin/api/login (password)
    G->>G: Constant-time compare
    G-->>B: 200 + HttpOnly SameSite=Strict cookie
    B->>G: GET /admin/... (cookie)
    G-->>B: Page, Cache-Control no-store
```

## Failure behaviour

- If the registry call fails, the built-in product list stays on screen.
- If the admin password is not configured, the gate returns 503 instead of opening the area.
- If storage credentials are missing, storage-backed endpoints report that and fall back where a static default exists (for example the ad-verification file).
- A React error boundary prevents a render error from blanking the whole app.
