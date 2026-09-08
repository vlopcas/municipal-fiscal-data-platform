# Brazilian Municipal Economic & Fiscal Data Platform — Roadmap

## 1. Project Vision

Build a production-oriented data platform that ingests, validates, standardizes, historizes, transforms, and serves public economic, fiscal, accounting, demographic, and geographic data for Brazilian municipalities.

The platform is not intended to be merely a dashboard or a collection of ETL scripts. Its primary purpose is to explore and demonstrate robust Data Engineering and Analytics Engineering practices using real public data, real source-system constraints, and a clearly defined analytical domain.

---

## 2. Core Problem

Brazilian municipal economic and fiscal data is available from official public sources, but it is distributed across different systems, datasets, reporting periods, schemas, and update cycles.

The project aims to create a reliable integrated data layer around a canonical municipal entity:

```text
Municipality
├── Geography
├── Population
├── GDP and economic activity
├── Fiscal reports
├── Accounting data
└── Derived indicators
```

The main integration key should be the official IBGE municipality code whenever available.

---

## 3. Main Data Sources

### 3.1 Siconfi / Tesouro Nacional

Primary fiscal and accounting source.

Candidate datasets:

- DCA — Declaração de Contas Anuais
- RREO — Relatório Resumido da Execução Orçamentária
- RGF — Relatório de Gestão Fiscal
- MSC — Matriz de Saldos Contábeis
- Extrato de Entregas

Expected engineering challenges:

- pagination;
- large extractions;
- multiple reporting periods;
- rectified submissions;
- changing layouts/taxonomies;
- missing submissions;
- incremental ingestion;
- backfills;
- source rate/usage constraints;
- schema evolution.

### 3.2 IBGE Localities

Canonical geographic dimension.

Expected fields include:

- IBGE municipality code;
- municipality name;
- state;
- state abbreviation;
- region;
- intermediate region;
- immediate region.

### 3.3 IBGE / SIDRA — Municipal GDP

Initial target: PIB dos Municípios, including SIDRA table 5938 where applicable.

Candidate measures:

- GDP;
- taxes net of subsidies;
- total gross value added;
- agriculture;
- industry;
- services;
- public administration and related activities;
- activity shares.

### 3.4 IBGE Population

Population estimates/census-derived municipal population data appropriate to the reference period.

Used for indicators such as:

- GDP per capita;
- revenue per capita;
- expenditure per capita;
- debt per capita.

Population reference dates and methodology must be preserved rather than treated as interchangeable annual values.

---

## 4. Explicit Non-Goals

The project should not initially attempt to:

- build a complete Brazilian government data lake;
- ingest every dataset available from IBGE or Tesouro;
- use Kafka without an event-streaming requirement;
- use Spark merely to demonstrate Spark;
- deploy Kubernetes prematurely;
- build predictive ML before the data platform is reliable;
- rank municipalities as "well" or "poorly" managed without defensible methodology;
- hide source-specific semantics behind oversimplified metrics.

Complexity should be introduced only when justified by observed requirements.

---

## 5. Architectural Principles

1. **Raw data is reproducible and traceable.**
2. **Transformations do not silently destroy source semantics.**
3. **Every published dataset has provenance.**
4. **Pipelines are idempotent where feasible.**
5. **Backfills are first-class operations.**
6. **Data quality is tested, not assumed.**
7. **Source revisions are explicitly handled.**
8. **Reference period, publication time, and ingestion time are distinct concepts.**
9. **Schemas and contracts are versioned when necessary.**
10. **Infrastructure grows with measured requirements.**
11. **The municipality is the primary analytical entity.**
12. **Official identifiers are preferred over fuzzy joins.**

---

## 6. Target Architecture

```text
                         OFFICIAL SOURCES

             Siconfi          IBGE SIDRA        IBGE Localities
                │                  │                   │
                └──────────────────┼───────────────────┘
                                   │
                                   ▼
                            SOURCE CONNECTORS
                                   │
                                   ▼
                         INGESTION / EXTRACTION
                                   │
                                   ▼
                         ┌───────────────────┐
                         │    RAW / BRONZE   │
                         │ immutable source  │
                         │ representations   │
                         └─────────┬─────────┘
                                   │
                         validation/contracts
                                   │
                                   ▼
                         ┌───────────────────┐
                         │      SILVER       │
                         │ standardized and  │
                         │ normalized data   │
                         └─────────┬─────────┘
                                   │
                            transformations
                                   │
                                   ▼
                         ┌───────────────────┐
                         │       GOLD        │
                         │ analytical marts  │
                         └─────────┬─────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                   SQL            API         BI/Analytics
```

