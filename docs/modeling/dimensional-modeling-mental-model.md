# Dimensional Modeling: A Mental Model for Distinguishing Dimensions and Facts

## The core idea

When deciding whether something is a **Dimension** or a **Fact**, start with three questions:

> **Dimension = what is something?**<br>
> **Fact = what happened?**<br>
> **Grain = what exactly does one row represent?**

This is a guide, not a substitute for business analysis. The final classification depends on the functional meaning of the data and the required level of detail.

[Open the visual cheat sheet: Fact Table vs Dimension Table vs Factless Fact](../../assets/images/fact-vs-dimension-vs-factless-fact.png)

## Dimension: “what is something?”

A Dimension describes an entity, classification, or context. For example:

- Airline: which airline it is;
- Airport: which airport it is;
- Country: which country it is;
- Aircraft Type: which type of aircraft it is;
- Date: which date it is and which calendar properties apply.

Its attributes describe, group, and filter the entity: name, code, country, category, description, or group.

A Dimension does not need to be immutable. An airline can change its name or alliance and remain a Dimension. When historical truth must be preserved, a technique such as **Slowly Changing Dimension Type 2** can be used.

The rule is not “data that never changes.” The better rule is:

> **A Dimension describes what something is in the analytical context.**

## Fact: “what happened?”

A Fact represents an occurrence, transaction, or event:

- a flight movement;
- a sale;
- a booking;
- a call;
- a shipment;
- an incident;
- a strike recorded on a date.

A Fact usually relates to several Dimensions that explain who, what, where, when, and how the event is classified.

```text
                  dim_date
                     |
dim_airline ---> fact_flight <--- dim_airport
                     |
                dim_aircraft
```

A Fact can contain measures, foreign keys, business identifiers, and required technical attributes. The number of columns alone does not determine whether the design is correct.

## Grain: the first decision for a Fact

Before designing or reviewing a Fact, complete this sentence:

> **One row represents...**

Examples:

- one row = one flight movement;
- one row = one sales line;
- one row = one booking;
- one row = one event;
- one row = one event per day.

The grain must be expressed as a business statement. Finding a technically unique combination of columns is not enough.

> **Technical key ≠ functional grain.**

After declaring the grain, validate every column against it. Passenger age does not fit a Fact whose grain is one flight because a flight can include many passengers. Aircraft type can be defined at flight grain, although it may still belong as an attribute in a Dimension.

## Factless Fact

A **Factless Fact** records that something occurred or that a relationship existed, even when there is no numeric measure.

```text
date        event
2026-01-10  strike
2026-02-03  storm
```

It does not need passengers, currency, weight, or duration to be a fact. Its value is in recording the occurrence and relating it to its Dimensions.

## Applying it to the Event Tracker

The Event Tracker contains date-related occurrences such as strikes, weather, major events, IT problems, war, drones, or ATC disruption. Conceptually, it answers “what happened?” more directly than “what is something?”, making it a candidate for a **Factless Fact**.

The decision should not be finalized without confirming the functional grain with the business owner. One row might represent:

- one event;
- one event on a specific day;
- one contextual annotation associated with a date.

Knowing columns such as `datum`, `lfd_nummer`, `kategorie`, `radius`, or `ereignis_effekt` does not by itself establish what one row means, whether an event can span several days, or whether multiple events can occur on the same day.

## Decision sequence

1. Ask whether the table describes an entity or context. If it does, it is probably a Dimension.
2. Ask whether it records something that happened. If it does, it is probably a Fact.
3. If it is a Fact, state in one sentence what a row represents.
4. Validate that every measure and relationship exists at that same grain.
5. Consult the business owner when the meaning of a row or key has not been established.

## Related reading

- [Reviewing a wide Fact table](dimensional-modeling-interview-case.md): a process for classifying columns and redesigning a Fact without reducing it arbitrarily.
- [Star Schema in Silver vs Gold](star-schema-silver-vs-gold.md): where grain, keys, history, and canonical relationships should live.
- [Snowflakes, Outriggers, and Bridge Tables](snowflakes-outriggers-and-bridge-tables.md): patterns for relationships that do not fit a simple star.
