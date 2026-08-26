---
name: Query NeuroTherapia public site content over MCP
description: Retrieve NeuroTherapia's business details and search its public website content (pipeline, leadership, press releases, policies) through the anonymous Model Context Protocol server the company serves from its own domain.
api: mcp/neurotherapia-mcp.yml
endpoint: https://www.neurotherapia.com/_api/mcp
operations:
  - GetBusinessDetails
  - SearchInSite
  - GenerateVisitorToken
  - CallWixSiteAPI
---

# Query NeuroTherapia public site content over MCP

## What this surface is — and is not

NeuroTherapia, Inc. is a clinical-stage biotechnology company. **It ships no product API.** There is no
therapeutics, pipeline, clinical-trial or NTRX-07 data API, and no OpenAPI document exists anywhere on
its domain.

What does exist is a Model Context Protocol server at `https://www.neurotherapia.com/_api/mcp`,
provisioned by the Wix website platform and advertised by the company's own `/llms.txt`. It exposes the
**public website content** — nothing more. Do not represent anything retrieved here as clinical,
regulatory or trial data.

## Connect

No authentication is required to connect or to list tools.

```
POST https://www.neurotherapia.com/_api/mcp
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":1,"method":"tools/list"}
```

Verified 2026-08-26: HTTP 200, nine tools returned. Always issue `tools/list` first — the platform may
change the tool set, and the provider's `llms.txt` explicitly asks clients to re-list on a tool-update
notification. There is no version number to pin to.

## Steps

1. **`GetBusinessDetails`** — no arguments. Call this first. It returns the company's timezone, contact
   email, phone, and physical address, plus a one-line business description and the list of site
   capabilities currently installed (a Blog app, as of 2026-08-26). Use it to answer any contact,
   location or "what does this company do" question without a second call.

2. **`SearchInSite`** — argument `searchTerm` (required). Use this for content questions: the NTRX-07
   pipeline, Phase 2a results, leadership, board, investors, press releases, the FCOI policy. Prefer one
   specific term per call.

3. **`SearchSiteApiDocs`** — argument `searchTerm` (required). Use this *instead of* `SearchInSite` when
   the question is about products or services offered through the site's own APIs. On this site that
   surface is thin, so expect little back.

4. **`GenerateVisitorToken`** — no arguments. Only needed before a call in step 5. The token is minted
   anonymously; no account exists and none can be created.

5. **`CallWixSiteAPI`** — arguments `visitorToken`, `url`, `method` (all required), `body` (optional).
   **Read-only use only on this site.** Discover the exact method URL with `SearchSiteApiDocs` first.

## Rules

- **Do not mutate.** `CallWixSiteAPI` accepts any HTTP method and `ExecuteWixAPI` runs arbitrary
  JavaScript against the Wix REST API. Nothing on this surface is reversible: no cancel, undo, restore or
  rollback operation exists and no reversal window is published
  (`conventions/neurotherapia-conventions.yml`). Treat every write as permanent, and do not attempt one.
- **No retries on ambiguity.** There is no idempotency key on any tool, so a repeated write-capable call
  can duplicate its effect.
- **No backoff signal.** The endpoint returns no `RateLimit-*` or `Retry-After` headers
  (`rate-limits/neurotherapia-rate-limits.yml`). Pace calls conservatively and treat a non-200 as a stop,
  not as a retry cue.
- **Errors are JSON-RPC error objects**, not RFC 9457 problem details. No error catalog is published.
- **Attribute correctly.** The MCP server is Wix platform infrastructure served on NeuroTherapia's
  domain. It is not a NeuroTherapia engineering artifact.
