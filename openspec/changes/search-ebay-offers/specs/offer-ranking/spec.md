# Spec Delta

## Purpose

Defines how raw listings from a price source are filtered and ordered into offers, so that the best deal appears first and unsuitable listings never mislead the user.

## ADDED Requirements

### Requirement: Offers are ranked by total cost
The system SHALL order offers by total cost ascending, where total cost is item price plus shipping when shipping is known.

#### Scenario: Shipping changes the order
- **WHEN** one listing costs 100.00 with 20.00 shipping and another costs 110.00 with free shipping
- **THEN** the 110.00 listing is ranked above the 100.00 plus shipping listing

#### Scenario: Unknown shipping
- **WHEN** a listing has no shipping information
- **THEN** the system ranks it using item price only
- **AND** marks the shipping cost as unknown in the offer

### Requirement: Listings are matched to the query
The system SHALL exclude listings that are not the searched product, such as accessories, parts, or unrelated items that only share keywords.

#### Scenario: Accessory excluded
- **WHEN** the query is "iPhone 15" and a listing is a phone case for the iPhone 15
- **THEN** the case is not included in the offers

### Requirement: Condition is modeled and filtered
The system SHALL record each offer's condition (new, used, refurbished, or unknown) and SHALL by default rank only offers whose condition matches what the query implies, defaulting to new.

#### Scenario: Default excludes used items
- **WHEN** a user searches without specifying a condition
- **THEN** used and unknown-condition listings are not ranked above new listings
- **AND** each offer shows its condition

#### Scenario: Condition named in the query
- **WHEN** the query contains "used" or "refurbished"
- **THEN** the system ranks offers of that condition

### Requirement: Auctions are not ranked as fixed prices
The system SHALL distinguish fixed-price listings from auctions and SHALL NOT present an auction's current bid as a final price.

#### Scenario: Auction listing
- **WHEN** a listing is an auction with a current bid
- **THEN** the system excludes it from the default ranking
- **AND** if shown, labels it as an auction with a current bid rather than a buy-now price

### Requirement: Ranking is deterministic
The system SHALL produce the same ordering for the same input listings, breaking ties in a stable, documented way.

#### Scenario: Equal total cost
- **WHEN** two offers have the same total cost
- **THEN** the system orders them consistently on every run
