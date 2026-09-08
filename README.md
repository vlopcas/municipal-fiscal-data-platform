# Municipal Fiscal Data Platform

A data platform for ingesting, standardizing, validating, and serving Brazilian municipal economic and fiscal data from official public sources.

## Status

> Planned — development has not started yet.

## Overview

Brazilian municipal economic and fiscal data is publicly available across multiple government systems, but it is distributed across different sources, schemas, reporting periods, and update cycles.

This project aims to build a production-oriented data platform that integrates these datasets into a reliable and traceable analytical layer centered around Brazilian municipalities.

The platform will evolve incrementally, starting with a small end-to-end pipeline and introducing additional infrastructure only when justified by data volume, reliability, or operational requirements.

## Initial Data Sources

The initial scope includes official public data from:

- **Siconfi / Tesouro Nacional** — municipal fiscal and accounting data
- **IBGE / SIDRA** — municipal GDP and economic indicators
- **IBGE Population Data** — municipal population estimates and statistics
- **IBGE Localities** — canonical municipality and geographic information

Additional sources may be introduced as the platform evolves.

## Initial Scope

The first version will focus on integrating:

- Municipality and geographic dimensions
- Population
- Municipal GDP
- Annual fiscal data
- Revenue and expenditure
- Derived per-capita and economic indicators

The initial vertical slice will use municipalities from Goiás before expanding to the entire country.

## Engineering Goals

The project is designed to explore production-oriented Data Engineering and Analytics Engineering practices, including:

- Batch data ingestion
- Source connectors
- Raw and curated data layers
- Data contracts
- Data quality
- Incremental processing
- Idempotency
- Backfills
- Schema evolution
- Data historization
- Dimensional modeling
- Analytical marts
- Pipeline orchestration
- Data lineage
- Observability
- Reproducible data pipelines

## Architecture

The platform will follow an incremental layered architecture:

```text
Official Sources
      ↓
Source Connectors
      ↓
Ingestion
      ↓
Raw / Bronze
      ↓
Validation
      ↓
Silver / Standardized
      ↓
Transformations
      ↓
Gold / Analytical Marts
      ↓
SQL / API / Analytics
```

Technology choices will be introduced progressively according to actual requirements rather than predetermined architecture.

## Potential Technology Stack

The initial implementation is expected to explore technologies such as:

- Python
- Parquet
- DuckDB
- PostgreSQL
- dbt
- Dagster or Airflow
- Docker

The stack is intentionally not fixed and may evolve as architectural requirements become clearer.

## Data Modeling

The municipality will be the primary analytical entity.

Official IBGE municipality codes will be used as canonical identifiers whenever possible.

```text
Municipality
├── Geography
├── Population
├── GDP
├── Economic Activity
├── Fiscal Data
├── Accounting Data
└── Derived Indicators
```

## Development Strategy

Development will follow incremental vertical slices.

The initial progression is expected to be:

```text
Source Discovery
      ↓
Goiás Vertical Slice
      ↓
Raw / Silver / Gold Layers
      ↓
Data Quality
      ↓
National Expansion
      ↓
Orchestration
      ↓
Incremental Processing
      ↓
Historization
      ↓
RREO / RGF
      ↓
MSC
      ↓
Analytics & Serving
```

Complexity will be introduced only when justified by measured requirements.

## Roadmap

A detailed technical roadmap will guide the implementation of the project as development begins.

## License

This project is licensed under the MIT License.
