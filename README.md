# Data Solution Architect

Personal knowledge base for data architecture, data modeling and related technical concepts.

This repository is currently in its **organization phase**: the existing material is being classified and made easier to navigate before new topics are added.

## Start here

| Area | Document | Current material |
| --- | --- | --- |
| Architecture | [Deciphering Data Architectures](docs/architecture/deciphering-data-architectures.md) | Book notes covering architecture foundations, design sessions and common architecture patterns |
| Data modeling | [Cafeteria POS data modeling](docs/modeling/cafeteria-pos-data-modeling.md) | Operational modeling, normalization and dimensional modeling in one worked example |
| Data modeling | [Snowflakes, outriggers, and bridge tables](docs/modeling/snowflakes-outriggers-and-bridge-tables.md) | Dimensional-modeling patterns for normalized dimensions, dimension-to-dimension links, and complex relationships |
| Technical concepts | [Technical data concepts](docs/concepts/technical-data-concepts.md) | Explanations and cheat sheets collected at different levels of detail |

## Knowledge index

### Architecture and platforms

- [Data architecture conceptual overview](docs/concepts/technical-data-concepts.md#data-architecture-conceptual-overview)
- [Data Mesh](docs/concepts/technical-data-concepts.md#data-mesh)
- [Data Hub](docs/concepts/technical-data-concepts.md#data-hub)
- [Data Lake](docs/concepts/technical-data-concepts.md#data-lake)
- [Data Lakehouse](docs/concepts/technical-data-concepts.md#data-lakehouse)
- [Architecture design sessions](docs/architecture/deciphering-data-architectures.md#the-architecture-design-session)

### Modeling and consumption

- [Operational and dimensional data modeling](docs/modeling/cafeteria-pos-data-modeling.md)
- [Snowflakes, outriggers, and bridge tables](docs/modeling/snowflakes-outriggers-and-bridge-tables.md)
- [Semantic Layer](docs/concepts/technical-data-concepts.md#semantic-layer)
- [Slowly Changing Dimensions](docs/concepts/technical-data-concepts.md#slowly-changing-dimensions-scd)

### Engineering practices

- [Smoke Testing](docs/concepts/technical-data-concepts.md#smoke-testing)

## Repository structure

```text
.
|-- assets/
|   `-- images/
|-- docs/
|   |-- architecture/
|   |-- concepts/
|   `-- modeling/
`-- README.md
```

## Related knowledge base

The implementation-oriented material is maintained in the [Data Engineering repository](https://github.com/Enrique1987/data-engineering).

> These are personal learning notes. Their organization and depth will continue to evolve as existing material is reviewed.
