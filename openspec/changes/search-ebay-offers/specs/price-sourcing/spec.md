# Spec Delta

## Purpose

Defines how the system obtains product listings from external stores, starting with eBay, while respecting provider limits and keeping the rest of the system independent of any one store.

## ADDED Requirements

### Requirement: Listings come from eBay
The system SHALL retrieve product listings for a query from eBay's official Browse API.

#### Scenario: Query is sent to eBay
- **WHEN** a search needs listings and no fresh cached results exist
- **THEN** the system requests matching active listings from eBay
- **AND** each returned listing carries title, price, currency, shipping, condition, listing type, seller, and a URL

### Requirement: Sources are interchangeable
The system SHALL treat every store as a source that returns listings in one common form, so adding another store does not change how offers are ranked or displayed.

#### Scenario: Ranking is source-agnostic
- **WHEN** listings from any source are given for ranking
- **THEN** the ranking behavior depends only on the common listing fields and not on which store they came from

### Requirement: Repeated searches use cached listings
The system SHALL cache retrieved listings for a query for a limited time and SHALL serve repeat searches within that time from the cache without contacting the source.

#### Scenario: Cache hit
- **WHEN** the same query is searched again within the cache lifetime
- **THEN** the system returns results without calling eBay

#### Scenario: Cache expired
- **WHEN** the same query is searched after the cache lifetime has passed
- **THEN** the system retrieves fresh listings from eBay and updates the cache

### Requirement: Provider call limits are respected
The system SHALL stay within the source's published call limits and SHALL NOT make calls once the daily limit is reached.

#### Scenario: Daily limit reached
- **WHEN** the day's call budget for eBay is exhausted
- **THEN** the system makes no further eBay calls that day
- **AND** serves cached results where available, otherwise reports that offers are temporarily unavailable

### Requirement: Credentials are never exposed
The system MUST keep source credentials out of client responses, logs, and the repository.

#### Scenario: Credentials stay server-side
- **WHEN** any search request is handled
- **THEN** no eBay credential appears in a response to the browser or in application logs
