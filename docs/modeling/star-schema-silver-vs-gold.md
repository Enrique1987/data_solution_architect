# Star Schema in Silver vs Gold

## Abstract

A star schema can technically be built in either Silver or Gold.

The important question is not **"Which layer is correct?"**, but:

> **What architectural responsibility do we want Silver and Gold to have?**

A useful rule of thumb is:

- **Silver = clean, integrated, reusable data**
- **Gold = consumption-oriented analytical models**

Therefore, a star schema usually fits naturally in **Gold**.

However, building the dimensional model already in **Silver** can make sense when Silver is intended to act as a **central, governed dimensional core** that all downstream teams or spokes must reuse.

The main reason is not simply "reusability".

It is **reusability + governance + consistency**.

---

## 1. Typical Approach: Star Schema in Gold

A common Medallion architecture looks like this:

```text
Bronze
   ↓
Silver
   ├── clean facts
   ├── clean dimensions
   └── integrated business data
           ↓
      ┌────┼─────┐
      ↓    ↓     ↓
   Gold A Gold B Gold C
```

Each Gold layer represents a particular analytical use case.

For example:

```text
Silver
├── flight
├── airline
├── airport
└── country
```

could be reused by:

```text
Gold Marketing
├── fact_flight
├── dim_airline
└── dim_country
```

and:

```text
Gold Operations
├── fact_flight
├── dim_airport
├── dim_airline
└── dim_country
```

This already provides **reusability**.

The fact that several Gold models reuse the same Silver tables does **not by itself justify creating the star schema in Silver**.

---

## 2. Why "Reusability" Alone Is Not Enough

An argument such as:

> "We should build the dimensional model in Silver because it is reusable."

is incomplete.

Silver can already be reusable without being dimensional.

For example:

```text
Silver
├── airline
├── airport
├── country
└── flight
```

can be consumed by many Gold models.

Therefore, the important question is:

> **What exactly becomes more reusable by creating the dimensional relationships already in Silver?**

If there is no concrete answer, "reusability" is mostly an architectural buzzword rather than a real justification.

---

## 3. When a Star Schema in Silver Makes Sense

A stronger reason appears when Silver is not just a clean-data layer, but a **governed enterprise dimensional core**.

For example:

```text
                    HUB
                     │
              Silver Dimensional Core
              ├── fact_flight
              ├── dim_airline
              ├── dim_airport
              └── dim_country
                     │
             ┌───────┼────────┐
             ↓       ↓        ↓
         Gold A   Gold B   Gold C
         Spoke    Spoke    Spoke
```

The Hub defines centrally:

- which dimensions officially exist;
- the grain of the facts;
- relationships between facts and dimensions;
- surrogate keys;
- unknown/default dimension members;
- referential integrity;
- Slowly Changing Dimension logic;
- canonical business definitions.

The Spokes are then **not allowed or expected to reinvent those concepts**.

---

## 4. The Real Benefit: Governance

This is the strongest argument for dimensional modeling in Silver.

Imagine the central model defines:

```text
dim_airline
dim_airport
fact_flight
```

Without centralized governance, one Spoke might decide to create:

```text
dim_airline
```

while another creates:

```text
dim_carrier
```

with slightly different rules.

Another team could create a different airline dimension containing additional attributes or different historical handling.

Eventually:

```text
Marketing airline ≠ Operations airline ≠ Finance airline
```

The dimensional Silver model prevents this.

It effectively says:

> **This is the official Airline dimension.**
>
> **This is the official Airport dimension.**
>
> **This is the official Flight fact.**
> These are their keys and relationships.**

Every downstream model must start from that same semantic foundation.

---

## 5. "Making Sure Everyone Does It Correctly"

A simple way to understand the idea is:

> **The dimensional Silver layer acts as a controlled contract between the Hub and the Spokes.**

Instead of allowing every Spoke to decide:

```text
How do I model Airline?
How do I model Airport?
What is the grain of Flight?
How do I handle missing dimension keys?
How do I implement SCD?
```

the Hub answers those questions once.

The Spokes receive an already governed model.

Therefore:

```text
Silver dimensional modeling
        ↓
less freedom for each Spoke
        ↓
more consistency across the platform
```

This is both an advantage and a trade-off.

---

## 6. Surrogate Keys

Surrogate keys are another example.

If every Gold model generates its own keys, one Spoke could have:

```text
airline_sk = 17
```

while another could have:

```text
airline_sk = 8421
```

This is not necessarily technically wrong.

However, if Silver establishes a common dimensional core:

```text
LH → airline_sk = 42
```

then every downstream model can reuse the same representation.

More importantly, the same central process can manage:

- SCD Type 2;
- unknown members;
- expired records;
- historical versions;
- key consistency.

The real benefit is therefore not merely:

> "We save the calculation of the surrogate key."

The benefit is:

> **We centralize the dimensional lifecycle.**

---

## 7. Referential Integrity

