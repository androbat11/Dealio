# Tasks

Defaults assumed (configurable, from the design's open questions): cache TTL 30 minutes, result cap 20 offers, auctions hidden in the first UI.

## 1. Solution scaffolding

- [ ] 1.1 Create the .NET solution with a C# ASP.NET API project, an F# engine class library, and test projects for each; verify `dotnet build` succeeds for the whole solution
- [ ] 1.2 Reference the F# engine from the API project; verify the API compiles while calling a placeholder engine function
- [ ] 1.3 Create the React app (Vite) with a dev proxy to the API; verify `npm run dev` serves a page that can reach an API health endpoint
- [ ] 1.4 Add `.gitignore` entries for build output, the SQLite file, and secrets, and document how to run both apps in a README; verify a fresh clone follows the README and starts both

## 2. F# engine: listing model and ranking

- [ ] 2.1 Define the common listing and offer types (condition, listing type, shipping known/unknown, price, currency, seller, URL); verify the project compiles and the types are usable from C#
- [ ] 2.2 Implement total-cost ranking with shipping handling and stable tie-breaking (seller rating, then listing id); verify unit tests for the shipping-changes-order, unknown-shipping, and equal-cost scenarios in `offer-ranking` pass
- [ ] 2.3 Implement condition filtering with "new" as the default and query keywords ("used", "refurbished") overriding it; verify tests cover the default and keyword scenarios
- [ ] 2.4 Exclude auctions from the default ranking; verify a test shows an auction with a current bid never appears as a fixed price
- [ ] 2.5 Implement product matching that drops accessories and unrelated keyword matches, with an example-based test table (for example "iPhone 15" versus a case, charger, and screen protector); verify the table passes and add a case whenever a miss is found
- [ ] 2.6 Apply the result cap to the ranked list; verify a test with more listings than the cap returns only the top-ranked offers

## 3. Storage and caching

- [ ] 3.1 Add EF Core with SQLite in WAL mode and a busy timeout, and a repository layer for cached searches and listings; verify an integration test round-trips a search and its listings
- [ ] 3.2 Store the raw source payload as JSON text alongside each normalized listing; verify a test reads the payload back unchanged
- [ ] 3.3 Implement query normalization (trim, lowercase, collapse whitespace) and TTL-based cache lookup; verify tests for cache hit, cache expiry, and equivalent queries mapping to one entry pass
- [ ] 3.4 Add a per-UTC-day eBay call counter with a configurable limit; verify a test shows no further calls are permitted once the limit is reached and the counter resets the next day

## 4. eBay price source

- [ ] 4.1 Define the `IPriceSource` interface in C# and register it with dependency injection; verify the API resolves it and a fake source can be substituted in tests
- [ ] 4.2 Implement client-credentials OAuth against eBay with in-memory token reuse and refresh; verify against the eBay sandbox that a token is obtained and reused until near expiry
- [ ] 4.3 Implement the eBay Browse search adapter mapping eBay fields (title, price, currency, shipping, condition, listing type, seller, URL) into the common listing; verify a test using recorded eBay responses maps every field and handles missing shipping
- [ ] 4.4 Load credentials from environment variables or user secrets only; verify a test or log check shows no credential appears in responses or logs, and that no secret is committed
- [ ] 4.5 Handle eBay errors, timeouts, and rate-limit responses by raising a typed source failure; verify tests for each failure mode

## 5. Search endpoint

- [ ] 5.1 Implement `GET /api/offers?q=` that validates the query, checks the cache and call budget, calls the source, runs the engine, stores results, and returns offers plus metadata (cached or fresh, last updated, source unavailable); verify integration tests using a fake source cover the successful search and blank-query scenarios in `offer-search`
- [ ] 5.2 Return cached offers with their last-updated time when the source fails or the budget is exhausted, and a generic "temporarily unavailable" error otherwise; verify tests for both paths and that error bodies contain no internal details
- [ ] 5.3 Return an explicit empty result, not an error, when nothing matches; verify the no-results scenario test passes

## 6. React UI

- [ ] 6.1 Build the search box and offers list showing title, total price, condition, seller, and a link to the listing; verify a component test renders offers from a sample response
- [ ] 6.2 Add loading, empty ("no offers found"), blank-query, and error states, and show the last-updated time for cached results; verify component tests for each state
- [ ] 6.3 Verify end to end against the eBay sandbox or recorded responses: search "iPhone 15 128GB" in the browser and confirm ranked offers appear with working listing links

## 7. Integration checks

- [ ] 7.1 Run the full backend and front-end test suites; verify all pass from a clean checkout
- [ ] 7.2 Walk through every scenario in `offer-search`, `offer-ranking`, and `price-sourcing` and confirm each is covered by a passing test or a manual check noted in the README
- [ ] 7.3 Record the eBay licensing check in the README (what the terms say about production use of the Browse API) so the launch decision is explicit

## Workflow follow-up

- Archive the change after review requirements are satisfied.
- Verify the archived result.
