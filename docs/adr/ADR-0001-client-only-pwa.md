# ADR-0001: Client-only PWA with no backend

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer), for Zac |

## Context

Zac wants a phone app without the App Store (S-02), data on the phone only (S-09), and his own Gemini key stored on the phone (S-10). OQ-001 asked whether Gemini, USDA and Open Food Facts accept browser requests; the Phase 0 spike resolved it: all 3 return CORS headers and were readable from Chromium and WebKit (`docs/spikes.md` T0.2). Requirements served: REQ-020, REQ-051, REQ-057, NFR-008.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Static PWA calling the 3 APIs directly from the browser | No server to run or pay for; GitHub Pages hosting (S-18); keys never leave the phone except to their own API; works offline for local features | API keys sit in IndexedDB on the device; each API's browser policy can change; no server-side rate limiting |
| B. PWA plus a serverless proxy (Cloudflare Worker or Vercel Function) | Keys could live server-side; one place to adapt to API changes | Adds an always-on service, a deploy target and a secret store; data passes through a third host; contradicts S-09 and S-10 |

## Decision

We will build Macrolens as a static PWA served from GitHub Pages that calls Gemini, USDA FoodData Central and Open Food Facts directly from the browser, with no backend.

## Consequences

| Type | Consequence |
|---|---|
| Positive | No running cost; no server attack surface; the app keeps working offline for logging, search and favourites (REQ-057) |
| Negative | API keys are stored in plain IndexedDB on the phone; anyone with the unlocked phone and dev tools can read them |
| Negative | A CORS policy change by any provider breaks that feature until a proxy is added; ADR-0010 would then be written |
| Follow-up | Claude Code re-runs the CORS probe as live smoke tests TC-090 to TC-092 in Phase 8 |
