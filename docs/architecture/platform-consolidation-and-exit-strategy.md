# Platform Consolidation and Exit Strategy

## The decision to make

Modern platforms increasingly cover storage, processing, governance, serving, streaming, and operational data. The architecture question is not whether an integrated platform can perform a task, but whether it meets the workload requirements well enough to justify retiring or avoiding another system.

```text
Integrated capability
        |
        v
Meets workload SLOs and controls? -- no --> Select or retain a specialist
        |
       yes
        v
Operational simplification exceeds lock-in cost? -- no --> Keep boundaries portable
        |
       yes
        v
Adopt on the primary platform with a tested exit path
```

## Consolidation value

Every additional production platform introduces more than a license or compute bill. It adds identity and access management, networking, data movement, observability, incident response, upgrades, skills, procurement, and ownership boundaries.

An integrated capability can be the better enterprise choice even when a specialist is technically stronger in isolation. Consolidation is valuable when the integrated option:

- satisfies measured latency, throughput, consistency, availability, and recovery objectives;
- reduces duplicated data and cross-system synchronization;
- uses the existing governance and operating model;
- lowers total lifecycle cost and cognitive load;
- keeps failure domains and ownership understandable.

## When a specialist earns its place

Retain or introduce a specialist when its workload advantage is material rather than theoretical. Typical examples include advanced search semantics, ultra-low-latency caching, mature event-stream guarantees, demanding OLTP behavior, or regulatory isolation that the primary platform cannot satisfy.

Document the threshold explicitly. “Best tool” is not a complete decision criterion; neither is “one platform.” Compare the capability gap with the full cost of another production system.

| Dimension | Evidence to collect |
| --- | --- |
| Workload fit | Representative latency, concurrency, throughput, query, and consistency tests |
| Reliability | Availability model, failure behavior, replay, restore, RTO, and RPO |
| Security | Identity boundaries, encryption, network controls, audit, and policy coverage |
| Operations | Deployment, monitoring, on-call ownership, upgrades, and incident tooling |
| Economics | Compute, storage, data movement, licenses, people, and migration cost |
| Portability | Data formats, protocols, SQL/API coupling, contracts, and export time |

## Useful lock-in versus irreversible lock-in

Lock-in is not binary. Managed capabilities deliberately exchange some portability for speed and lower operational burden. The goal is to accept useful dependency without making departure impractical.

Maintain an exit path by:

- storing critical data in open, documented formats where feasible;
- defining stable contracts between domains and consumers;
- isolating proprietary orchestration and APIs from core business rules;
- preferring portable SQL and standard protocols where they meet the need;
- keeping source-to-target lineage and reconstruction procedures;
- measuring how long a complete export and restore would take;
- recording the components that must be replaced during migration;
- periodically testing the highest-risk recovery or extraction path.

An exit plan is credible only if it names data volumes, dependencies, sequence, duration, acceptable downtime, and owners. “We use open formats” alone is not an exit strategy.

## Resilience is designed by layer

Running two complete equivalent platforms in parallel is rarely the default resilience strategy. It doubles synchronization, security, testing, operations, and cost while introducing consistency questions during failover.

Start from business requirements:

- **RTO**: the maximum acceptable time to restore service;
- **RPO**: the maximum acceptable data loss measured in time;
- cost and impact of downtime;
- regulatory and geographic constraints.

Then design redundancy at the layer that addresses the failure:

| Failure concern | Typical control |
| --- | --- |
| Compute or zone failure | Multi-zone service design and automatic restart or failover |
| Regional failure | Cross-region copy/replication and a tested regional recovery runbook |
| Data corruption or accidental deletion | Versioning, immutable backups, point-in-time recovery, and restore tests |
| Platform control-plane outage | Degraded-mode procedures and clearly defined recovery dependencies |
| Vendor or strategic exit | Portable data, contracts, export tooling, and a migration plan |

Multi-cloud active-active operation may be justified for exceptional regulatory or continuity requirements, but it should follow quantified RTO/RPO evidence rather than a generic belief that two vendors are automatically safer.

## Architecture decision checklist

Before consolidating a workload onto the primary platform:

1. Define the workload SLO, security controls, RTO, and RPO.
2. Benchmark the integrated capability with representative data and concurrency.
3. Compare it with the specialist on total lifecycle cost, not feature count alone.
4. Identify unsupported requirements and Preview/Beta dependencies.
5. Describe data ownership, interfaces, failure domains, and operational ownership.
6. Quantify switching cost and document a realistic exit path.
7. Record the decision, evidence, expiry/review date, and reversal triggers in an ADR.

## Databricks as an example

Databricks illustrates the consolidation trend by spanning analytical storage and processing, governance, low-latency stream processing and serving, and managed PostgreSQL-compatible OLTP. These capabilities can simplify some architectures, but their boundaries and maturity differ. The product-specific capability map and current limitations belong in the [Databricks Engineering and Architecture Portfolio](https://github.com/Enrique1987/databricks/blob/main/docs/architecture/platform-capability-map.md).
