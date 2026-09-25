# Dimensional Modeling Interview Case: Refactoring a Wide Fact Table

## Scenario

A project receives data from SAP through Bronze, Silver, and Gold before it is consumed in Power BI. The dimensional model already exists, but its Fact contains many columns, including descriptive attributes that may belong elsewhere.

The correct response is not to remove columns simply because the Fact is wide. The review must establish what each row and column means and whether the model preserves the business semantics.

## Start with the grain

Ask:

> **What exactly does one row in this Fact represent?**

Valid answers are business statements such as:

- one operated flight;
- one passenger on one flight;
- one booking;
- one invoice line;
- one airport operation.

Every measure must make sense at exactly that level. If one row represents one flight, `PassengerCount`, `DelayMinutes`, and `CargoWeightKg` can be flight-level measures. `PassengerAge` cannot: one flight can contain many passenger ages.

An attribute can be compatible with the Fact grain and still belong in a Dimension. `AircraftType` may have one value per flight without breaking the grain, but it describes the aircraft rather than measuring the flight.

See the [Fact, Dimension, and Grain mental model](modelo-dimensional-marco-mental.md) for the underlying distinction.

## Classify every column before moving anything

Use the following inventory:

| Category | Question | Typical treatment |
| --- | --- | --- |
| Measure | What quantitative observation exists at the declared grain? | Keep in the Fact and define aggregation behavior. |
| Foreign key | Which dimension member gives context to the event? | Keep the key in the Fact. |
| Descriptive attribute | Which business entity or classification does it describe? | Place it in an existing or justified new Dimension. |
| Degenerate dimension | Is it a business identifier without useful attributes of its own? | Keep the identifier in the Fact. |
| Technical/audit field | Does it support ingestion, lineage, or reconciliation? | Retain in the appropriate engineering layer; expose to Gold only when useful. |
| Unknown | Is the functional meaning or grain unclear? | Obtain business clarification before remodeling. |

Calculated does not mean measure. `DelayMinutes = 23` is a measure; a calculated `PunctualityStatus = Delayed` is a descriptive classification.

## Reuse existing Dimensions first

For each descriptive attribute, ask:

> **If I know the dimension member, can I determine this attribute?**

If Airline determines `AirlineName`, `IATACode`, `ICAOCode`, and `AirlineCountry`, those attributes belong naturally in `DimAirline`. The Fact should normally reference the airline member through a key.

This test is evidence for placement, but historical variation must also be considered. If an attribute changes over time and reports need the value valid when the Fact occurred, the dimension may require SCD handling.

## Propose missing Dimensions by business concept

Remaining descriptive columns may reveal a missing concept:

```text
AircraftManufacturer
AircraftModel
AircraftFamily
AircraftCategory
```

These can form `DimAircraft` when the attributes describe a coherent entity. Likewise, disruption category, reason, and group may form a disruption dimension.

The goal is to model business concepts, not to minimize the Fact's column count.

## Junk Dimensions

A Junk Dimension groups related flags, indicators, statuses, and other small classifications with low combined cardinality.

For three binary flags, the maximum domain is eight combinations:

```text
IsCancelled × IsCodeshare × IsDomestic
2 × 2 × 2 = 8
```

The Fact keeps the same number of rows and stores one key for the observed combination. A Junk Dimension is not a container for unrelated leftovers. Evaluate:

- whether the attributes are semantically suitable together;
- individual and combined cardinality;
- whether combinations should be generated exhaustively or only when observed;
- how unknown, missing, and source-null values differ;
- whether consumers benefit from the grouping.

## Degenerate Dimensions

Business identifiers such as invoice, booking, ticket, order, or transaction numbers may remain directly in the Fact when they have no descriptive attributes requiring a separate table.

```text
FactBooking
-----------
DateKey
CustomerKey
AirlineKey
BookingNumber
PassengerCount
Revenue
```

`BookingNumber` is a Degenerate Dimension. It is not the same as a surrogate key: a surrogate key is an artificial identifier for a dimension member; a degenerate dimension is a business identifier stored in the Fact.

## Technical columns