This is a logical architecture. Physical technologies should be selected incrementally.

---

## 7. Proposed Repository Structure

Initial target:

```text
municipal-fiscal-data-platform/
├── AGENTS.md
├── README.md
├── LICENSE
├── pyproject.toml
├── .gitignore
├── src/
│   ├── connectors/
│   ├── ingestion/
│   ├── validation/
│   ├── transformations/
│   ├── quality/
│   └── common/
├── tests/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   ├── data-sources/
│   ├── data-model/
│   └── experiments/
├── pipelines/
├── data/
│   ├── raw/
│   ├── silver/
│   └── gold/
└── infrastructure/
```

The final structure should evolve from implementation needs rather than being fully scaffolded on day one.

---

# 8. Phase 0 — Discovery and Source Contracts

## Objective

Convert the initial technical discovery into executable knowledge about each source before building the platform.

## Tasks

### Siconfi

Document:

- endpoints;
- parameters;
- pagination;
- response structure;
- reporting periods;
- entity identifiers;
- DCA semantics;
- RREO semantics;
- RGF semantics;
- MSC semantics;
- submission/rectification metadata;
- usage restrictions;
- error behavior.

### IBGE

Document:

- Localities API;
- SIDRA API;
- municipal GDP dataset/table;
- population dataset(s);
- municipality codes;
- territorial changes;
- available years;
- source metadata.

### Deliverables

```text
docs/data-sources/
├── siconfi.md
├── ibge-localities.md
├── ibge-gdp.md
└── ibge-population.md
```

Create source contracts describing expected schema and semantics.

## Exit Criteria

We can answer:

- What exactly are we extracting?
- From which endpoint/dataset?
- At what grain?
- With which identifier?
- For which period?
- How do we detect updates?
- How do we reproduce an extraction?

---

# 9. Phase 1 — Minimal Vertical Slice: Goiás

## Objective

Build the smallest end-to-end useful pipeline.

Do not start with the entire country.

Initial scope:

```text
IBGE Localities
      +
Siconfi DCA
      +
IBGE Municipal GDP
      +
IBGE Population
      ↓
Municipal analytical dataset for Goiás
```

## Tasks

### 9.1 Municipality Dimension

Build:

```text
dim_municipality
```

Candidate fields:

```text
ibge_code
municipality_name
state_code
state_name
state_abbreviation
region_code
region_name
immediate_region
intermediate_region
```

### 9.2 DCA Connector

Implement:

```text
Siconfi
  ↓
extract
  ↓
raw
```

Requirements:

- HTTP timeout;
- retries;
- pagination;
- structured errors;
- request metadata;
- reproducible extraction;
- no duplicate ingestion after retry.

### 9.3 GDP Connector

Extract municipal GDP data for Goiás.

Persist the source representation before transformations.

### 9.4 Population Connector

Extract population data compatible with the selected analytical periods.

### 9.5 First Normalization

Create standardized tables.

### 9.6 First Mart

Create:

```text
mart_municipal_economic_fiscal
```

Initial measures may include:

```text
population
gdp
gdp_per_capita
revenue
expenditure
revenue_per_capita
expenditure_per_capita
fiscal_balance
```

Only publish indicators whose definitions have been documented and validated.

## Exit Criteria

For municipalities in Goiás, the platform can reproducibly answer basic economic/fiscal queries from integrated official sources.

---

# 10. Phase 2 — Raw / Bronze Layer

## Objective

Make ingestion reproducible and auditable.

Raw data should preserve source information as closely as practical.

## Metadata

Each ingestion should record information such as:

```text
source
dataset
endpoint/resource
reference_period
extracted_at
ingested_at
request_parameters
source_version when available
content/checksum when useful
pipeline_run_id
```

## Storage

Evaluate:

- JSON for source payload preservation;
- Parquet for scalable analytical storage;
- local filesystem during development;
- S3-compatible object storage later if justified.

## Partitioning

Potential partitions:

```text
source=siconfi/
dataset=dca/
year=2025/
state=GO/
```

or equivalent structures based on measured access patterns.

## Exit Criteria

A failed transformation can be rerun without unnecessarily redownloading unchanged source data.

---

# 11. Phase 3 — Data Contracts and Validation

## Objective

Detect unexpected source changes before they corrupt downstream models.

Validate:

- expected fields;
- data types;
- required identifiers;
- accepted enumerations;
- period formats;
- numeric parsing;
- nullability expectations.

