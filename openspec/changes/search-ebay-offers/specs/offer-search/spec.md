# Spec Delta

## Purpose

Lets a user search for a product by name and see a ranked list of offers for it, so they can quickly find the best deal available.

## ADDED Requirements

### Requirement: User can search for a product
The system SHALL accept a free-text product query and return a list of offers for that product, ordered best-first.

#### Scenario: Successful search
- **WHEN** a user submits the query "iPhone 15 128GB"
- **THEN** the system returns a non-empty list of offers ordered best-first
- **AND** each offer shows title, total price, condition, seller or store, and a link to the listing

#### Scenario: Blank query
- **WHEN** a user submits an empty or whitespace-only query
- **THEN** the system rejects the request with a clear validation message
- **AND** no external source is contacted

### Requirement: Search handles no results
The system SHALL tell the user when no offers match their query instead of showing an empty or broken page.

#### Scenario: No matching offers
- **WHEN** a user searches for a query that matches no listings
- **THEN** the system shows a "no offers found" message
- **AND** does not report an error

### Requirement: Search degrades gracefully when the source fails
The system SHALL report a source failure to the user without exposing internal details, and SHALL NOT show partial data as if it were complete.

#### Scenario: Source unavailable
- **WHEN** the price source is unreachable or returns an error and no cached results exist for the query
- **THEN** the system shows a message that offers could not be retrieved right now
- **AND** the message contains no credentials, stack traces, or raw upstream responses

#### Scenario: Source unavailable with cached results
- **WHEN** the price source fails but cached results for the same query exist
- **THEN** the system returns the cached offers
- **AND** indicates when they were last updated

### Requirement: Search results are bounded
The system SHALL return at most a fixed maximum number of offers per search so responses stay fast and the page stays readable.

#### Scenario: Many matching listings
- **WHEN** a query matches more listings than the maximum
- **THEN** the system returns only the top-ranked offers up to the maximum
