# Benny Wambua · `BenWam254`

Building async CDC engines, runtime schema guards, and local OLAP pipelines with Python, DuckDB, and Polars.

Data Infrastructure, Async Backend, Developer Tooling. Nairobi, Kenya.

![Python](https://img.shields.io/badge/Python_3.11+-24292e?style=flat-square&logo=python&logoColor=white)
![AsyncIO](https://img.shields.io/badge/AsyncIO-24292e?style=flat-square&logo=python&logoColor=white)
![DuckDB](https://img.shields.io/badge/DuckDB-24292e?style=flat-square&logo=duckdb&logoColor=white)
![Polars](https://img.shields.io/badge/Polars-24292e?style=flat-square&logo=polars&logoColor=white)
![dbt](https://img.shields.io/badge/dbt--core-24292e?style=flat-square&logo=dbt&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-24292e?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-24292e?style=flat-square&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-24292e?style=flat-square&logo=docker&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-24292e?style=flat-square&logo=typescript&logoColor=white)

---

## Systems & Architecture

### Stream Processing & CDC
- **[`async-cdc-engine`](https://github.com/BenWam254/async-cdc-engine)**: AsyncIO listener that parses CDC log events and applies state mutations without blocking the event loop.
- **[`data-stream-pipeline`](https://github.com/BenWam254/data-stream-pipeline)**: Multi-stage stream processor. FastAPI ingress, Polars LazyFrame transforms, DuckDB sinks.
- **[`scraper-engine`](https://github.com/BenWam254/scraper-engine)** / **[`scraper_tracker`](https://github.com/BenWam254/scraper_tracker)**: Crawl4AI async crawlers with crawl state persisted in SQLite/DuckDB.

### Analytical Data Engines
- **[`analytics-dbt-duckdb`](https://github.com/BenWam254/analytics-dbt-duckdb)**: In-process OLAP modeling with dbt-duckdb. No warehouse server, no network hop.
- **[`local-elt-pipeline`](https://github.com/BenWam254/local-elt-pipeline)**: End-to-end ELT orchestrated with Docker Compose. One command to bring up the stack.

### Runtime Reliability
- **[`schema-guard`](https://github.com/BenWam254/schema-guard)**: Validates incoming JSON against structural contracts before anything reaches storage.
- **[`data-ops-dashboard`](https://github.com/BenWam254/data-ops-dashboard)**: Pipeline health, latency tracking, and monitoring UI.

---

## Design Notes

### `async-cdc-engine`
- **AsyncIO over threads.** The workload is I/O-bound: a socket reading log events and small state updates. A single event loop avoids per-thread stack memory, GIL contention, and lock management around shared state. CPU-heavy steps (if any) get pushed to an executor so they do not stall the loop.
- **Mutations are applied in log order.** State changes are sequential per stream, which keeps replay deterministic after a reconnect.
- **Cost:** one blocking call in a handler stalls every consumer. Handlers must be strictly non-blocking.

### `schema-guard`
- **Validate at the boundary.** Contract checks run before the storage load, so bad payloads are rejected or quarantined instead of corrupting tables downstream.
- **Runtime enforcement over load-time failure.** A drifted field is caught on the offending record, not discovered as a failed batch.
- **Cost:** per-record validation adds CPU on the hot path. It is the price of catching drift before it lands.

### `analytics-dbt-duckdb`
- **DuckDB is in-process.** Models run inside the dbt process against local files. No connection pool, no server to operate, no network latency.
- **Columnar scans over row-store.** Aggregations over wide tables read only the columns they touch. PostgreSQL stays the choice for concurrent writes and transactional workloads; DuckDB covers single-writer analytical work.
- **Zero-copy reads** from Parquet/Arrow avoid materializing intermediate copies between stages.
- **Cost:** single-writer model. Not a fit for many concurrent writers.

### `data-stream-pipeline`
- **Polars LazyFrames over Pandas.** The query plan is built first and optimized (projection and predicate pushdown) before execution. Pandas executes eagerly and materializes every intermediate.
- **FastAPI for ingress** because it is async-native and shares the event loop with the rest of the pipeline.
- **DuckDB as sink** keeps the analytical output queryable in-process with no separate load step.

---

## Stack

| Domain | Tools |
|---|---|
| Languages | Python 3.11+, SQL, TypeScript, JavaScript (ES6+), HTML5, CSS3 |
| Async & ingestion | AsyncIO, FastAPI, Crawl4AI |
| Analytical engines | DuckDB, Polars, dbt-core, PostgreSQL, SQLite |
| Infrastructure | Docker, Docker Compose, WSL (Ubuntu), Git, Vercel |
| Frontend | Next.js (App Router), React, Tailwind CSS |

---

## Client Work

Forge Digital Solutions is my design and development studio. It covers production web interfaces (Next.js, React, Tailwind, deployed on Vercel), data automation, and API integrations.

- Site: [benwam.vercel.app](https://benwam.vercel.app)
- LinkedIn: [in/wambuabenny-007](https://www.linkedin.com/in/wambuabenny-007)