Example:

```text
SiconfiDCARecord
├── ibge_code
├── year
├── account
├── value
└── metadata
```

Unexpected schema changes should fail visibly or be quarantined, not silently ignored.

---

# 12. Phase 4 — Silver / Standardization Layer

## Objective

Separate source representation from canonical platform representation.

Typical transformations:

```text
source-specific names
        ↓
canonical names

source numeric strings
        ↓
typed numeric values

different period formats
        ↓
canonical period representation

source municipality identifiers
        ↓
canonical ibge_code
```

Preserve source identifiers where useful for traceability.

## Candidate Tables

```text
dim_municipality
fact_population
fact_gdp
fact_fiscal_annual
```

---

# 13. Phase 5 — Data Quality Framework

## Objective

Treat quality as part of the product.

### Structural Tests

```text
ibge_code IS NOT NULL
year IS VALID
numeric fields parse correctly
```

### Referential Tests

```text
fact_*.ibge_code
    ↓
dim_municipality.ibge_code
```

### Uniqueness Tests

Define uniqueness at the correct grain for every fact table.

### Completeness Tests

Compare:

```text
expected municipalities
vs
received municipalities
```

### Freshness Tests

Detect whether expected new reporting periods have arrived.

### Domain Tests

Examples, after verifying semantics:

```text
population > 0
```

and other defensible invariants.

### Quarantine

Bad records should be inspectable rather than silently discarded.

---

# 14. Phase 6 — National Expansion

## Objective

Scale the validated Goiás vertical slice to Brazil.

Progression:

```text
Goiás
  ↓
Central-West
  ↓
Brazil
```

Do not assume that behavior observed for one state generalizes perfectly.

Measure:

- extraction duration;
- API calls;
- raw storage size;
- normalized storage size;
- transformation time;
- memory consumption;
- failure frequency.

These measurements determine whether architectural changes are justified.

---

# 15. Phase 7 — Orchestration

## Objective

Replace manual execution with explicit pipeline dependencies and schedules.

Potential orchestrator:

- Dagster; or
- Airflow.

Choose after evaluating project requirements.

Conceptual DAG:

```text
municipalities
      │
      ├───────────────┐
      ▼               ▼
   population        GDP
      │               │
      └──────┬────────┘
             │
          Siconfi
             │
             ▼
         validation
             │
             ▼
          silver
             │
             ▼
           marts
             │
             ▼
      quality/publish
```

Required capabilities:

- retries;
- schedules;
- dependency management;
- run history;
- backfills;
- parameterized runs;
- failure visibility.

---

# 16. Phase 8 — Incremental Loading and Idempotency

## Objective

Stop rebuilding everything unnecessarily.

Each source needs an explicit incremental strategy.

Possible patterns:

```text
watermark
reference period
source update timestamp
content checksum
submission version
```

A rerun of the same logical ingestion should not create duplicate facts.

Document idempotency guarantees per pipeline.

---

# 17. Phase 9 — Revisions and Historization

## Objective

Correctly represent official corrections and revisions.

The platform must distinguish:

```text
reference period
publication/submission time
ingestion time
current version
historical version
```

Possible model:

```text
valid_from
valid_to
is_current
source_updated_at
ingested_at
```

Do not automatically adopt SCD Type 2 everywhere. Select the historization model according to the semantics of each dataset.

Use Siconfi delivery/rectification information where appropriate.

---

# 18. Phase 10 — RREO and RGF

## Objective

Move beyond annual fiscal snapshots.

Add:

- RREO;
- RGF.

This introduces intra-year fiscal reporting and additional domain semantics.

Expected new challenges:

- reporting periods;
- demonstratives/annexes;
- different grains;
- account/metric taxonomies;
- source-specific validation;
- temporal comparisons.

Do not force these datasets into the DCA schema if their semantics differ.

---

# 19. Phase 11 — MSC

## Objective

Introduce the richer and potentially heavier accounting dataset only after the platform foundation is stable.

MSC can introduce:

- monthly accounting data;
- richer account structures;
- multiple dimensions;
- significantly larger volumes;
- changing taxonomies/layouts.

At this point reevaluate processing technology based on measurements.

Potential progression:

```text
DuckDB / Polars
      ↓
measure
      ↓
distributed processing only if justified
```

Spark should be adopted only if the workload demonstrates a real need.

---

# 20. Phase 12 — Schema Evolution

## Objective

Handle changes between reporting exercises and source versions.

