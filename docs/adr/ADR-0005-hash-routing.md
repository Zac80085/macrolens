# ADR-0005: Hash routing for GitHub Pages

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

GitHub Pages serves static files and has no rewrite rules, so a reload of `/macrolens/favourites` returns 404 unless the route is in the URL fragment. The app is served under `/macrolens/` (S-18, `VITE_BASE_PATH`). Requirements served: REQ-020, REQ-057.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. `createHashRouter` (`/macrolens/#/favourites`) | Every route loads `index.html`; works offline from the precache; no 404 tricks | URLs contain `#` |
| B. Browser history routing plus a `404.html` that redirects | Clean URLs | Each deep link costs a 404 and a redirect; a home-screen app opened offline gets no `404.html` from the network, so a deep link fails unless the service worker adds a navigation fallback |

## Decision

We will use React Router's hash router.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Reload and deep links work on GitHub Pages and offline |
| Negative | URLs show `#/`; the home-screen app hides the URL, so the effect is limited to browser tabs |
| Follow-up | None |
