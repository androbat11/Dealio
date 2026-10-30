# Proposal

## Why

Dealio's purpose is to help people find the best online deal for a product. Search with ranked offers is the core value and the foundation for everything later (price alerts, history, more stores). It needs no accounts and no email, so it is the smallest slice that exercises every layer: React, the C# API, the F# recommendation engine, and a real price source.

eBay is the first source because its Browse API is free, official, and available today. More stores follow later through the same interface.

## What Changes

- A user types a product into a React search box and sees a list of offers, ranked best-first.
- A C# API endpoint accepts the query, retrieves listings from a price source, runs them through the F# engine, and returns ranked offers.
- The F# engine (a class library referenced by the C# API) normalizes listings, filters out unsuitable ones, and ranks the rest.
- A price-source interface in C# hides where listings come from. The first and only implementation is eBay's Browse API.
- Listings and responses are cached in SQLite so repeated searches do not spend the eBay call budget, and so later features (alerts, price history) have data to build on.
- Offers are limited to what the engine can compare fairly: condition and listing type (fixed price versus auction) are modeled from the start.

Out of scope: user accounts, price alerts and email, price history views, any store other than eBay, and a "this is a good deal" verdict.

## Capabilities

### New Capabilities
- `offer-search`: A user submits a product query and receives a list of ranked offers, including empty-result and failure behavior.
- `offer-ranking`: How the engine normalizes, filters, and orders listings into offers (condition, listing type, total cost).
- `price-sourcing`: How listings are obtained from an external store through a swappable source, with eBay as the first, including caching and call-limit behavior.

### Modified Capabilities
<!-- None: the project has no existing specs. -->

## Impact

- **New code**: a C# ASP.NET API, an F# engine library, an eBay source adapter, a SQLite store (EF Core), and a React front end. The repository currently contains no code.
- **External dependency**: eBay Developer Program keyset. Default limit is 5,000 Browse calls per day.
- **Risk to confirm before launch**: eBay's terms state that Buy APIs need an additional license and that production use is restricted. Building and testing against the free tier is fine, but a public launch may need eBay's approval. This is recorded in the design as an open question.
- **Later changes depend on this one**: alerts (needs a background worker and email), additional sources, and price history reuse the source interface and the SQLite cache introduced here.