Create explicit mechanisms for:

```text
source schema
      ↓
version-aware parser
      ↓
canonical schema
```

Document breaking changes.

Possible structure:

```text
schemas/
├── siconfi/
│   ├── dca/
│   ├── rreo/
│   ├── rgf/
│   └── msc/
└── ibge/
```

Tests should include historical source fixtures from different schema versions where legally and practically appropriate.

---

# 21. Phase 13 — Transformation Layer / Analytics Engineering

## Objective

Build trustworthy analytical models rather than querying raw fiscal records directly.

Potential marts:

```text
mart_municipal_overview
mart_municipal_fiscal
mart_municipal_economy
mart_municipal_per_capita
mart_municipal_fiscal_history
mart_state_aggregates
mart_regional_comparison
```

Consider dbt when transformations and dependencies become sufficiently complex to justify it.

Every published metric should include:

- definition;
- source;
- grain;
- unit;
- applicable period;
- transformation logic;
- known limitations.

---

# 22. Phase 14 — Semantic Indicators

Candidate indicators:

```text
GDP per capita
revenue per capita
expenditure per capita
fiscal balance
revenue growth
expenditure growth
revenue / GDP
expenditure / GDP
economic activity composition
```

Do not publish derived indicators merely because they are mathematically possible.

For each indicator document:

```text
name
formula
meaning
source datasets
grain
limitations
```

---

# 23. Phase 15 — Data Lineage and Provenance

## Objective

Allow a published number to be traced back to its origin.

Desired chain:

```text
dashboard/API value
        ↓
gold model
        ↓
silver model
        ↓
raw record
        ↓
source dataset
        ↓
official source
```

Lineage can initially be documented through metadata and transformations before adopting dedicated lineage infrastructure.

---

# 24. Phase 16 — Observability

Monitor the platform itself.

Potential metrics:

```text
pipeline duration
records extracted
records rejected
API failures
retry count
freshness
missing municipalities
schema violations
duplicate records
storage growth
transformation duration
```

Create alerts only for actionable conditions.

---

# 25. Phase 17 — Serving Layer

Once the data platform is stable, expose data through one or more interfaces.

### SQL

Primary analytical interface.

### API

Optional FastAPI service for curated datasets.

Example conceptual resources:

```text
/municipalities
/municipalities/{ibge_code}
/municipalities/{ibge_code}/fiscal
/municipalities/{ibge_code}/economy
/municipalities/{ibge_code}/indicators
```

### BI / Dashboard

Potential tools:

- Metabase;
- Superset;
- lightweight custom interface.

The dashboard is a consumer of the platform, not the platform itself.

---

# 26. Phase 18 — Infrastructure Evolution

## Local Development

Start simple:

```text
Python
Parquet
DuckDB
Docker where useful
```

Then potentially evolve toward:

```text
Object Storage
PostgreSQL
dbt
Dagster/Airflow
BI
API
```

Cloud infrastructure should come after local architecture is understood.

---

# 27. Phase 19 — CI/CD

CI should eventually run:

```text
lint
unit tests
connector contract tests with fixtures/mocks
transformation tests
data model tests
documentation checks
```

Do not make CI depend on uncontrolled large live API extractions.

Use small public/synthetic fixtures where appropriate.

---

# 28. Testing Strategy

## Unit Tests

Test:

- parsers;
- normalization;
- period handling;
- metric calculations;
- validation logic.

## Integration Tests

Test:

```text
connector → raw
raw → silver
silver → gold
```

## Contract Tests

Detect source incompatibilities.

## Data Tests

Validate production datasets.

## Regression Tests

Ensure known historical inputs continue producing expected outputs.

---

# 29. Documentation

Documentation is part of the project.

Suggested structure:

```text
docs/
├── architecture/
│   ├── overview.md
│   └── data-flow.md
├── decisions/
│   ├── ADR-001-...
│   └── ...
├── data-sources/
│   ├── siconfi.md
│   ├── ibge-localities.md
│   ├── ibge-gdp.md
│   └── ibge-population.md
├── data-model/
│   ├── dimensions.md
│   ├── facts.md
│   └── marts.md
├── metrics/
└── experiments/
```

---

# 30. Architecture Decision Records

Use ADRs for decisions that matter.

Examples:

```text
ADR-001 — Canonical municipality identifier
ADR-002 — Raw storage format
ADR-003 — Partitioning strategy
ADR-004 — Orchestrator selection
ADR-005 — Warehouse/storage selection
ADR-006 — Historization strategy
ADR-007 — Transformation framework
```