A dimensional Silver layer can also guarantee that every foreign key appearing in a fact has a valid corresponding dimension member.

For example:

```text
fact_flight.airline_sk = 42
```

must exist in:

```text
dim_airline.airline_sk = 42
```

If an airline cannot be resolved, Silver could assign:

```text
airline_sk = -1
```

representing:

```text
Unknown Airline
```

This means downstream Spokes do not need to solve the problem themselves.

---

## 8. Grain

Another important responsibility is defining the grain centrally.

For example:

```text
fact_flight
```

could mean:

> One row per flight + airport + airline + date.

Once that definition is established in the Hub, individual Spokes should not silently interpret the same fact differently.

This reduces ambiguity across the organization.

---

## 9. What Should Still Stay in Gold?

Even with a dimensional Silver core, Gold can still have an important role.

Silver might provide:

```text
fact_flight
dim_airline
dim_airport
dim_country
```

while Gold creates use-case-specific models.

For example:

```text
Gold Marketing
├── fact_flight
├── dim_airline
└── dim_country
```

and:

```text
Gold Operations
├── fact_flight
├── dim_airline
├── dim_airport
└── dim_country
```

Gold can also contain:

- KPIs;
- aggregations;
- reporting logic;
- dashboard-specific calculations;
- business-use-case-specific subsets;
- semantic models.

A useful distinction is therefore:

```text
Silver = canonical dimensional core
Gold   = use-case-specific analytical products
```

---

## 10. Hub & Spoke Changes the Discussion

In a Hub & Spoke architecture, the argument for dimensional modeling in Silver becomes stronger.

Without it:

```text
Hub Silver
    ↓
Spoke A → builds dimensional logic
Spoke B → builds dimensional logic
Spoke C → builds dimensional logic
```

This creates the risk that dimensional logic diverges.

With a dimensional core:

```text
                 HUB
                  │
         Silver Dimensional Core
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Spoke A    Spoke B    Spoke C
```

the Hub owns the common semantics.

This reduces the technical burden on the Spokes and enforces consistency.

---

## 11. The Important Distinction

One of the most useful conclusions is:

> **Centralization does not automatically require dimensional modeling.**

You can centralize:

- clean data;
- dimensions;
- facts;
- business rules;
- master data;

without creating a complete star schema.

Therefore, this architecture is perfectly valid:

```text
Bronze
   ↓
Silver
   ├── canonical clean facts
   └── canonical clean dimensions
            ↓
      Gold Star Schemas
```

The alternative is:

```text
Bronze
   ↓
Silver
   └── canonical dimensional model
            ↓
      Gold use-case models
```

Both are legitimate.

The decision depends on **how much semantic and dimensional governance the Hub should own**.

---

## 12. Decision Framework

A practical decision tree:

```text
Do multiple downstream teams consume the same data?
                │
                ↓
               Yes
                │
                ↓
Do they only need the same clean facts/dimensions?
        │                       │
       Yes                      No
        │                       │
        ↓                       ↓
Keep Silver generic      Do they need the same
Star Schema in Gold      dimensional semantics?
                                │
                               Yes
                                │
                                ↓
                     Dimensional core in Silver
```

Another useful question is:

> **What logic would otherwise have to be implemented repeatedly in the Gold Spokes?**

If the answer is:

```text
Nothing significant.
They just reuse the same Silver tables.
```

then dimensional modeling in Silver adds little value.

If the answer is:

```text
Surrogate keys
SCD
grain
referential integrity
unknown members
conformed dimensions
canonical relationships
```

then there is a strong case for doing that work centrally.

---

## 13. Questions to Ask During an Architecture Discussion

Instead of asking only:

> Why Silver instead of Gold?

ask:

### Reusability

> **What exactly are we reusing more effectively by moving the dimensional model to Silver?**

### Governance

> **Which decisions do we want to prevent individual Spokes from making independently?**

### Technical Complexity

> **Which dimensional logic would otherwise have to be implemented repeatedly in every Gold Spoke?**

### Consumption

> **Will users or systems consume Silver directly?**

If yes, an easy-to-query dimensional structure in Silver may provide additional value.

### Gold Responsibility

> **If the dimensional model moves to Silver, what remains the architectural responsibility of Gold?**

This is important because otherwise Silver and Gold may become conceptually redundant.

---

## 14. Key Takeaway

The question is not:

```text
Star Schema in Silver
        VS
Star Schema in Gold
```

The deeper architectural question is:

> **Where should the canonical business semantics be owned?**

If Silver is intended primarily as:

```text
clean + integrated + reusable data
```

then:

```text
Star Schema → Gold
```

is usually natural.

If Silver is intended as:

```text
central governed dimensional contract
```

then:

```text
Star Schema / Dimensional Core → Silver
```

can be justified.

The strongest argument for the latter is not simply **reusability**.

It is:

> **Centralized dimensional governance so that every downstream Spoke uses the same facts, dimensions, grain, keys, history and relationships instead of reinventing them independently.**