Fields such as `ingestion_timestamp`, `batch_id`, `source_system`, `load_id`, `record_hash`, `created_at`, and `updated_at` support engineering and auditability. They may be essential in Bronze or Silver without belonging in the consumer-facing Gold model.

Review them separately from business measures and attributes. Removing them from Gold must not break lineage, reconciliation, replay, or operational support requirements.

## The source schema is not the analytical model

An SAP table with 150 columns reflects the operational source's requirements. It does not prove that the analytical model should be one 150-column Fact. Review all inherited columns, including those already present before Gold transformations; reusable entities and classifications may be embedded in the source structure.

At the same time, a wide Fact is not automatically wrong. It may legitimately contain many measures, keys, degenerate identifiers, and audit fields at a consistent grain.

## Silver and Gold responsibilities

Reusable entities such as Airline, Airport, Country, and Aircraft can be harmonized in Silver as conformed Dimensions. Gold can then assemble consumer-oriented star schemas.

If Silver is intended to be a governed dimensional core, it can also own grain, surrogate keys, unknown members, referential integrity, SCD logic, and canonical relationships. If Silver provides clean reusable entities without that contract, Gold is the natural place for use-case-specific star schemas.

See [Star Schema in Silver vs Gold](star-schema-silver-vs-gold.md) for the full decision framework.

## Other patterns to identify

### Role-playing Dimensions

The same physical Dimension can appear in multiple semantic roles:

- `DimAirport` as origin and destination airport;
- `DimDate` as scheduled, actual, and booking date.

The role belongs to the relationship with the Fact, not to a duplicate business definition.

### Slowly Changing Dimensions

If an airline changes alliance and reports must retain the alliance valid when a historical Fact occurred, SCD Type 2 can create a new dimension version with a new surrogate key and validity interval. Facts continue pointing to the version that was valid for their event.

### Grain and normalization

Grain is the semantic level represented by one Fact row. Second Normal Form concerns functional dependencies in relational design. The ideas can interact, but a normalized table can still have an unsuitable analytical grain.

For dimension-to-dimension links and many-to-many relationships, see [Snowflakes, Outriggers, and Bridge Tables](snowflakes-outriggers-and-bridge-tables.md).

## Complete review workflow

1. State why the current model is suspected to be wrong. A high column count alone is not sufficient evidence.
2. Declare the Fact grain as a business sentence and confirm it with the data owner.
3. Test keys and duplicates against the declared grain; uniqueness alone does not define the grain.
4. Classify every column as a measure, foreign key, descriptive attribute, degenerate dimension, technical field, or unresolved item.
5. Verify that every value is defined at the Fact grain; separate mixed levels of detail.
6. Move descriptive attributes to existing Dimensions when their dependency and history support it.
7. Propose a new Dimension only when remaining attributes form a meaningful business concept.
8. Evaluate degenerate and Junk Dimensions where their patterns apply.
9. Validate measures, including their definitions and additive, semi-additive, or non-additive behavior.
10. Decide which dimensional responsibilities belong in Silver and which belong in Gold.
11. Reconcile row counts, measures, null semantics, and historical results before changing consumers.

## Compact interview answer

> I would not start by removing columns because the Fact is wide. First, I would identify and validate the Fact grain and confirm that all data is represented at a consistent level of detail. I would then classify columns into measures, keys, descriptive attributes, technical fields, and business identifiers. Descriptive attributes should first be tested against existing Dimensions; remaining coherent concepts may justify new Dimensions. Transaction identifiers may remain as degenerate dimensions, while related low-cardinality flags can be evaluated as a Junk Dimension. Finally, I would validate every measure at the declared grain, define its aggregation behavior, preserve reconciliation controls, and decide whether reusable dimensional semantics should be governed in Silver or assembled in Gold.

## Principle to remember

The objective is not to make the Fact as small as possible. It is to make the model semantically correct.

A Fact with many legitimate measures can be valid. A Fact with few columns can still be wrong when it mixes grains or stores descriptive attributes without coherent semantics.

## References

- [Kimball: The 10 Essential Rules of Dimensional Modeling](https://www.kimballgroup.com/2009/05/the-10-essential-rules-of-dimensional-modeling/)
- [Kimball: Junk Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/junk-dimension/)
- [Kimball: Degenerate Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/degenerate-dimension/)