Each ADR should capture:

```text
context
decision
alternatives
consequences
```

---

# 31. Security and Responsible Use

The intended datasets are public government data.

Still:

- do not commit credentials;
- do not commit private datasets;
- respect official API usage guidance;
- preserve source attribution;
- avoid presenting derived metrics as official government statistics;
- clearly distinguish source values from project-derived indicators;
- document methodology and limitations.

---

# 32. Performance Strategy

Do not optimize prematurely.

Measure first:

```text
extraction time
rows
bytes
memory
CPU
query latency
transformation time
```

Then optimize using:

```text
partitioning
columnar formats
predicate pushdown
incremental processing
parallelism
caching
```

Only consider distributed computation after simpler approaches are demonstrably insufficient.

---

# 33. Potential Technology Evolution

This is a roadmap, not a mandatory stack.

## Early

```text
Python
HTTP client
Pydantic or equivalent validation
Parquet
DuckDB
pytest
```

## Intermediate

Potentially:

```text
PostgreSQL
dbt
Dagster or Airflow
S3-compatible object storage
Docker
```

## Serving

Potentially:

```text
FastAPI
Metabase / Superset
```

## Advanced

Only if justified:

```text
distributed processing
cloud object storage
managed warehouse/lakehouse
dedicated lineage platform
```

Kafka is intentionally absent from the baseline because the identified official sources are primarily batch-oriented.

---

# 34. Milestones

## M0 — Source Discovery

- official sources documented;
- initial contracts defined;
- sample requests reproducible.

## M1 — Goiás Vertical Slice

- municipalities;
- DCA;
- GDP;
- population;
- first integrated mart.

## M2 — Reliable Data Layers

- raw;
- silver;
- gold;
- validation;
- tests.

## M3 — Brazil

- national ingestion;
- performance measurements;
- partition strategy validated.

## M4 — Orchestration

- scheduled pipelines;
- retries;
- backfills;
- run history.

## M5 — Quality & Observability

- automated quality checks;
- freshness;
- completeness;
- pipeline metrics.

## M6 — Fiscal Expansion

- RREO;
- RGF.

## M7 — Historization

- revisions;
- rectifications;
- source versions;
- temporal semantics.

## M8 — MSC

- monthly accounting ingestion;
- richer accounting dimensions;
- performance reevaluation.

## M9 — Analytics Engineering

- curated marts;
- documented metrics;
- lineage.

## M10 — Serving

- SQL;
- API and/or BI;
- public-facing demonstration.

## M11 — Production Hardening

- CI/CD;
- reproducible environments;
- stronger observability;
- architecture documentation;
- public release.

---

# 35. Public V1 Definition

A credible first public version does **not** require every roadmap phase.

A strong V1 should have:

```text
official source connectors
        ↓
raw storage
        ↓
validation
        ↓
normalized municipal models
        ↓
DCA + GDP + population
        ↓
quality checks
        ↓
curated mart
        ↓
reproducible pipeline
        ↓
tests + documentation
```

Prefer a coherent small system over a large unfinished architecture.

---

# 36. Learning Objectives

The project should deliberately exercise:

### Data Engineering

- source ingestion;
- APIs;
- batch pipelines;
- incremental processing;
- idempotency;
- retries;
- backfills;
- partitioning;
- schema evolution;
- historization;
- data quality;
- observability.

### Data Modeling

- grain;
- dimensions;
- facts;
- canonical identifiers;
- temporal models;
- dimensional modeling;
- analytical marts.

### Analytics Engineering

- transformations;
- metric definitions;
- testing;
- lineage;
- documentation.

### Software Engineering

- modular architecture;
- abstractions around external sources;
- error handling;
- testing;
- CI/CD;
- configuration;
- maintainability.

### Infrastructure

- containers;
- storage;
- databases;
- orchestration;
- deployment.

---

# 37. Portfolio Positioning

The project should demonstrate a capability distinct from the rest of the portfolio:

```text
medaudit
→ Applied AI / RAG / LLM Systems

third-party-lifecycle
→ Backend / SaaS / Software Architecture

municipal-fiscal-data-platform
→ Data Engineering / Analytics Engineering

coping_struggles_prediction
→ ML Engineering / MLOps
```

---

# 38. Final Principle

The project should not ask:

> Which technologies can we fit into a data platform?

It should ask:

> What does a reliable municipal economic and fiscal data platform require?

Then introduce technologies only when the data, scale, reliability requirements, or operational constraints justify them.
