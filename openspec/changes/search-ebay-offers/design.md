# Design

## Context

The repository has no code yet. The stack is decided: an F# recommendation engine, a C# API for everything else, React for the UI, and SQLite for storage. Motivation and scope are in `proposal.md`; requirements are in the three specs under `specs/`.

Constraints that shape the approach:
- Both languages run on .NET, so the engine can be an in-process library rather than a separate service.
- eBay's default limit is 5,000 Browse calls per day, and it returns active listings only.
- SQLite allows one writer at a time. This change has a single writer (the API), so that is not yet a constraint.
- Alerts, a background worker, and more stores come later and should not require reworking what is built here.

## Goals / Non-Goals

**Goals:**
- Keep the F# engine pure: listings in, ranked offers out, no I/O, so it is easy to test.
- Keep the store behind one interface so a second source is an additive change.
- Make the data layer easy to move from SQLite to Postgres later.

**Non-Goals:**
- No background worker, scheduler, or email.
- No user accounts or per-user state.
- No price history views. Listings are stored, but nothing reads history yet.

## Decisions

**1. Solution layout: one C# API project, one F# engine library, one React app.**
The API references the engine project directly. Alternative: run the engine as its own service. Rejected because it adds a network hop, deployment, and failure mode with no benefit at this scale. The engine's public surface stays small and data-only, so it can be split out later.

**2. The engine is pure functions over immutable records.**
It exposes roughly `rank : SearchContext -> Listing list -> Offer list`, where condition filtering, product matching, total-cost computation, and tie-breaking all live. No database, HTTP, or clock access (the current time, if needed, is passed in). Alternative: put ranking in C#. Rejected because this logic is the product's core value and is the best fit for F# discriminated unions (condition, listing type, shipping known or unknown).

**3. Price sources sit behind a C# interface returning a common listing shape.**
An `IPriceSource` takes a query and returns normalized listings. The eBay adapter is the only implementation. It maps eBay's fields into the common shape and nothing downstream sees eBay-specific types. Alternative: call eBay directly from the endpoint. Rejected because it would leak eBay's schema into the engine and make a second store a rewrite.

**4. eBay access uses the Browse API with application (client-credentials) OAuth.**
No user login is needed for searching public listings. The token is fetched and refreshed server-side and held in memory. Credentials come from environment variables or user secrets, never the repository, and never reach the browser. Development uses eBay's sandbox where practical.

**5. Caching and call budget live in SQLite.**
Searches are keyed by a normalized query string (lowercased, trimmed, whitespace collapsed) and stored with a fetched-at timestamp. A repeat search within the TTL is served from the cache. A small counter table tracks eBay calls per UTC day so the system stops calling once the daily budget is reached and falls back to cache. A short TTL (on the order of minutes to an hour, configurable) balances freshness against the call limit. Alternative: in-memory cache only. Rejected because it loses data on restart and gives alerts and history nothing to build on.

**6. Data access uses EF Core, behind a repository layer.**
SQLite runs in WAL mode with a busy timeout. Raw source payloads are stored as JSON text so they can be re-parsed when matching improves. Queries stay in the repository layer so a later move to Postgres is mostly a provider change plus a check of JSON and text-search queries. Alternative: raw SQL with Dapper. Rejected to keep migrations and a future provider swap simple.

**7. Default ranking policy: new items, fixed price, total cost ascending.**
Used, refurbished, and unknown-condition listings and auctions are excluded from the default ranking, matching the specs. A condition word in the query ("used", "refurbished") switches the condition filter. Ties break on seller rating, then listing id, so ordering is stable.

**8. React talks to one JSON endpoint, `GET /api/offers?q=...`.**
Responses contain offers plus metadata (cached or fresh, last updated, whether the source was unavailable). In development, the front end is proxied to the API to avoid CORS configuration; production CORS is restricted to the site's own origin. The page shows loading, empty, and error states explicitly.

## Risks / Trade-offs

- **[eBay terms may restrict production use of the Buy APIs]** → Build and test on the free tier now. Before any public launch, read eBay's license terms and request approval if needed. If approval is refused, the price-source interface lets a different source replace eBay without touching the engine or UI.
- **[5,000 calls per day could be exhausted by traffic]** → Caching, the daily counter, and graceful fallback to cached results or a clear "temporarily unavailable" message.
- **[Product matching is imperfect: keyword search returns accessories and unrelated items]** → Matching rules live in the engine with example-based tests, so they can be improved without touching the API or UI. Expect to tune them against real queries.
- **[Cached prices can be stale]** → Show last-updated time in results and keep the TTL short and configurable.
- **[eBay returns active listings only, no sold history]** → Out of scope here. Any "is this a good price" feature will need its own stored history or another source.
- **[SQLite has one writer]** → Acceptable for a single API process. Revisit when the alert worker arrives.

## Open Questions

- Exact cache TTL and result cap values. These are configuration and can be tuned after seeing real usage.
- Whether to show auctions as a labeled secondary section or hide them entirely in the first UI.
