# ADR-0009: Open Food Facts identification from a browser

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

Open Food Facts asks every app to send `User-Agent: AppName/Version (ContactEmail)`. Pages cannot set `User-Agent` in all browsers. OFF's CORS allow-list includes `X-User-Agent`, and a request with it passed in Chromium and WebKit (`docs/spikes.md` T0.2). The repository is public (ASM-011), so a contact email in the bundle would publish Zac's address. Requirements served: REQ-034, ASM-012, ASM-014.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Header `X-User-Agent: Macrolens/<APP_VERSION> (https://github.com/Zac80085/macrolens)` | Identifies the app and a contact route without publishing an email; allowed by CORS | Not the documented header name (ASM-014) |
| B. `User-Agent` header | Documented form | Ignored or rejected by browsers; adds a non-standard preflight on some engines |
| C. `X-User-Agent` with an email address | Matches the documented contact form | Publishes Zac's email in a public bundle |
| D. No identification | Nothing to maintain | Risk of being treated as a bot (OFF documentation) |

## Decision

We will send `X-User-Agent: Macrolens/<APP_VERSION> (https://github.com/Zac80085/macrolens)` (`OFF_USER_AGENT`, `docs/design.md` Section 6.3) on every OFF request, where `<APP_VERSION>` is the app version from `package.json`.

## Consequences

| Type | Consequence |
|---|---|
| Positive | OFF can identify the app and reach the owner through the repository |
| Negative | If OFF stops honouring `X-User-Agent`, requests may be throttled (`OFF_RATE_LIMITED`) |
| Follow-up | Claude Code checks for OFF 403 or 429 responses during Phase 8 live tests and UAT (ASM-014) |
